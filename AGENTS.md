# AGENTS.md

This file defines how agents should work in this repository.

These instructions apply repo-wide unless the user gives a more specific request.

## Intent

This is the `bouola.os_upgrade` Ansible role, published as open source. It applies Debian and Ubuntu package
updates on demand, in a maintenance window, and keeps self-managed packages (for example a GitLab version
pinned by its own playbook) on their own upgrade procedure.

The role is meant to run only when asked, for example from a play tagged `never` and `os_upgrade`. It is not a
replacement for unattended-upgrades and never runs as part of a normal deployment.

## Layout

- `tasks/`: `validate.yml`, `upgrade.yml` and `reboot.yml`, included from `main.yml`.
- `vars/main.yml`: the allowed upgrade types and the reboot flag file.

## Variables

- Public variables start with `os_upgrade_` and are all documented in `defaults/main.yml` and `README.md`.
- Private variables (loop variables, registered results, facts) start with `_os_upgrade_`. `.ansible-lint`
  enforces this for loop variables.
- Defaults are typed and truthful: every default must be implemented end to end. Do not add variables for a feature
  that is not implemented yet.
- Use YAML booleans `true` and `false`, never `yes` or `no`, and write lists as block lists.

## Task style

- Every task has a sentence-case `name:` in double quotes and uses FQCNs.
- Use modules instead of `command` whenever one exists. When `command` is needed, set `changed_when` explicitly.
- Validation uses `ansible.builtin.assert` with an explicit multiline `fail_msg: >-`.
- Every loop sets `loop_control.loop_var` and `loop_control.label`.
- Role tags are `role-os-upgrade-validate`, `role-os-upgrade-packages` and `role-os-upgrade-reboot`, applied
  through `include_tasks` with `apply.tags`.

## Safety

- Held packages are held for the duration of the run only and released in an `always` section, even when the
  upgrade fails. Holding and releasing run in check mode too (`check_mode: false`), so the preview matches the
  real run. A package that was held before the run must stay held: never release it.
- The default upgrade type is `safe`: it never installs or removes a package. `full` and `dist` are explicit
  choices of the user.
- Never reboot unless `os_upgrade_reboot` is true. The default only reports a pending reboot.

## Testing and validation

Before considering a change done, run what is available locally:

- `yamllint .`
- `ansible-lint`
- `molecule test -s default` when Docker access is available

Scenario checks belong in Ansible `molecule/*/verify.yml` playbooks. Containers cannot reboot, so the scenarios
keep `os_upgrade_reboot` false. If a scenario cannot run, say so in the final report.
