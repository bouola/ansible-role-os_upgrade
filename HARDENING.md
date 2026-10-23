# Introduction
In this document, a list of relevant settings for hardening os_upgrade is provided.
This document is non-exhaustive, however, it provides a solid base of security hardening
measures.

## Update sources

- The role installs what the configured apt sources offer. Keep the sources signed (`signed-by` keyrings) and
  limited to the distribution, its security repository and the vendor repositories you need.
- Run it regularly: a held package or a missed maintenance window leaves known vulnerabilities in place.

## Held packages

- A held package receives no security update from this role. Track each one in its own upgrade procedure, and
  review the list when a procedure is retired.

## Reboots

- Kernel and core library updates only take effect after a reboot. Plan it in the same maintenance window, or set
  `os_upgrade_reboot` to `true` for hosts that can restart unattended.
