ansible-role-hawser
===================

Installs, configures and updates [Hawser](https://github.com/Finsys/hawser),
the remote Docker agent for [Dockhand](https://github.com/Finsys/dockhand), as
a systemd service on Debian/Ubuntu hosts.

Rather than piping the upstream `install.sh` into bash, the role downloads the
release tarball for the host's architecture (amd64, arm64, arm), verifies it
against the release's `checksums.txt`, and installs the same systemd unit
upstream ships. Docker must already be installed.

What it does
------------

- `hawser_state: present` (default)
  1. Installs `curl` and `ca-certificates`.
  2. Resolves the version (`latest` asks the GitHub API; or a pinned release).
  3. Downloads and installs the binary only when the version recorded in
     `/etc/hawser/.installed-version` differs, so re-runs are idempotent and
     bumping `hawser_version` (or re-running with `latest`) is the update path.
  4. Writes `/etc/hawser/config` (mode 0600) and the systemd unit, restarts
     the service on change, and waits for `/_hawser/health`.
- `hawser_state: absent` stops and removes the service, binary and
  `/etc/hawser`. The stacks directory is left in place.

Modes
-----

- **Standard** (default): Dockhand connects to the agent on `hawser_port`
  (2376). A `hawser_token` is required unless `hawser_bind_address` is loopback.
- **Edge**: set `hawser_dockhand_server_url`
  (`wss://<dockhand>/api/hawser/connect`) and `hawser_token`; the agent dials
  out. Consider `hawser_bind_address: 127.0.0.1`.

Role variables
--------------

See `defaults/main.yml` for the full list. The important ones:

| Variable | Description |
|---|---|
| `hawser_state` | `present` or `absent` (default `present`) |
| `hawser_version` | `latest` (default) or a release such as `0.2.50` |
| `hawser_token` | Shared secret with Dockhand (store it in Infisical/vault) |
| `hawser_dockhand_server_url` | Enables edge mode when set |
| `hawser_port`, `hawser_bind_address` | Listener (standard mode / healthcheck) |
| `hawser_tls_cert`, `hawser_tls_key` | Standard-mode TLS, paths on the host |
| `hawser_ca_cert`, `hawser_tls_skip_verify` | Edge-mode TLS to Dockhand |
| `hawser_stacks_dir` | Compose stacks directory (default `/data/stacks`) |
| `hawser_extra_config` | Extra `KEY: value` pairs for the config file |
| `hawser_no_log` | Hide token-handling tasks (default `false`) |

Example playbook
----------------

    - hosts: hawser_agents
      become: true
      roles:
        - role: hawser
          vars:
            hawser_token: "{{ vault_hawser_token }}"

Then add the host in Dockhand under Settings -> Environments with the same token.

Installation
------------

    roles:
      - name: hawser
        src: https://github.com/dpisano/ansible-role-hawser.git
        scm: git
        version: v1.0.0

Testing
-------

`molecule test` (Docker driver) installs an older release, updates it, checks
idempotence and verifies the health endpoint, using a stub Docker API.
