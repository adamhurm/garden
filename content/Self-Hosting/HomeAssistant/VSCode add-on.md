---
created: 2025-07-17T12:12
updated: 2025-07-21T18:39
tags:
  - homeassistant
  - vscode
  - proxmox
  - shareable
---
- https://github.com/hassio-addons/addon-vscode

This add-on provides a VSCode instance accessible within the HomeAssistant Web UI. This is really nice for editing HomeAssistant's [configuration.yaml](https://www.home-assistant.io/docs/configuration/) file which cannot usually be edited via web UI. This solution is also recommended by [official HomeAssistant documentation](https://www.home-assistant.io/common-tasks/os/#installing-and-using-the-visual-studio-code-vsc-add-on).

Additional context: I run HomeAssistant inside a Proxmox VM and expose externally via newt+pangolin. I want to limit the network attack surface as much as possible so I disable SSH and add firewall rules. On the rare occasion that I need to access Home Assistant CLI, I go to the Console feature in PVE Web UI (noVNC). This "shell-less" approach makes more sense to me because I can also disable VSCode add-on when not in active use.

