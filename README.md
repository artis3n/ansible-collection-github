# github_version

[![Build Status](https://img.shields.io/endpoint.svg?url=https%3A%2F%2Factions-badge.atrox.dev%2Fartis3n%2Fgithub_version-ansible_plugin%2Fbadge&style=flat)](https://actions-badge.atrox.dev/artis3n/github_version-ansible_plugin/goto)

You can find this collection on Ansible Galaxy [here](https://galaxy.ansible.com/artis3n/github).

## Usage

Install this collection locally:

```bash
ansible-galaxy collection install artis3n.github_version
```

This installs to the first location in your [`ANSIBLE_COLLECTION_PATHS`](https://docs.ansible.com/ansible/devel/reference_appendices/config.html#collections-paths), which by default is `~/.ansible/collections`. You can modify the installation path with `-p`:

```bash
ansible-galaxy collection install artis3n.github_version -p collections/
```

Then you can use the lookup plugin in your playbooks. Yes, it is verbose and gross and I'm [looking into it](https://github.com/artis3n/github_version-ansible_plugin/issues/22). Ansible 2.9 doesn't seem to expose a way to shorten the absolute path invocation of a lookup plugin in a collection.

```yaml
- name: Ansible | Get latest release
  set_fact:
    ansible_version: "{{ lookup('artis3.github_version.github_version', 'ansible/ansible')[1:] }}
```

Note: `[1:]` is used to strip out the `v` in the version tag, e.g. `v1.1.0` becomes `1.1.0`.

<br>

<details><summary>Here's a longer example to demonstrate the benefit of this plugin to download the latest released Terraform version by validating its checksum:</summary>

```yaml
- name: Terraform | Get latest release
  set_fact:
    terraform_version: "{{ lookup('artis3.github_version.github_version', 'hashicorp/terraform')[1:] }}"

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

## Contents

This collection includes the following items related to interacting with GitHub:

### Plugins

#### Lookup plugins

##### latest_release

This lookup plugin retrieves the latest tagged release version of a public Github repository.

A future version will support a Github token as an environment variable to work against private repositories.

## Development

### TODO

- [ ] Tests :grimacing:
- [ ] Support Github tokens for private repository lookups

### Contributing

This repository welcomes contributions. Please fork this repository and file a pull request to the master branch. Your changes must include tests in the `tests/` directory. Run your code through `flake8` and `black` before creating the pull-request.

### Publishing a new version

This repository uses [Github Actions][] to build and publish a new version artifact to [Ansible Galaxy][] upon a merge to master.

[github actions]: https://help.github.com/en/github/automating-your-workflow-with-github-actions/about-github-actions
[ansible galaxy]: https://galaxy.ansible.com/
