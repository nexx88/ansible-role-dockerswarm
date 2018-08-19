Ansible Role: Docker Swarm
==========================

This Ansible Role is a fork of [atosatto/ansible-dockerswarm](https://github.com/atosatto/ansible-dockerswarm), customized to bootstrap a set of VMs to become a Docker Swarm, setup vSphere VDVS and setup all necessary Docker Secrets.  This requires Docker 1.12.0 or higher.

Requirements
------------
Role contains credentials and secrets in vars/secrets.yml therefore this file is encrypted with ansible-vault.  
In order to decrypt it you need to have file which contains passphrase. Location of the file needs to be passed to ansible-playbook call
Signed certs and keys are pulled from Hashicorp Vault.


```
ansible-playbook docker-swarm/playbook.yml --vault-password-file=~/.vault_pass.txt
```

Role Variables
--------------

Available variables are listed below, along with default values (see defaults/main.yml):

    skip_group: True
    skip_swarm: False
    skip_vdvs: False
    skip_secrets: False
    driver_alias: vsphere
    vdvs_plugin_name: vmware/docker-volume-vsphere
    docker_plugin_registry: docker-registry.acme.com
    vdvs_version: "0.20" #this needs to be string value so use quotes!

`skip_vdvs` is used to determine whether vSphere Docker Volume Service plugin should be installed.

`driver_alias` is what to call the vSphere Docker Volume Storage plugin locally

`vdvs_plugin_name` is the name of the vSphere Docker Volume Storage plugin in the plugin repo

`docker_plugin_registry` is the name/url of the Docker plugin registry.  Change this if you need to download the plugin from a different location.
More info our local plugin repo can be found in [Conluence](https://eagleinvsys.atlassian.net/wiki/spaces/M2/pages/61702249/Docker+Repository#DockerRepository-CreatingaDockerRepositoryForPlugins)

`vdvs_version` is where you define version of vSphere Docker Volume Storage plugin.
  NOTE: if you update this you need to edit also ansible-role-vdvs-esx/defaults/main.yml

`skip_secrets` if used secrets from vault will be obtained and stored in docker with docker secret command.
For now eagleinvsys.crt and eagleinvsys.key are loaded to docker. Both secrets and vault configs are stored in vars/vault.yml
The file is encrypted with ansible-vault. If you want to use secrets in docker container you can set it in docker-compose file:
```
secrets:
  acme.crt:
    external: true
  acme.key:
    external: true
```
This will make files available at /run/secrets dir in docker container

    docker_dependencies: "{{ default_docker_dependencies }}"

Extra packages that have to installed together with Docker.
The value of `default_docker_dependencies` depends on the target OS family.

    docker_swarm_interface: "{{ ansible_default_ipv4['alias'] }}"

Setting `docker_swarm_interface` allows you to define which network interface will be used for cluster inter-communication.

    docker_swarm_addr: "{{ hostvars[inventory_hostname]['ansible_' + docker_swarm_interface]['ipv4']['address'] }}"

By default, the ip address for raft API will be taken from desired interface.
You can setup listening address where the raft APIs will be exposed, overriding
the `docker_swarm_addr` variable value in your playbook.

    docker_swarm_port: 2377

Listening port where the raft APIs will be exposed.

Dependencies
------------

[ansible-role-docker](https://www.google.com) 

Example Playbook
----------------

    $ cat inventory
    [docker_swarm_manager]
    swm01.acme.com
    swm02.acme.com
    swm03.acme.com

    [docker_swarm_worker]
    sww01.acme.com
    sww02.acme.com
    sww03.acme.com
    sww04.acme.com
    sww05.acme.com

    $ cat playbook.yml
    - name: "Provision Docker Swarm"
      hosts: docker_swarm*
      roles:
        - { role: ansible-role-docker-swarm }

Example Execution
----------------

    $ ansible-playbook docker-swarm/playbook.yml -i docker-swarm/inventory --vault-password-file=~/.vault_pass.txt --extra-vars "docker_plugin_registry=docker-registry.acme.com"

License
-------

MIT

Author Information
------------------

EIS EngOps Team