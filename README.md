# Ansible role - os_upgrade

Apply Debian and Ubuntu package updates on demand, in a maintenance window, while self-managed packages keep their
own upgrade procedure.

Galaxy FQCN: `bouola.os_upgrade`

Some packages must not move with a generic `apt upgrade`, for example a GitLab or GitLab Runner version pinned by
its own playbook. The role holds the packages you list for the duration of the run, upgrades everything else, then
releases them, so their own procedure can still install the version it pins. A package that was already held before
the run stays held. By default nothing is held.

When an update needs a reboot (`/var/run/reboot-required`), the role reports it. It reboots the host only when
`os_upgrade_reboot` is `true`.

## Requirements

- Debian 12 or 13, Ubuntu 22.04 or 24.04.
- Ansible 2.15 or later on the control node.

## Role variables

| Variable | Default | Description |
|---|---|---|
| `os_upgrade_type` | `"safe"` | `safe` never installs or removes a package; `full` is `apt full-upgrade`; `dist` is `apt-get dist-upgrade`. |
| `os_upgrade_held_packages` | `[]` | Packages held during the upgrade and released afterwards. Packages that are not installed are ignored. |
| `os_upgrade_reboot` | `false` | Reboot the host when the upgrade requires it. |
| `os_upgrade_reboot_timeout` | `600` | Seconds to wait for the host to come back after a reboot. |

Upgrading Docker Engine (`docker-ce`, `containerd.io`) restarts every container of the host. Hold it only if a
playbook pins its version; otherwise run the role in the window where those containers may restart.

## Example playbook

Run the role only on demand: with the `never` tag, a normal run skips it, and `--tags os_upgrade` selects it.

```yaml
- name: "Operating System Configuration"
  hosts: all
  tasks:
    - name: "Apply the Debian updates"
      ansible.builtin.import_role:
        name: "bouola.os_upgrade"
      tags:
        - never
        - os_upgrade
```

```yaml
# group_vars/gitlab.yml: GitLab is upgraded by its own playbook
os_upgrade_held_packages:
  - "gitlab-ce"
  - "gitlab-runner"
```

`os_upgrade_reboot` and `os_upgrade_reboot_timeout` also accept the strings that `-e key=value` passes, for
example `-e os_upgrade_reboot=true`.

Preview the updates without installing them:

```bash
ansible-playbook site.yml --tags os_upgrade --check --diff
```

In check mode the role still holds the listed packages and releases them at the end, so the preview leaves them out
like the real run. It is the only change a check-mode run makes, and the dpkg state is the same afterwards.

## Tags

| Tag | Tasks |
|---|---|
| `role-os-upgrade-validate` | Input and operating system checks |
| `role-os-upgrade-packages` | Hold, upgrade and release |
| `role-os-upgrade-reboot` | Reboot check, report and optional reboot |

## Testing

```bash
molecule test -s default
```

The scenario serves two versions of two test packages from a local apt repository. A check-mode run must preview
the upgrade of the free package and leave the held one out; the real run must then upgrade the free package and
keep the held one. The scenario also holds a package before the run, checks that it stays held, that the other
listed packages are released, and that a pending reboot is reported without rebooting.

## License

MIT
