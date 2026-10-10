# Downloading Files Using Terminal

### Using Wget

filename will be saved as it is

```bash
wget DIRECT_LINK_OF_FILE
```

set a new filename

```bash
wget -O SAVE_AS_FILENAME DIRECT_LINK_OF_FILE

# add -c to resume a partially downloaded file
wget -c -O SAVE_AS_FILENAME DIRECT_LINK_OF_FILE
```

### Using Curl

```bash
# -L follows redirects, -o sets the output file name
curl <https-file-download-url> -o <new-file-name> -L
```
