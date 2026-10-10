# Rustlang Notes

### **Installing rustup on Linux or MacOS**

I suggest getting the command from the official rust website since the command given below might be subject to changes in the future.

```
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

### **Update rust**
```
$ rustup update
```

### **Uninstall rust**
```
$ rustup self uninstall
```

### **Documentation**

Access the rust documentation.

```
rustup docs
rustup docs --book
```

### **Compiling Single File Source Code**
```
rustc main.rs
./main
```

### **Creating a Project with Cargo**
```
cargo new project_name
cd project_name
```

### **Build and Run - method 1**

Compiles the code.

Creates the `Cargo.lock` file, similar to how `npm install` creates a `package-lock.json` file.

by default produces a **debug build**, to enable
optimizations add the **`--release`** flag.

```
cargo build
./target/debug/project_name # run manually
```

### **Build and Run - method 2**

Compiles the code then run it automatically.

```
cargo run
```

### **Cargo File Tracking**

The `cargo build` and `cargo run` commands work
like **makefiles**, similar to C and C++ where
if we run the `make` command, only the changed sources
are recompiled (cargo recompiles the crates that changed, dependencies are not rebuilt).

### **Cargo Compile Time Check**

This command will check if your rust code can be
compiled, but unlike `cargo build` and `cargo run`,
`cargo check` **does not** create an executable, because
of this, it is much faster than `cargo run` and
`cargo build`.

```
cargo check
```

### **Add a Package**

- **using cargo**

    ```
    cargo add <package-name>
    ```

- **manually**

    add the name of the package to the Cargo.toml

    ```toml
    <package-name> = "major.minor.patch"
    ```

    the major.minor.patch is the version number of the package that you want to install.


### **Cargo Update Dependencies**

```shell
cargo update
```

### **Cargo Build Dependency Documentation**

Another neat feature of Cargo is that running the cargo doc --open command will build documentation provided by all your dependencies locally and open it in your browser.

```shell
cargo doc --open
```

### **Cargo format your rust code**

Fix the format of your rust code using the following command, this will format all the rust source files of the crates in your package (not just `main.rs`).

```bash
cargo fmt -v
```