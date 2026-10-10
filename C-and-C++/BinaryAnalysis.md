# Show shared libraries needed by an executable in linux

```bash
readelf -d executable.out   # look for the (NEEDED) entries

# alternatively, list the resolved shared libraries (do not run on untrusted binaries, it may execute them)
ldd executable.out
```
