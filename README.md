# Ansible Role: piaware

[![CI](https://github.com/tjg-homelab/ansible-role-piaware/actions/workflows/ci.yml/badge.svg)](https://github.com/tjg-homelab/ansible-role-piaware/actions/workflows/ci.yml)

Installs and manages **PiAware** (the [FlightAware](https://flightaware.com/adsb/piaware/)
ADS-B receiver stack: `dump1090-fa`, `piaware`, and `piaware-web`) on Debian 12
(Bookworm) and Debian 13 (Trixie).

FlightAware has not released official packages for Debian 13 Trixie. This role fills
that gap using the [abcd567a community apt repository](https://github.com/abcd567a/debian13),
and — per upstream recommendation — removes the community repo immediately after
install so it cannot conflict with official FlightAware packages once they ship.

## Requirements

- Debian 12 or 13 (Raspberry Pi OS included)
- An RTL-SDR dongle for 1090 MHz ADS-B reception
- A [FlightAware feeder ID](https://flightaware.com/adsb/piaware/claim) if you are
  migrating an existing feeder (configure with `piaware-config feeder-id <uuid>`;
  managing the feeder ID is currently outside this role's scope)

## Role Variables

All variables and their defaults:

| Variable | Default | Description |
|---|---|---|
| `piaware_manage_packages` | `true` | Set `false` to skip apt installation entirely (packages pre-installed or managed elsewhere) |
| `piaware_packages` | `[piaware, dump1090-fa, piaware-web]` | Packages to install |
| `piaware_remove_apt_repo` | `true` | Remove the community apt repo after install (upstream recommendation). Set `false` to keep it for `apt upgrade` |
| `piaware_receiver_serial` | `""` | RTL-SDR serial to dedicate to 1090 MHz. Leave empty on single-dongle hosts; required when multiple SDRs are attached |
| `piaware_apt_key_url` | abcd567a GitHub URL | GPG key source |
| `piaware_apt_key_dest` | `/etc/apt/keyrings/abcd567a-key.gpg` | GPG key destination |
| `piaware_apt_list_url` | abcd567a GitHub URL | apt source list URL |
| `piaware_apt_list_dest` | `/etc/apt/sources.list.d/abcd567a.list` | apt source list destination |

### Multiple SDR dongles

If the host has more than one RTL-SDR (for example a second dongle for 978 MHz UAT or
433 MHz sensors), give each dongle a unique serial and tell dump1090-fa which one to use:

```bash
rtl_eeprom -s 1090
```

```yaml
piaware_receiver_serial: "1090"
```

## Example Playbook

```yaml
- hosts: adsb_receivers
  roles:
    - role: piaware
      vars:
        piaware_receiver_serial: "1090"
```

Installing via `requirements.yml`:

```yaml
roles:
  - name: piaware
    src: https://github.com/tjg-homelab/ansible-role-piaware.git
    version: v1.0.0
```

## Testing

Molecule (Docker driver) converges and verifies against Debian 12 and Debian 13
containers. Package installation is skipped in containers (no RTL-SDR hardware,
and services are stubbed); the scenario verifies configuration management and
service enablement.

```bash
pip install ansible-core molecule molecule-plugins[docker] docker
molecule test
```

## When official Trixie packages ship

Point `piaware_apt_key_url` / `piaware_apt_list_url` at the official FlightAware
repository (or set `piaware_manage_packages: false` and manage packages yourself).
Watch the [abcd567a README](https://github.com/abcd567a/debian13/blob/master/README.md)
for package updates in the meantime.

## License

MIT

## Author

Rodney Nissen ([The Jira Guy](https://thejiraguy.com))
