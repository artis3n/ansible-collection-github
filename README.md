# github_version

[![Build Status](https://img.shields.io/endpoint.svg?url=https%3A%2F%2Factions-badge.atrox.dev%2Fartis3n%2Fgithub_version-ansible_plugin%2Fbadge&style=flat)](https://actions-badge.atrox.dev/artis3n/github_version-ansible_plugin/goto)
![GitHub](https://img.shields.io/github/license/artis3n/ansible-collection-github)
![GitHub release (latest SemVer)](https://img.shields.io/github/v/release/artis3n/ansible-collection-github)
![GitHub followers](https://img.shields.io/github/followers/artis3n?style=social)
![Twitter Follow](https://img.shields.io/twitter/follow/artis3n?style=social)

You can find this collection on Ansible Galaxy [here](https://galaxy.ansible.com/artis3n/github).

## Usage

Install this collection locally:

```bash
ansible-galaxy collection install artis3n.github
```

This installs to the first location in your [`ANSIBLE_COLLECTIONS_PATHS`](https://docs.ansible.com/ansible/devel/reference_appendices/config.html#collections-paths), which by default is `~/.ansible/collections`. You can modify the installation path with `-p`:

```bash
ansible-galaxy collection install artis3n.github -p collections/
```

Then you can use the lookup plugin in your playbooks. Note that, unlike roles and modules imported via a collection, plugins seem to always require their full name.

```yaml
collections:
  - artis3n.github
tasks:
  - name: Ansible | Get latest release
    set_fact:
      ansible_version: "{{ lookup('artis3.github.latest_release', 'ansible/ansible')[1:] }}
```

Note: `[1:]` is used to strip out the `v` in the version tag, e.g. `v1.1.0` becomes `1.1.0`.

## Contents

This collection includes the following items related to interacting with GitHub:

### Plugins

#### Lookup plugins

##### latest_release

This lookup plugin retrieves the latest tagged release version of a public Github repository.

A future version will support a Github token as an environment variable to work against private repositories.

Example:

```yaml
tasks:
  - name: Ansible | Get latest release
    set_fact:
      ansible_version: "{{ lookup('artis3.github.latest_release', 'ansible/ansible')[1:] }}
```

<br>

<details><summary>Here's a longer example to demonstrate the benefit of this plugin to download the latest released Terraform version by validating its checksum:</summary>

```yaml
- name: Terraform | Get latest release
  set_fact:
    terraform_version: "{{ lookup('artis3.github.release_version', 'hashicorp/terraform')[1:] }}"

- name: Terraform | Ensure directory
  file:
    path: "{{ install_dir }}/terraform_{{ terraform_version }}"
    state: directory
  register: terraform_directory

- name: Terraform | Get hashes
  get_url:
    url: https://releases.hashicorp.com/terraform/{{ terraform_version }}/terraform_{{ terraform_version }}_SHA256SUMS
    dest: "{{ terraform_directory.path }}/SHA256SUMS"
  register: terraform_shas_file
  changed_when: false

- name: Terraform | Construct regex
  set_fact:
    terraform_sha_hash: "{{ '.*\\s\\sterraform_' + (terraform_version | regex_escape()) + '_linux_amd64\\.zip' }}"

- name: Terraform | Extract sha hash
  set_fact:
    # https://regex101.com/r/RS94Us/1
    terraform_sha_string: "{{ lookup('file', terraform_shas_file.dest) | regex_findall(terraform_sha_hash) | first }}"

- name: Terraform | Download
  get_url:
    url: https://releases.hashicorp.com/terraform/{{ terraform_version }}/terraform_{{ terraform_version }}_{{ os_short }}.zip
    dest: "{{ install_dir }}/terraform_{{ terraform_version }}.zip"
    checksum: sha256:{{ terraform_sha_string.split(' ')[0] }}
  register: terraform_download
```
</details>

## Development

Ansible has very strict expectations for where to look for a collection. There are [open issues][] against Ansible to change this behavior to make local testing easier. However for the time being, you must do the following.

You can modify the `ANSIBLE_COLLECTIONS_PATHS` environment variable to add custom paths for Ansible to search for collections, however collections **must** be in the following format _at the directory location specified in `ANSIBLE_COLLECTIONS_PATHS`_:

```text
collection/
├── ansible_collections/
│   ├── <namespace, e.g. artis3n>/
│   │   ├── <collection name, e.g. github>/
|   |   |   ├── plugins/
|   |   |   |   ├── lookup/
|   |   |   |   |   └── .../
|   |   |   └── .../
```

So, you _must_ clone this repository to a location under `collection/ansible_collections/` and modify `ANSIBLE_COLLECTIONS_PATHS` to include that absolute path in order for `molecule test` to find this collection.

You will notice that in `molecule/default/molecule.yml` I hard-code my personal directory in order to easily run molecule tests:

```yaml
ANSIBLE_COLLECTIONS_PATHS: "~/.ansible/collections:~/Nextcloud/Development/collections"
```
 
 I don't have a good way of writing something in code that can apply more generally due to the way Ansible currently handles collections so you'll likely need to override that to test this locally.

Alternatively, you can submit a draft pull request to this repository and a GitHub Actions workflow will trigger and run `molecule test` on your pull request.

### TODO

- [x] Tests :grimacing:
- [ ] Support Github tokens for private repository lookups

### Contributing

This repository welcomes contributions. Please fork this repository and file a pull request to the master branch. Your changes must include any appropriate tests under the `molecule/` directory. Read about how to use Molecule from the official documentation [here][molecule official] and from Jeff Geerling [here][geerlingguy molecule].

### Publishing a new version

This repository uses [Github Actions][] to build and publish new releases to [Ansible Galaxy][]. The action in question can be found [on the GitHub Marketplace][github marketplace].

[github actions]: https://help.github.com/en/github/automating-your-workflow-with-github-actions/about-github-actions
[ansible galaxy]: https://galaxy.ansible.com/
[github marketplace]: https://github.com/marketplace/actions/deploy-ansible-galaxy-collection
[geerlingguy molecule]: https://www.jeffgeerling.com/blog/2019/how-add-integration-tests-ansible-collection-molecule
[molecule official]: https://molecule.readthedocs.io/en/stable/
[open issues]: https://github.com/ansible/ansible/issues/60215
