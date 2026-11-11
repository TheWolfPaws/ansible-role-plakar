**plakar**
==============

An Ansible role to install [Plakar](https://plakar.io/) on Debian from the `.deb` package published with each GitHub release.

Requirements
------------

- Debian.
- Ansible Core 2.18 or newer.
- `python3-apt` on the managed host (`package_facts` and `apt` modules).
- Network access from the managed host to `github.com` and its release assets.
- When `plakar_version` is `latest`: network access from the control node to `api.github.com`.
- Privileged access to the managed host.

Role Variables
--------------

| Variable | Default | Description |
|---|---|---|
| `plakar_version` | `"latest"` | Release to install, without the leading `v` (for example `"1.1.8"`). `latest` resolves the newest release through the GitHub API and warns; pin a version afterwards. |
| `plakar_release_url` | `https://github.com/PlakarKorp/plakar/releases` | Base URL of the releases. Change it to use a mirror. |
| `plakar_architecture` | derived from `ansible_facts['architecture']` | Architecture suffix of the package (`amd64`, `arm64`). Fails on an unmapped architecture. |
| `plakar_download_dir` | `/tmp` | Where the package is downloaded before installation. It is deleted afterwards. |
| `plakar_user` | `plakar` | System account that owns the key files and the stores, and runs Plakar. Created by the role, no login shell. |
| `plakar_group` | `plakar` | Primary group of that account. |
| `plakar_home` | `/var/lib/plakar` | Home of that account; Plakar keeps its configuration and cache there. |
| `plakar_keyfile_dir` | `/etc/plakar` | Directory holding the key files, owned by `plakar_user`, created `0700`. |
| `plakar_klosets` | `[]` | Kloset stores to create. Each entry is a dict with `path`, `passphrase` and optionally `name`. Empty: the role ends after installation. |

Dependencies
------------

No additional Ansible roles are required.

Example Playbook
----------------

```yaml
---
- name: Install Plakar on the backup server
  hosts: backup_servers
  become: true

  roles:
    - role: plakar
      vars:
        plakar_version: "1.1.8"
        plakar_klosets:
          - path: /var/lib/plakar/store
            passphrase: "{{ vault_plakar_store_passphrase }}"
```

The passphrase is the only way to read the store: keep a copy outside the managed host and the control node.

License
-------

MIT

Author Information
------------------

TheWolfPaws
