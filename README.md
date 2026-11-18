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
| `plakar_user` | `plakar` | System account that owns the stores and runs Plakar. Created by the role, no login shell. It cannot read the key files. |
| `plakar_group` | `plakar` | Primary group of that account. |
| `plakar_home` | `/var/lib/plakar` | Home of that account; Plakar keeps its cache there. |
| `plakar_config_dir` | `{{ plakar_home }}/.config/plakar` | Plakar configuration of that account. The role writes `stores.yml` and `sources.yml` from `plakar_klosets` and `plakar_sources`, and creates an empty `destinations.yml` when it is missing. |
| `plakar_klosets` | `[]` | Kloset stores to create and declare. Each entry is a dict with `name` (unique), `path` and `passphrase`, for example `{name: main, path: /var/lib/plakar/store, passphrase: "{{ vault_main_passphrase }}"}`. Its key file is `/etc/credstore/<name>.key`, owned by root, `0600`, passed to the units through `LoadCredential=`. One check unit per store. Empty: the role ends after installation. |
| `plakar_sources` | `[]` | Sources to declare and back up. Each entry is a dict with `name` (unique), `location`, `kloset` (a `name` from `plakar_klosets`) and optionally `ignore`, a list of gitignore patterns, for example `{name: web, location: "sftp://backup@host.example/srv", kloset: main, ignore: ["cache/"]}`. One backup unit per source. A source removed from the list loses its units and files; its snapshots stay. |
| `plakar_ignore_dir` | `/etc/plakar/ignore` | One exclusion file per source, `<name>.ignore`, passed to `plakar backup -ignore-file`. |
| `plakar_libexec_dir` | `/usr/local/libexec/plakar` | Holds the script that reads the error count of the last snapshot after each backup. |
| `plakar_generate_ssh_key` | `false` | Generate an ed25519 key for the account; the role then sets `plakar_fact_ssh_public_key`. |
| `plakar_known_hosts` | `[]` | Host keys to trust, as a list of `{name, key}` dicts for the `known_hosts` module, for example `{name: host.example, key: "host.example ssh-ed25519 AAAA..."}`. |
| `plakar_token` | `""` | plakar.io token; when set, the account logs in with `plakar login -env`. |
| `plakar_plugins` | `[]` | Plugins to install with `plakar pkg add`, as `{name, version}` dicts, `version` optional, for example `{name: sftp}`. |
| `plakar_backup_schedule` | `daily` | `OnCalendar=` of `plakar-backup.timer`, which starts the backup of every source. |
| `plakar_check_schedule` | `weekly` | `OnCalendar=` of `plakar-check.timer`, which starts the check of every store. |

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
          - name: main
            path: /var/lib/plakar/store
            passphrase: "{{ vault_plakar_store_passphrase }}"
        plakar_sources:
          - name: web
            location: "sftp://backup@host.example/srv"
            kloset: main
```

The passphrase is the only way to read the store: keep a copy outside the managed host and the control node.

License
-------

MIT

Author Information
------------------

TheWolfPaws
