# ansible-docker-swarm-jenkins

>
> Provisions a docker swarm cluster with docker-compose
> installed on each manager and worker.
>
> Jenkins 2.x is installed on the primary manager.
>

## ACKNOWLEDGEMENTS

Bare-facedly code-napped then man-handled from these sources:

* https://github.com/caylent/ansible-docker-swarm
* https://github.com/nickjj/ansible-docker
* https://github.com/karlmdavis/ansible-role-jenkins2

---

## Tell me about ...

* [RUNNING THIS](#running-this)

* [DEPLOYED CONFIGURATION](#deployed-configuration)
    * [... platform](#platform)
    * [... installed components](#installed-components)
    * [... system cfg](#system-cfg)

* [ROLE CONFIGURATION](#role-configuration)

* [PYTHON-DOCKER](#python-docker)

---

## RUNNING THIS

Configure hosts.ini for your platform:

e.g.

```dosini
# ... example hosts.ini for 2 managers, 2 workers
[swarm-manager-primary]
manager1 ansible_host="10.0.0.10"

[swarm-manager-secondary]
manager2 ansible_host="10.0.0.11"

[swarm-worker]
worker1 ansible_host="10.0.1.100"
worker2 ansible_host="10.0.1.101"

[swarm-manager-all]
manager1
manager2
```

Now run ansible playbook ...

*Note the -e options for jenkins_admin are only required if you've enabled jenkins security.*

If you are allowing anonymous access or running for the first time, *omit* those -e options
in the example below. However if you've enabled security and created a user / password,
make sure you pass those -e args.

```bash
#
# In below example, hosts are accessed as 'ubuntu' user, using /path/to/ssh/key file.
# Additionally the jenkins has already been installed and anonymous access disabled.
#
export ANSIBLE_HOST_KEY_CHECKING="False"
ansible-playbook \
    -i hosts.ini \
    --key-file /path/to/ssh/key                      `# change to your key` \
    -e "ansible_user=ubuntu"                         `# change to your remote user` \
    -e "ansible_python_interpreter=/usr/bin/python3" `# change to your desired python` \
    -e "jenkins_url_external=http://foo.com"         `# change to your PUBLIC ip or dns` \
    -e "jenkins_admin_username=admin"                `# if jenkins security enabled, provide user` \
    -e "jenkins_admin_password=secret"               `# if jenkins security enabled, provide pass` \
        site.yml
```

## DEPLOYED CONFIGURATION

### platform

Sets up swarm managers and workers (one host allocated as primary) as
a true cluster.

Configure hosts.ini to scale. Also change the docker daemon opts
to use tcp:// instead of unix socket. (See roles/common/defaults/main.yml)

Default: single host, acting as primary manager.

### installed components

* docker - default: latest CE
    * editions, channels and version pinning are all supported)

* docker-compose - default: latest
    * pip-installed in virtual-env to avoid clobbering system python

* pip `docker` package - default: latest
    * to support installation of ansible `docker_*` modules

* jenkins 2.x LTS with vendor-recommended list of plugins.

### system cfg

* adds list of users to docker group, to avoid need for root access
    * default: ubuntu

* configures docker daemon options and env vars
    * default: logs to journalctl, uses local unix socket (change to tcp for cluster)

* scheduled docker system cleanup.
    * default: runs every Sunday at midnight.

* scheduled jenkins workspace cleanup.
    * default: runs nightly.

* runs docker registry login credentials

## ROLE CONFIGURATION

* [common](roles/common/README.md)
    - docker, docker-compose, swarm cluster, python3, virtual-env

* [jenkins2](roles/jenkins2/README.md)
    - java, jenkins 2.x

#### PYTHON-DOCKER

This role uses `docker_login` to login to a Docker registry, but you may also
use the other `docker_*` modules in your own roles. They are not going to work
unless you instruct Ansible to use this role's Virtualenv.

At either the inventory, playbook or task level you'll need to set
`ansible_python_interpreter: "/usr/bin/env python-docker"`. This works because
this role symlinks the Virtualenv's Python binary to `python-docker`.

You can look at this role's `docker_login` task as an example on how to do it
at the task level.

## SUPPORTED OSes

- Ubuntu 16.04 LTS (Xenial)
- Ubuntu 18.04 LTS (Bionic)
- Debian 9 (Stretch)

