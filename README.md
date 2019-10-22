# github_version-ansible_plugin

This collection includes a single lookup plugin that returns the latest tagged release version of a public Github repository.
A future version will support a Github token as an environment variable to work against private repositories.

## Usage

Install this collection locally:

```bash
ansible-galaxy collection install artis3n.github_version -p ./collections
```

Then you can use the lookup plugin in your playbooks:

```yaml
- name: Ansible | Get latest release
  set_fact:
    ansible_version: "{{ lookup('github_version', 'ansible/ansible')[1:] }}
```

Note: `[1:]` is used to strip out the `v` in the version tag, e.g. `v1.1.0` becomes `1.1.0`.

Here's a longer example to demonstrate the benefit of this plugin to download the latest released Terraform version by validating its checksum:

```yaml
- name: Terraform | Get latest release
  set_fact:
    terraform_version: "{{ lookup('github_version', 'hashicorp/terraform')[1:] }}"

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

## Development

### Contributing

This repository welcomes contributions. Please fork this repository and file a pull request to the master branch. Your changes must include tests in the `tests/` directory.

### Publishing a new version

This repository uses [Github Actions][] to build and publish a new version artifact to [Ansible Galaxy][] upon a merge to master.

[github actions]: https://help.github.com/en/github/automating-your-workflow-with-github-actions/about-github-actions
[ansible galaxy]: https://galaxy.ansible.com/
