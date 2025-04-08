# Codeberg Release Download Manager
Easily download and extract latest source releases from Codeberg repositories.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_codeberg/blob/main/meta/main.yml)

[collections/roles](https://github.com/r-pufky/ansible_codeberg/blob/main/meta/requirements.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_codeberg/tree/main/defaults/main/)

### Generated Variables
After successful execution the following variables are available for further
manipulation of extracted release during the same play (standard role variable
scope):

 Variable                  | type | Description
---------------------------|------|-----------------------------------------
 _codeberg_target          | str  | Full version release tag.
 _codeberg_target_url      | str  | Download url for target release.
 _codeberg_archive         | str  | Local versioned archive location.
 _codeberg_dir             | str  | Local versioned extract location.
 _codeberg_remote_metadata | dict | Release metadata for requested version.
 _codeberg_changed         | bool | True if new archive extracted/installed.

## Dependencies
Part of the [r_pufky.srv](https://github.com/r-pufky/ansible_collection_srv)
collection.

## Example Playbook
This role is intended to be called multiple times in other roles with most
options being set per repository release being downloaded. Files may optionally
be migrated to new installations.

group_vars/all/main.yml
``` yaml
# Accessible to all hosts which will consume this role.
codeberg_personal_access_token: '{ACCESS TOKEN FROM CODEBERG}'
```

roles/my_custom_role/tasks/task.yml
``` yaml
- name: 'download and extract latest release'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.codeberg'
  vars:
    codeberg_repo_owner: 'forgejo'
    codeberg_repo_repo: 'forgejo'
    codeberg_repo_release: 'latest'
    codeberg_repo_archive_type: 'tar.gz'
    codeberg_file_owner: 'forgejo'
    codeberg_file_group: 'forgejo'
    codeberg_extract_dest: '/opt/forgejo'
    codeberg_extract_mode: 'a-st,o-rwx'
    codeberg_extract_extra_opts: '--strip-components=1'
    codeberg_extract_symlink_target: 'opt/forgejo/latest'
    codeberg_extract_migrate_files: ['config.db']
    codeberg_extract_remove_files:
      - 'README.md'
```

Binary releases may be explicitly specified for repositories which release
source and binaries:

roles/my_custom_role/tasks/task.yml
``` yaml
- name: 'download and extract latest BINARY release'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.codeberg'
  vars:
    codeberg_binary_enable: true
    codeberg_binary_specifier: 'v'
    codeberg_binary_url: '{VERSION}/Binary.{BARE_VERSION}.x86-64.tar.gz'
    codeberg_repo_owner: 'forgejo'
    codeberg_repo_repo: 'forgejo'
    codeberg_repo_release: 'latest'
    codeberg_repo_archive_type: 'xz'
    codeberg_file_owner: 'forgejo'
    codeberg_file_group: 'forgejo'
    codeberg_extract_dest: '/opt/forgejo'
    codeberg_extract_mode: 'a-st,o-rwx'
    codeberg_extract_extra_opts: '--strip-components=1'
    codeberg_extract_symlink_target: 'opt/forgejo/latest'
    codeberg_extract_remove_files:
      - 'README.md'
```

## Development
Configure [environment](https://github.com/r-pufky/ansible_collection_srv/blob/main/docs/dev/environment/README.md)

Run all unit tests:
``` bash
molecule test --all
```

Run integration tests against live API:
```
molecule test -s live_api_test -- -v -e 'codeberg_testing_enable=true,codeberg_personal_access_token={TOKEN}'
```

### Testing in other Roles
Testing may be toggled on the role to facilitate testing other roles which
consume this role, removing the need to hammer the Codeberg REST API or
download archive files. Never rely on this for live code. Effectively this is a
mock.

group_vars/archives:
```
project_v1.0.0.tar.gz
project_v1.0.1.tar.gz
```

roles/my_custom_role/molecule/molecule.yml
``` yaml
provisioner:
  inventory:
    group_vars:
      all:
        codeberg_testing_enable: true
        codeberg_testing_versions:
          - 'v1.0.0'
          - 'v1.0.1'
        codeberg_testing_archives:
          - 'group_vars/archives/project_v1.0.0.tar.gz'
          - 'group_vars/archives/project_v1.0.1.tar.gz'
```
When the role is called during testing, the first execution of the role will
return `v1.0.0` and the Codeberg download will be simulated by synchronizing the
specified archive from the ansible controller to the testing node.

The Second execution of the role will return `v1.0.1` and the Codeberg download
will be simulated by synchronizing the specified archive from the ansible
controller to the testing node.

### Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_codeberg/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [Codeberg gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)
