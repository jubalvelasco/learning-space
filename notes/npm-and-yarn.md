# **Commands for npm and yarn**

### Update `node` runtime itself using `npm`

```bash
sudo npm cache clean -f

# (you can run this once)
sudo npm install -g n

sudo n stable
```

### **Enable corepack (for yarn/pnpm | one time, first use)**

(Corepack is bundled with Node.js 16.9+ up to Node.js 24; newer Node.js versions no longer bundle it, so install it first with `npm install -g corepack`)

```shell
corepack enable
```

### **Initialize npm/yarn projects**
```cmd
npm init
yarn init
```

### **For yarn 2**
```shell
yarn init -2
yarn set version stable
```

### Clean Install (mostly used for CI)
```
npm ci
```

### Install with development dependencies
```
npm install
yarn install
```

### Install production dependencies only
```
npm install --omit=dev
# older (deprecated) form:
npm install --production
```

### Show outdated local packages (npm, yarn 1)
```
npm outdated
yarn outdated
```

### Update local packages [npm version 5.0.0^] (Minor and Patch only)
```
npm update
```

### Update local packages (Major)

1. pre-requisite tool - (install once)
    ```
    npm install -g npm-check-updates
    ```

2. upgrade all version hints in ```package.json```
    ```
    ncu -u
    ```

3. install major updates
    ```
    npm install
    ```
    
### Semver Note
**Version number** format `MAJOR.MINOR.PATCH`:

- `MAJOR` version when you make incompatible API changes (`PATCH` and `MINOR` versions MUST be reset to 0 when `MAJOR` version is incremented.).
- `MINOR` version when you add functionality in a backwards compatible manner (`PATCH` version MUST be reset to 0 when `MINOR` version is incremented.).
- `PATCH` version when you make backwards compatible bug fixes.

### Update globally installed packages (minor and patch)
```
sudo npm update -g
```
omit `sudo` if node and npm are not installed in a write protected directory.
    
### Update globally installed packages (major)
```
npm outdated -g
sudo npm update -g
sudo npm install -g <new version showed in outdated | ex: npm@8.15.0>
```
    
### Run locally installed npm CLI tool/program
```
npx <tool/program>
```

### Run globally installed npm CLI tool/program
```
<tool/program>
```

### Displaying npm config flag values
```
npm config ls -l
```

### Setting-up npm config flag values (will override defaults)

Example:

```
npm config set fetch-retries 5
npm config set fetch-retry-mintimeout 600000
npm config set fetch-retry-maxtimeout 1200000
npm config set fetch-timeout 1800000
```

This example is helpful for slow internet connections.

### **Upgrading npm and yarn to latest stable version**

Upgrade npm itself
```
npm install -g npm@latest
```

Download the latest stable yarn release (yarn 1 command)
```cmd
yarn policies set-version
```

The command above might suggest running the command below after
it has finished, and if it did, run this:
```
yarn set version stable
```
### **Upgrading global yarn 2 version**

```shell
corepack prepare yarn@stable --activate
```



