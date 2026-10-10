# SSH Keys

Check to see if you have a key already:

```bash
cd ~/.ssh
ls

# Example Output:
authorized_keys2  id_dsa       known_hosts
config            id_dsa.pub
```

You’re looking for a pair of files named something like `id_ed25519`, `id_rsa` or `id_dsa` and
a matching file with a `.pub` extension. The `.pub` file is your public key,
and the other file is the corresponding private key.

NOTE: DSA (`id_dsa`) keys are obsolete and disabled by default in modern OpenSSH; prefer Ed25519 keys (`ssh-keygen -t ed25519`), or RSA with at least 3072 bits.

# [Generate SSH key (click here)](https://git-scm.com/book/en/v2/Git-on-the-Server-Generating-Your-SSH-Public-Key)

If you don’t have these files (or you don’t even have a .ssh directory), you can create them by running a program called `ssh-keygen`, which is _provided with the SSH package on Linux/macOS_ systems and comes with _Git for Windows_:


SSH files should appear in your `~/.ssh` for ubuntu systems.

# Authenticating Users

This section follows the git tutorial on [how to set up a Git server](https://git-scm.com/book/en/v2/Git-on-the-Server-Setting-Up-the-Server).

we will use the `authorized_keys` method for authenticating our users.

You need to add some SSH public keys to the `~/.ssh/authorized_keys` in the ssh server. 

Let’s assume you have some trusted public keys and have
saved them to temporary files. Again, the public keys look something like this:

```bash
cat /tmp/id_rsa.john.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCB007n/ww+ouN4gSLKssMxXnBOvf9LGt4L
ojG6rs6hPB09j9R/T17/x4lhJA0F3FR1rP6kYBRsWj2aThGw6HXLm9/5zytK6Ztg3RPKK+4k
Yjh6541NYsnEAZuXz0jTTyAUfrtU3Z5E003C4oxOj6H0rfIF1kKI9MAQLMdpGW1GYEIgS9Ez
Sdfd8AcCIicTDWbqLAcU4UpkaX8KyGlLwsNuuGztobF8m72ALC/nLF6JLtPofwFBlgc+myiv
O7TCUSBdLQlgMVOFq1I2uPWQOkOWQAHukEOmfjy2jctxSDBQ220ymjaNsHT4kgtZg2AYYgPq
dAv8JggJICUvax2T9va5 gsg-keypair
```

You just append them to the `authorized_keys` file in its `.ssh` directory:

```bash
cat /tmp/id_rsa.john.pub >> ~/.ssh/authorized_keys
cat /tmp/id_rsa.josie.pub >> ~/.ssh/authorized_keys
cat /tmp/id_rsa.jessica.pub >> ~/.ssh/authorized_keys

# you can also manually copy it if you want
```

YOU CAN NOW LOGIN VIA SSH KEY IN YOUR LOCAL MACHINE

# SSH login

1. If SSH key authentication is set up, SSH will automatically try to use your private key (`~/.ssh/id_rsa` or `~/.ssh/id_ed25519`).

    ```bash
    ssh <user_name>@<ip/domain>
    ```

2. On the other hand if your key is not named as `id_rsa` or `id_ed25519` then
you will need to manually supply it yourself to the command. Note that `-i` takes the **private** key file (the one without `.pub`).

    ```
    ssh -i <private-key-filename> <user_name>@<ip/domain>
    ```

3. Specifying a port number - sometimes an ssh server listens on a different port (e.g. 8022) instead of the usual port 22, to reduce noise from automated attacks that target port 22 (this is obscurity, not real security, so still use keys and disable password login).

    ```bash
    ssh -i <private-key-filename> <user_name>@<ip/domain> -p 8022
    ```

# Permission Classes

We can then further add a little bit more security to our `.ssh` directory
by:

```bash
cd ~
chmod 700 .ssh
chmod 600 .ssh/authorized_keys # only owner can read and write `authorized_keys` file
chmod 600 .ssh/id_rsa # only owner can read and write private keys 
```
If you want to learn more about permissions continue below.

In a machine like ubuntu we have three permission classes (plus a shorthand for all of them):

1. Owner (User) - `u`
2. Group - `g`
3. Others - `o`
4. All - `a` = `ugo` (shorthand, not a separate class)

# Numeric (Octal) File Permissions

File permissions in Linux/Unix are represented by three digits,
which are octal numbers (base 8), each digit represents
permissions for the; Owner (User), Group, Others.

If we do `ls -l filename` and we assume that the example output is

```bash
-rw-r--r--  1 user user 1234 Feb 28 12:00 myfile.txt
```

Then that would mean the following permission for that file:

- `rw-` (Owner can **r**ead and **w**rite but not e**x**ecute)
- `r--` (Group can only **r**ead but they can't **w**rite or e**x**ecute)
- `r--` (Others can only **r**ead and they can't **w**rite or e**x**ecute)

The corresponding numeric value for this is `644`, if we convert each digit
of this number into binary we would get; `0b110` for `6` and `0b100` for both `4`,
each bit in the binary represents the permission [r][w][x].

Also in `ls -l`, the first character represents the file type:
- `d` - Directory
- `-` - Regular file
- `l` - Symbolic link
- `c` - Character device (e.g., terminal, /dev/tty)
- `b` - Block device (e.g., disk partitions, /dev/sda)
- `p` - Named pipe
- `s` - Socket

# Letters File Permission

If you don't want to use numbers to set permission you can use letters instead.
for example:

```bash
chmod u=rwx,go= my-script.sh
```

Let's break down the command above, the letters `u`, `g`, and `o` are the
`Owner/User`, `Group` and `Others`, while the `r`, `w`, and `x` are the read,
write, and execute permissions. We use the `=` to set certain permissions (an empty value like `go=` removes all permissions), and
we use the `,` if we want to set permissions for different classes differently.

### Adding Permission to Everyone Using Letters

The `chmod +x` will add execute permission for `Owner/User`, `Group` and
`Others` to the file or directory, it's almost the same as `chmod a+x file`
(the difference: without a class letter, bits that are masked by your `umask` are not added).

### Adding Permission Selectively Using Letters

- `chmod u+x file` - Add execute for owner (user)
- `chmod g+x file` - Add execute for group
- `chmod o+x file` - Add execute for others

# Key Permission `0664`

### 1. Go to your `.ssh` folder.

```bash
cd /home/$USER/.ssh
```

### 2. Check the permission of the private key.

```bash
ls -l id_rsa
```

Output:

```bash
-rw-rw-r-- 1 your_username your_username 1234 Oct 30 12:34 id_rsa
```

The permissions `rw-rw-r--` in the above example are too open (664). To fix this, you should make the file readable and writable only by the owner and not accessible by others.

### 3. Change the permission to the private key

```bash
chmod 600 id_rsa
```

This command sets the file permissions to 600, which means it is readable and writable only by the owner.

### 4. Verify that the permissions have been updated correctly:

```bash
ls -l id_rsa
```

Output:

```bash
-rw------- 1 your_username your_username 1234 Oct 30 12:34 id_rsa
```

The permissions are now correct, and your private key is no longer accessible by others.