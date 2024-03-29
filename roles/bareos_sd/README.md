# Ansible role [bareos_sd](#bareos_sd)

Install and configure [Bareos](https://www.bareos.com/) Storage Daemon.

|GitHub|GitLab|Downloads|Version|
|------|------|---------|-------|
|[![github](https://github.com/adfinis/ansible-role-bareos_sd/workflows/Ansible%20Molecule/badge.svg)](https://github.com/adfinis/ansible-role-bareos_sd/actions)|[![gitlab](https://gitlab.com/robertdebock-iac/ansible-role-bareos_sd/badges/master/pipeline.svg)](https://gitlab.com/robertdebock-iac/ansible-role-bareos_sd)|[![downloads](https://img.shields.io/ansible/role/d/)](https://galaxy.ansible.com/adfinis/bareos_sd)|[![Version](https://img.shields.io/github/release/adfinis/ansible-role-bareos_sd.svg)](https://github.com/adfinis/ansible-role-bareos_sd/releases/)|

## [Example Playbook](#example-playbook)

This example is taken from [`molecule/default/converge.yml`](https://github.com/adfinis/ansible-role-bareos_sd/blob/master/molecule/default/converge.yml) and is tested on each push, pull request and release.

```yaml
---
- name: Converge
  hosts: all
  become: yes
  gather_facts: yes

  roles:
    - role: adfinis.roles.bareos_sd
      bareos_sd_backup_configurations: yes
      bareos_sd_install_debug_packages: yes
      bareos_sd_devices:
        - name: "FileStorage"
          description: "File device. A connecting Director must have the same Name and MediaType."
          media_type: "File"
          archive_device: "/var/lib/bareos/storage"
          label_media: yes
          random_access: yes
          automatic_mount: yes
          removable_media: no
          always_open: no
          maximum_concurrent_jobs: 1
        - name: "disabled-device"
          enabled: no
      bareos_sd_directors:
        - name: bareos-dir
          password: "somepassword"
        - name: "disabled-director"
          enabled: no
      bareos_sd_messages:
        - name: "Standard"
          description: "Send relevant messages to the Director."
          director:
            server: bareos-dir
            messages:
              - all
              - "!skipped"
              - "!restored"
          append:
            file: "/var/log/bareos/bareos.log"
            messages:
              - all
              - "!skipped"
              - "!terminate"
          console:
            - all
            - "!skipped"
            - "!saved"
        - name: "disabled-message"
          enabled: no
      bareos_sd_s3_profiles:
        - name: exoscale
          host: "sos.exo.io:443"
          use_https: yes
          access_key: "SomeAPIKey"
          secret_key: "SomeSecret"
          pricing_dir: ""
          backend: "s3"
          aws_auth_sign_version: 4
          aws_region: "ch-gva-2"
```

The machine needs to be prepared. In CI this is done using [`molecule/default/prepare.yml`](https://github.com/adfinis/ansible-role-bareos_sd/blob/master/molecule/default/prepare.yml):

```yaml
---
- name: Prepare
  hosts: all
  become: yes
  gather_facts: no

  roles:
    - role: robertdebock.bootstrap
    - role: adfinis.roles.bareos_repository
      bareos_repository_enable_tracebacks: yes
```

These roles are provided as is, without warranty of any kind. Use it at your own risk.

## [Role Variables](#role-variables)

The default values for the variables are set in [`defaults/main.yml`](https://github.com/adfinis/ansible-role-bareos_sd/blob/master/defaults/main.yml):

```yaml
---
# defaults file for bareos_sd

# The Storage Daemon has these configuration parameters.

# Backup the configuration files.
bareos_sd_backup_configurations: no

# Install debug packages. This requires the debug repositories to be enabled.
bareos_sd_install_debug_packages: no

# The hostname of the Storage Daemon.
bareos_sd_hostname: "{{ inventory_hostname }}"

# The messages configuration to use.
bareos_sd_message: "Standard"

# The amount of jobs to run concurrently.
bareos_sd_max_concurrent_jobs: 20

# Enable TLS.
bareos_sd_tls_enable: yes

# Verify the peer.
bareos_sd_tls_verify_peer: no

# A list of devices to configure.
bareos_sd_devices: []

# A list of directors to connect to.
bareos_sd_directors: []

# A list of messages to send to the director.
bareos_sd_messages: []

# A list of S3 profiles to configure.
bareos_sd_s3_profiles: []
```

## [Requirements](#requirements)

- pip packages listed in [requirements.txt](https://github.com/adfinis/ansible-role-bareos_sd/blob/master/requirements.txt).

## [State of used roles](#state-of-used-roles)

The following roles are used to prepare a system. You can prepare your system in another way.

| Requirement | GitHub | GitLab |
|-------------|--------|--------|
|[robertdebock.bootstrap](https://galaxy.ansible.com/adfinis/robertdebock.bootstrap)|[![Build Status GitHub](https://github.com/adfinis/robertdebock.bootstrap/workflows/Ansible%20Molecule/badge.svg)](https://github.com/adfinis/robertdebock.bootstrap/actions)|[![Build Status GitLab](https://gitlab.com/robertdebock-iac/robertdebock.bootstrap/badges/master/pipeline.svg)](https://gitlab.com/robertdebock-iac/robertdebock.bootstrap)|
|[adfinis.bareos_repository](https://galaxy.ansible.com/adfinis/bareos_repository)|[![Build Status GitHub](https://github.com/adfinis/ansible-role-bareos_repository/workflows/Ansible%20Molecule/badge.svg)](https://github.com/adfinis/ansible-role-bareos_repository/actions)|[![Build Status GitLab](https://gitlab.com/robertdebock-iac/ansible-role-bareos_repository/badges/master/pipeline.svg)](https://gitlab.com/robertdebock-iac/ansible-role-bareos_repository)|

## [Context](#context)

This role is a part of many compatible roles. Have a look at [the documentation of these roles](https://adfinis.com/) for further information.

Here is an overview of related roles:
![dependencies](https://raw.githubusercontent.com/adfinis/ansible-role-bareos_sd/png/requirements.png "Dependencies")

## [Compatibility](#compatibility)

This role has been tested on these [container images](https://hub.docker.com/u/robertdebock):

|container|tags|
|---------|----|
|[Debian](https://hub.docker.com/r/robertdebock/debian)|bookworm, bullseye, buster|
|[EL](https://hub.docker.com/r/robertdebock/enterpriselinux)|7, 8, 9|
|[Fedora](https://hub.docker.com/r/robertdebock/fedora/)|37, 38|
|[opensuse](https://hub.docker.com/r/robertdebock/opensuse)|all|
|[Ubuntu](https://hub.docker.com/r/robertdebock/ubuntu)|jammy, focal|

The minimum version of Ansible required is 2.12, tests have been done to:

- The previous version.
- The current version.
- The development version.

If you find issues, please register them in [GitHub](https://github.com/adfinis/ansible-role-bareos_sd/issues).

## [License](#license)

[Apache-2.0](https://github.com/adfinis/ansible-role-bareos_sd/blob/master/LICENSE).

## [Author Information](#author-information)

[Adfinis](https://adfinis.com/)

