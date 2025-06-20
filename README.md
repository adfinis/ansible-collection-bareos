# Ansible Collection - adfinis.bareos

A collection of roles to install and configure [Bareos](https://www.bareos.com).


## Update standalone role(s) and trigger Dependabot manually
Insights > Dependency graph > Dependabot > (.gitmodules) Recent update jobs > Check for updates

## Using roles in this collection.

1. Install the collection:

```shell
ansible-galaxy collection install adfinis.bareos
```

You can also list a collection in `requirements.yml`:

```yaml
---
collections:
  - name: adfinis.bareos
```

2. Include roles in your playbooks:

```yaml
---
- name: Setup Bareos Director
  hosts: directors
  roles
    - adfinis.bareos.repository
    - adfinis.bareos.bareos_dir

- name: Setup Bareos Filedaemons
  hosts: filedaemons
  roles
    - adfinis.bareos.repository
    - adfinis.bareos.bareos_fd
```

## Using playbooks in this collection
The collection includes multiple playbooks to use with the individual roles. Playbooks from collections can be called by their FQCN directly.
The playbooks aid to minimize the amount of necessary configuration and try to declare as much as possible in the form of defaults. In most scenarios stuff like client settings, backup file sets/excludes, retention are universally applicable for all (or most) clients.
Therefore the playbooks rely heavly on group_vars, as a lot of data has to be shared between the Bareos components.
For example: For every client/FD there needs to be some shared config data on both the Director (`/etc/bareos/bareos-dir.d/clients/<client fqdn>`) and the FD (`/etc/bareos/bareos-fd.d/<director>`) so a connection between them is possible.

The playbooks try to ease the management of lots of backup clients and jobs, by adding default values that are applied for all hosts.
These defaults can be applied in the from of group vars (`group_vars/all/defaults.yml`):
``` yaml
# list has to be defined here, to be able to add jobs later
bareos_dir_jobs: []

job_defaults:
  name: Daily_Incremental
  pool: Pool_Daily
  type: Backup
  messages: Messages
  jobdefs: JobDef_StandardIncremental
  schedule: Daily2300_Incremental
  storage: "Storage_bareosStorageDaemon.example.com"

# list has to be defined here, to be able to add clients later
bareos_dir_clients: []

client_defaults:
  address: ""  # leave empty as filedaemons initiate TCP connection to director
  password: "{{ hostvars['bareos-director']['director_password'] }}"
  maximum_concurrent_jobs: 3
  enabled: true
  connection_from_fd: true
  connection_from_dir: false
  heartbeat: 60
```

All values can be overwritten by other group vars or individual host vars.

The following playbooks are supported:

### adfinis.bareos.manage_clients_playbook
Playbook will setup the Bareos Filedaemon on all clients in the host group `filedaemons` and will deploy a Bareos client resource with the settings from `client_defaults` on the Director for each host.

Supported tags:
  - `manage_clients::setup` (Only sets up dhe FDs, without adding them on the Director)
  - `mange_clients::deployment` (Deploys the configuration for each FD on the Director)

``` bash
ansible-playbook adfinis.bareos.manage_clients_playbook -i inventory
```

### adfinis.bareos.manage_jobs_playbook
Playbook will deploy a dedicated backup job on the Bareos Director for every host in the host group `filedaemons` with the settings from `job_defaults`.
Multiple jobs per client can be configured in the clients `host_vars`.
Example for a host with multiple backup jobs (`host_vars/db-host.example.com/jobs.yml`):

``` yaml
 ---
 
 bareos_fd_jobs:
   - name: db-host.example.com_Daily_Incremental
     client: db-host.examle.com
     jobdefs: JobDef_StandardIncremental
     pool: Pool_Daily
     schedule: Daily2200_Incremental
     storage: example-bareos-storagedaemon-01
     messages: Daily_Messages
 
   - name: db-host.example.com_PostgreSQL
     client: db-host.examle.com
     jobdefs: JobDef_PostgreSQL_Backup
     pool: Pool_Daily
     schedule: Daily2200_Incremental
     storage: example-bareos-storagedaemon-01
     messages: Daily_Messages
```

``` bash
ansible-playbook adfinis.bareos.manage_jobs_playbook -i inventory
```

### adfinis.bareos.fetch_encryption_keys
Playbook fetches the individual Filedaemon encryption keys, so that they can be stored off-site.

``` bash
ansible-playbook adfinis.bareos.fetch_encryption_keys -i inventory --check
```

### adfinis.bareos.manage_director_playbook
Can be used to call bareos_dir role in the scope of a client group, without the need to add the director to the client group.
Requires `-l/--limit` parameter.

``` bash
ansible-playbook adfinis.bareos.manage_director_playbook -i inventory --limit webservers --check
```

### adfinis.bareos.remove_clients_playbook
Can be used to remove the bareos backup client and files from a client and then disable the client resource(s) on the director

Disabled client/job resources on the director need to be cleaned up manually!

Clients need to be part of the group filedaemons to be targeted by the playbook.
Make sure to call the playbook with -l/--limit so not all clients are removed. Always use check mode first!

``` bash
ansible-playbook adfinis.bareos.remove_clients_playbook -i inventory --limit removal-client --check
```

Supported tags:
  - `cleanup_clients` just uninstalls the bareos FD client without touching the director
  - `cleanup_director` just disables all client resources within the ansible_play_hosts scope
  - `cleanup_repo` just removes all bareos repository files on the client
