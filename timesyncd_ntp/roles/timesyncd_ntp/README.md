# timesyncd_ntp

Sets the IPv4 address of an NTP server in `/etc/systemd/timesyncd.conf` and (optionally) the system timezone, restarts the service, and reports whether the clock synchronized.

## Variables

| Variable | Default | Description |
|---|---|---|
| `timesyncd_ntp_server` | `""` | **Required.** IPv4 address of the NTP server. Set globally in `group_vars/all.yml`. |
| `timesyncd_ntp_timezone` | `""` | Timezone name, e.g. `Asia/Tehran` or `UTC`. Empty = leave unchanged. Validated against the tz database. Set globally in `group_vars/all.yml`. |
| `timesyncd_ntp_tzdata_dir` | `/usr/share/zoneinfo` | Timezone database directory used for validation |
| `timesyncd_ntp_config_file` | `/etc/systemd/timesyncd.conf` | Path to the config file |
| `timesyncd_ntp_sync_retries` | `12` | Number of sync status checks |
| `timesyncd_ntp_sync_delay` | `5` | Delay between checks (seconds) |
| `timesyncd_ntp_fail_if_unsynced` | `false` | If `true`, the play fails when the clock does not synchronize. Set globally in `group_vars/all.yml`. |

## Global configuration

The project sets the main variables in `group_vars/all.yml`:

```yaml
timesyncd_ntp_server: 192.168.1.10
timesyncd_ntp_timezone: Asia/Tehran
timesyncd_ntp_fail_if_unsynced: false
```

## Usage

```yaml
# playbook.yml
- hosts: all
  serial: "20%"
  max_fail_percentage: 10
  become: true
  roles:
    - role: timesyncd_ntp
```

Run:

```bash
ansible-playbook -i inventory.ini playbook.yml --check --diff
ansible-playbook -i inventory.ini playbook.yml
```

## Sample report

```
Clock synchronized successfully ✅ | host=web01 | configured=192.168.1.10 | active=192.168.1.10 | config_changed=true
```

## Notes
- The timezone is set with `timedatectl set-timezone` only when it differs from the current one (idempotent). An invalid or missing timezone fails the play before anything is changed.
- Sync status is checked with `timedatectl show -p NTPSynchronized --value`.
- In `--check` mode the config change is shown, but the service is not restarted and sync is not checked.
