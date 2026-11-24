# Ansible-Koolitus

Ansible training lab. A Vagrant file builds a small virtual machine environment (one Ansible control node and two managed servers), and the `ansible_data` directory contains the playbooks, roles and Docker Compose files used in the exercises.

## Lab environment

`Vagrantfile` defines these VirtualBox machines:

| VM | Box | IP (private network) | RAM | CPUs | Role |
|---|---|---|---|---|---|
| `ansible` | `generic/ubuntu2204` | 10.11.12.21 | 6 GB | 4 | Control node. Git and Ansible are installed automatically |
| `ubuntu` | `generic/ubuntu2204` | 10.11.12.22 | 2 GB | 2 | Managed server |
| `oracle` | `generic/oracle9` | 10.11.12.24 | 2 GB | 2 | Managed server (Oracle Linux 9) |
| `debian` | `generic/debian11` | 10.11.12.23 | 2 GB | 2 | Managed server, **commented out** in the Vagrantfile |

Every VM has a bridged network (`public_network`) and a private network `ansible_lab` for communication between the machines. The directory `./ansible_data` is mounted on the control node as `/vagrant_data`.

## Repository structure

```
.
├── Vagrantfile
└── ansible_data/
    ├── inventory                 Inventory with plain-text credentials
    ├── playbook.yml              First playbook (ping, facts)
    ├── playbooks/
    │   ├── inventory             Inventory using variables (vmuser, vmpass)
    │   ├── group_vars/all.yml    Encrypted variables (Ansible Vault)
    │   ├── hostname.yml          Playbook for the hostname role
    │   ├── http_check.yml        Playbook for the http_check role
    │   ├── webserver.yml         Playbook for the webserver role
    │   └── roles/
    │       ├── hostname/
    │       ├── http_check/
    │       └── webserver/
    ├── DOCKER/
    │   ├── docker-compose.yml    To-do app with MySQL
    │   └── portainer/docker-compose.yml
    └── GIT/                      Git exercise projects
```

## Playbooks

### `playbook.yml`

Runs on the group `virtualmachines` and:

1. Pings the hosts
2. Prints "Hello world" only on Debian hosts
3. Prints the operating system and its major version

### Roles in `playbooks/`

| Playbook | Role | What it does |
|---|---|---|
| `hostname.yml` | `hostname` | Writes `/home/vagrant/hostname` from the template `test.j2` (host name, OS and version). Copies `test.txt`, an Ansible Vault encrypted file, to `/home/vagrant/` |
| `http_check.yml` | `http_check` | Sends an HTTP GET to `http://<ansible_host>`, accepts status 200 or 403 and prints the `Server` header |
| `webserver.yml` | `webserver` | Installs and starts Apache: `httpd` with `yum` on Oracle Linux, `apache2` with `apt` on Ubuntu and Debian |

All three playbooks run with `become: true` on all hosts of the inventory.

### Ansible Vault

`playbooks/group_vars/all.yml` and `roles/hostname/files/test.txt` are encrypted with Ansible Vault. The vault password is **not** in the repository. The inventory `playbooks/inventory` uses the variables `vmuser` and `vmpass`, which are expected to be defined in the encrypted `all.yml`, so these playbooks must be run with the vault password.

## Docker files

| File | Contents |
|---|---|
| `DOCKER/docker-compose.yml` | To-do app: `node:18-alpine` on port 3000 and `mysql:8.0` with a health check. The app code is mounted from `/GIT/getting-started-app` |
| `DOCKER/portainer/docker-compose.yml` | Portainer CE 2.21.4 on ports 9443 (HTTPS) and 8000, with access to the Docker socket |

The directory `GIT/` contains the Git exercise projects `my-project` (GitLab template README, remote on `http://10.11.12.21`) and `to-do-docker-compose`.

## Prerequisites

- [VirtualBox](https://www.virtualbox.org/)
- [Vagrant](https://www.vagrantup.com/)
- About 10 GB of free RAM and 8 CPU cores for all three VMs
- `sshpass` on the control node, because the inventories use password authentication:

```bash
sudo apt-get install -y sshpass
```

## Usage

Start the VMs and log in to the control node:

```bash
vagrant up
vagrant ssh ansible
cd /vagrant_data
```

Run the first playbook:

```bash
ansible-playbook -i inventory playbook.yml
```

Run the role playbooks (they need the vault password):

```bash
cd playbooks
ansible-playbook -i inventory --ask-vault-pass hostname.yml
ansible-playbook -i inventory --ask-vault-pass webserver.yml
ansible-playbook -i inventory --ask-vault-pass http_check.yml
```

Run `webserver.yml` before `http_check.yml`, so there is a web server to check.

Start the Docker examples (Docker must be installed on the machine):

```bash
cd DOCKER && docker compose up -d
cd portainer && docker compose up -d
```

Remove the lab:

```bash
vagrant destroy -f
```

## Known issues

- The inventories list the host `debian` (10.11.12.23), but the Vagrant machine `debian` is commented out. Playbooks fail on this host with "unreachable". Uncomment the `debian` block in the `Vagrantfile` or remove the host from the inventories.
- The inventory `ansible_data/inventory` contains plain-text passwords (`vagrant`). These are the default Vagrant credentials and acceptable for a local lab, but do not copy this pattern to real servers. Use Ansible Vault or SSH keys.
- `docker-compose.yml` contains the MySQL root password `secret` in plain text. Use this only for the exercise.
- `sshpass` is not installed by the Vagrant provisioning, so it has to be installed by hand (see Prerequisites).
- On the first SSH connection Ansible asks to confirm the host key. For the lab you can disable this with `export ANSIBLE_HOST_KEY_CHECKING=False`.
- The Vagrantfile does not install Docker, so the Docker Compose files need Docker installed first.
- The directory `/GIT/getting-started-app` required by `DOCKER/docker-compose.yml` is not part of this repository.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
