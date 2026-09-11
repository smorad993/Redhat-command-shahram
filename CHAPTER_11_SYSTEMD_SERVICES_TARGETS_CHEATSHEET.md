# CHAPTER 11 CHEATSHEET: SYSTEMD SERVICES & TARGETS

Quick RHCSA-style reference for managing services, boot behavior, targets, and custom systemd unit files in RHEL 9.

---

## Core Ideas

| Concept | Meaning | Exam Tip |
|---|---|---|
| `systemd` | First process after boot, PID 1 | Controls services, targets, timers, sockets, and more |
| Unit | A systemd configuration object | Common types: `.service`, `.target`, `.timer` |
| `.service` | Background service/daemon | Examples: `sshd.service`, `httpd.service` |
| `.target` | Group of units representing a system state | Similar to old runlevels |
| `.timer` | Scheduled systemd job | Alternative to cron for some tasks |

---

## Live Actions vs Boot Persistence

| Type | Commands | Effect | Survives Reboot? |
|---|---|---|---|
| Live actions | `start`, `stop`, `restart`, `reload` | Changes current running state | No |
| Boot persistence | `enable`, `disable` | Controls auto-start at boot | Yes |
| Combined | `enable --now` | Enables for boot and starts immediately | Yes |

Remember: `start` does not mean enabled, and `enable` does not necessarily mean running now.

---

## Service Management Commands

| Task | Command |
|---|---|
| Start service now | `sudo systemctl start httpd` |
| Stop service now | `sudo systemctl stop httpd` |
| Restart service | `sudo systemctl restart httpd` |
| Reload config without full restart, if supported | `sudo systemctl reload httpd` |
| Show service status | `systemctl status httpd` |
| Show full status lines | `systemctl status httpd -l` |
| Enable service at boot | `sudo systemctl enable httpd` |
| Disable service at boot | `sudo systemctl disable httpd` |
| Enable and start now | `sudo systemctl enable --now httpd` |
| Check boot enabled state | `systemctl is-enabled httpd` |
| Check current running state | `systemctl is-active httpd` |
| Prevent service from starting at all | `sudo systemctl mask httpd` |
| Remove mask | `sudo systemctl unmask httpd` |

---

## Status Keywords

| Output | Meaning |
|---|---|
| `active` | Service is currently running |
| `inactive` | Service is currently stopped |
| `failed` | Service tried to run but failed |
| `enabled` | Service starts automatically at boot |
| `disabled` | Service does not start automatically at boot |
| `masked` | Service is blocked from starting manually or automatically |

---

## Target Management

| Target | Purpose |
|---|---|
| `multi-user.target` | Text/server mode, networking enabled, no GUI |
| `graphical.target` | GUI mode; includes multi-user services |
| `rescue.target` | Single-user rescue mode for repair/troubleshooting |

| Task | Command |
|---|---|
| Show default boot target | `systemctl get-default` |
| Set text mode as default | `sudo systemctl set-default multi-user.target` |
| Set GUI mode as default | `sudo systemctl set-default graphical.target` |
| Switch current running target immediately | `sudo systemctl isolate multi-user.target` |

Exam note: `set-default` changes future boots. `isolate` changes the current running target immediately.

---

## Custom Service Unit Template

Create custom service files in:

```bash
/etc/systemd/system/name.service
```

Example:

```ini
[Unit]
Description=Discovery NRD Core Service
After=network.target

[Service]
Type=simple
ExecStartPre=/usr/local/bin/get-token.sh
EnvironmentFile=/run/nrd-token.env
ExecStart=/usr/local/bin/discovery-nrd-ccp
Restart=on-failure
TimeoutStartSec=180

[Install]
WantedBy=multi-user.target
```

---

## Unit File Sections

| Section | Purpose | Common Directives |
|---|---|---|
| `[Unit]` | Metadata and dependencies | `Description=`, `After=` |
| `[Service]` | How the service starts/runs | `Type=`, `ExecStart=`, `ExecStartPre=`, `EnvironmentFile=`, `Restart=` |
| `[Install]` | How service is enabled at boot | `WantedBy=multi-user.target` |

---

## After Editing a Unit File

Always reload systemd after creating or changing a unit file:

```bash
sudo systemctl daemon-reload
```

Then usually run:

```bash
sudo systemctl enable --now discovery-nrd.service
systemctl status discovery-nrd.service -l
```

---

## Troubleshooting Commands

| Task | Command |
|---|---|
| List failed units | `systemctl --failed` |
| Show matching active units | `systemctl list-units "discovery-*"` |
| View service status with full lines | `systemctl status service-name -l` |
| Reload systemd after unit changes | `sudo systemctl daemon-reload` |
| Restart service after config change | `sudo systemctl restart service-name` |

---

## Fast Exam Checklist

1. Need service running now? Use `systemctl start`.
2. Need service to start after reboot? Use `systemctl enable`.
3. Need both? Use `systemctl enable --now`.
4. Need default boot mode? Use `systemctl get-default` and `set-default`.
5. Need immediate target switch? Use `systemctl isolate`.
6. Created or edited a unit file? Always run `systemctl daemon-reload`.
7. Service will not start? Check `systemctl status -l` and `systemctl --failed`.
