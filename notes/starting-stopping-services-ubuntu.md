# Starting and Stopping Services in Ubuntu

### Check the status of all services (SysV init scripts) (`+` running, `-` not running, `?` unknown)

```bash
service --status-all

# show only the active ones
service --status-all | grep '\[ + \]'

# show only the inactive ones
service --status-all | grep '\[ - \]'
```

or use this command to get all system services (shows more information and services)

```bash
# by default this shows only the active (loaded) units
systemctl list-units

# show all units, both active and inactive
systemctl list-units -a

# show only the inactive ones
systemctl list-units -a --state=inactive

# show only services
systemctl list-units --type=service
```

### Check if a service is already running

```bash
sudo systemctl status <program>
# or
sudo service <program> status
```

### Run a service

```bash
sudo systemctl start <program>
# or
sudo service <program> start
```

### Stop a service

```bash
sudo systemctl stop <program>
# or
sudo service <program> stop
```

### Auto-start a service at startup

```
sudo systemctl enable <program>
```

### Disable auto-start a service at startup

```
sudo systemctl disable <program>
```
