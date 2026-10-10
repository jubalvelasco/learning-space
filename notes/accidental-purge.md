# Accidental Purge

I recently accidentally purged a specific package in ubuntu that also removed other packages that depend on it,
I quickly stopped the process when I found that it was removing OS specific packages with `ctrl+z` (`ctrl+c` was not working), then I used the following
commands below to try to bring back the purged packages.

NOTE: `ctrl+z` does not cancel a process, it only **suspends** it (it keeps running in the background in a stopped state, still holding the dpkg/apt lock), that's why it needs to be killed afterwards.

```bash
# find the suspended apt/dpkg process id
ps aux | grep -E 'apt|dpkg'

# kill if there's a process being held
sudo kill -9 <pid>   # SIGKILL - force kill
sudo kill -1 <pid>   # SIGHUP  - hang up signal

# repair the interrupted dpkg state before installing anything
sudo dpkg --configure -a
```

```bash
# look at the last packages that were removed/purged
grep -E " (remove|purge) " /var/log/dpkg.log | tail -150

# apt also keeps a readable history of each command and the packages it removed
less /var/log/apt/history.log

# find the packages that were purged then manually install them back and pray

# example: reinstall desktop
sudo apt install lubuntu-desktop
```
