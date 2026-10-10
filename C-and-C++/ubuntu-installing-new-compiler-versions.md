# Installing New GCC Version In Ubuntu

```bash

sudo apt update

# Install the Software Properties Common Package
sudo apt install software-properties-common

# Add the GCC Repository
sudo add-apt-repository ppa:ubuntu-toolchain-r/test

# Install the Desired GCC Version
sudo apt install gcc-13 g++-13

# Configure the Default GCC Version
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-13 100 --slave /usr/bin/g++ g++ /usr/bin/g++-13

# check new version
gcc --version
```

# Installing New Clang Version In Ubuntu

In some cases, a simple `update` and `upgrade` is sufficient.
However sometimes with Clang, running `update` and `upgrade` might not
install the latest version. This guide is for those situations
where additional steps are needed to ensure you have the latest Clang version.

### Remove old Clang installations (depending on the version)

```bash
# find all clang packages still installed
apt list --installed | grep clang

# find all llvm packages still installed
apt list --installed | grep llvm

# be careful when purging llvm packages and maybe clang,
# because your OS might be dependent on some llvm packages,
# if you see that the purge command will also remove a ton
# of other packages, chances are that llvm is a package
# dependency that you don't want to purge, so don't be quick
# in entering `Y` (yes), read carefully.
sudo apt-get purge <clang-or-llvm-packages>

# I suggest not using the wildcard character (`*`) when purging

# Example:
sudo apt-get purge llvm-3.2 clang-3.2

# check if some files are still there remove old clang and llvm
# files that are not a dependency if exist
sudo rm -r /usr/lib/clang*
sudo rm -r /usr/lib/llvm*
sudo rm -r /usr/bin/clang*
sudo rm -r /usr/bin/llvm*
sudo rm -r /usr/include/clang*
sudo rm -r /usr/include/llvm*

# Then lastly
sudo apt autoremove
```

### Verify that Clang is not installed

```bash
clang --version    # Should not work
clang++ --version  # Should also not work
```

### Download the Clang installation script

```bash
wget https://apt.llvm.org/llvm.sh
chmod u+x llvm.sh
```

### Run the Clang installation script

```bash
# Install the latest version
sudo ./llvm.sh

# Alternatively, specify a version to install:
sudo ./llvm.sh <new-selected-version>

# Example:
sudo ./llvm.sh 19
```

### Get the path of the installed Clang binary

```bash
which clang-<new-selected-version>

# Example:
which clang-19
```

Output:

```bash
/usr/bin/clang-19
```

### Locate the main binary of the installed Clang using the output from `which`

```bash
ls -l /usr/bin/clang-19
```

Output:

```bash
lrwxrwxrwx 1 root root XX XXX XX XXXX /usr/bin/clang-19 -> ../lib/llvm-19/bin/clang
```

This indicates the main binary path is `/usr/lib/llvm-19/bin/clang`.

### Confirm the binary is the actual Clang executable

```bash
ls -l /usr/lib/llvm-19/bin/clang
```

If the output does not start with `l`, as shown below, it confirms this is the Clang main binary:

```bash
-rwxr-xr-x 1 root root XX XXX XX XXXX /usr/lib/llvm-19/bin/clang
```

### Create a new symbolic link for Clang

```bash
sudo ln -s /usr/lib/llvm-19/bin/clang /usr/bin/clang 
sudo ln -s /usr/lib/llvm-19/bin/clang /usr/bin/clang++
sudo ln -s /usr/lib/llvm-19/bin/clangd /usr/bin/clangd
```


### Open a new terminal (or run `hash -r` to clear the shell's cached command paths), then verify the installation

```bash
clang --version
```

### Something is weird?

You might wonder why we link both `clang` (the C compiler) and `clang++` (the C++ compiler)
to the same Clang binary. The reason is that Clang determines its behavior based on the
program's name (`argv[0]`). If the executable is named `clang` (without `++`),
the binary behaves as a C compiler. On the other hand, if the executable is named `clang++`,
it operates as a C++ compiler. This naming convention allows a single binary to serve both purposes.
