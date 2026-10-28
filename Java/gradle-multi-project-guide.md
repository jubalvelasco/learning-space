# Gradle Multi-Project Java Setup

A guide to generating a Java build that is split into several subprojects (modules) with `gradle init`, then adding tests and dependencies to it.

> Verified with **Gradle 8.14.3**. Option names and generated files can change between Gradle versions, so run `gradle help --task init` to see what your installed version supports.

---

## 1. Generate the project

`gradle init` has a `--split-project` option that splits an application into one app subproject plus library subprojects.

```bash
mkdir demo && cd demo

gradle init \
  --type java-application \
  --split-project \
  --test-framework junit-jupiter \
  --dsl kotlin \
  --package com.example \
  --project-name demo \
  --java-version 17
```

| Option | Meaning |
| --- | --- |
| `--type java-application` | A runnable Java app. The split option only applies to the application type; `java-library` always produces a single project. |
| `--split-project` | Generate multiple subprojects (`--no-split-project` for one). |
| `--test-framework junit-jupiter` | JUnit 5. See [Testing](#4-testing) for the other choices. |
| `--dsl kotlin` | Build scripts in Kotlin (`.gradle.kts`). Use `groovy` for `.gradle` files. |
| `--package` | Base package for the generated source code. |
| `--project-name` | Root project name. Defaults to the folder name. |
| `--java-version` | Java toolchain version the build compiles with. |

### Interactive alternative

Run plain `gradle init` and answer the prompts:

1. *Select type of build to generate:* **Application**
2. *Select implementation language:* **Java**
3. *Enter target Java version:* e.g. **17**
4. *Project name:* press Enter to use the folder name
5. *Select application structure:* **Application and library project** (this is the split option)
6. *Select build script DSL:* **Kotlin** or **Groovy**
7. *Generate build using new APIs and behavior?* **no** (keeps to stable features)

There is no test framework question for the split structure, because JUnit Jupiter is the only option there.

---

## 2. What gets generated

```
demo/
├── settings.gradle.kts          # root project name + include("app", "list", "utilities")
├── gradle.properties
├── gradle/
│   ├── libs.versions.toml       # version catalog (starts empty)
│   └── wrapper/                 # Gradle wrapper files
├── gradlew, gradlew.bat         # use these instead of a system-wide `gradle`
├── buildSrc/                    # shared build logic (convention plugins)
│   ├── build.gradle.kts
│   ├── settings.gradle.kts      # makes the version catalog visible to buildSrc
│   └── src/main/kotlin/
│       ├── buildlogic.java-common-conventions.gradle.kts
│       ├── buildlogic.java-application-conventions.gradle.kts
│       └── buildlogic.java-library-conventions.gradle.kts
├── app/                         # runnable application
│   ├── build.gradle.kts
│   └── src/{main,test}/java/com/example/app/
├── list/                        # library
│   ├── build.gradle.kts
│   └── src/{main,test}/java/com/example/list/
└── utilities/                   # library
    ├── build.gradle.kts
    └── src/main/java/com/example/utilities/
```

### The three subprojects

They are demo modules arranged in a dependency chain. The code is placeholder; the structure is the useful part.

```
app  ──implementation──>  utilities  ──api──>  list
```

- **`list`** is the lowest layer: a library with a hand-written `LinkedList` class and no dependencies of its own. It stands in for core data structures or domain code.
- **`utilities`** is a library with string helpers (`StringUtils`, `SplitUtils`, `JoinUtils`) that split text into a `LinkedList` and join it back. It depends on `list` with **`api`** because `LinkedList` appears in its public method signatures, so anything using `utilities` also needs `list` to compile.
- **`app`** is the application. It applies the `application` plugin, has the `main` class (`App.java`), depends on `utilities`, and is the only module you run.

### Convention plugins (`buildSrc`)

Instead of repeating configuration in every module, shared settings live in three convention plugins:

| Plugin | Applied by | Contains |
| --- | --- | --- |
| `buildlogic.java-common-conventions` | the other two | `java` plugin, `mavenCentral()`, Java toolchain, JUnit setup, shared version constraints |
| `buildlogic.java-application-conventions` | `app` | common conventions + `application` plugin |
| `buildlogic.java-library-conventions` | `list`, `utilities` | common conventions + `java-library` plugin (enables `api`) |

Because of this, each module's own build file stays short. For example, `utilities/build.gradle.kts`:

```kotlin
plugins {
    id("buildlogic.java-library-conventions")
}

dependencies {
    api(project(":list"))
}
```

### Making it your own

Keep `app`, then replace `list` and `utilities` with real modules such as `core`, `persistence` or `api`. Follow the same pattern: library modules apply the library conventions, and the app applies the application conventions.

---

## 3. Adding a new module

1. Create the folder and source directories:

   ```bash
   mkdir -p core/src/main/java/com/example/core core/src/test/java/com/example/core
   ```

2. Add `core/build.gradle.kts`:

   ```kotlin
   plugins {
       id("buildlogic.java-library-conventions")
   }
   ```

3. Register it in `settings.gradle.kts`:

   ```kotlin
   include("app", "list", "utilities", "core")
   ```

4. Depend on it from another module, e.g. in `app/build.gradle.kts`:

   ```kotlin
   dependencies {
       implementation(project(":core"))
   }
   ```

---

## 4. Testing

### Choosing a framework at generation time

`--test-framework` accepts different values depending on the project layout:

- **Split (multi-project) application:** only `junit-jupiter` (JUnit 5). Passing `junit`, `testng` or `spock` fails with *"not supported for 'java-application' build type"*.
- **Single-project application** (`--no-split-project`): other frameworks such as `junit` (JUnit 4), `testng` and `spock` are accepted.

So for a multi-project build, generate with JUnit 5 and switch manually afterwards if you need something else.

### Where the test setup lives

It is in `buildSrc/src/main/kotlin/buildlogic.java-common-conventions.gradle.kts`, so every module inherits it:

```kotlin
dependencies {
    testImplementation("org.junit.jupiter:junit-jupiter:5.12.1")
    testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}

tasks.named<Test>("test") {
    useJUnitPlatform()
}
```

The generator includes sample tests for `app` and `list`. `utilities` has none, so add a `src/test/java/...` folder there when you need one.

### Switching to JUnit 4 or TestNG

Edit the common conventions file, then update imports and annotations in existing test classes.

**JUnit 4:**

```kotlin
dependencies {
    testImplementation("junit:junit:4.13.2")
}

tasks.named<Test>("test") {
    useJUnit()
}
```

**TestNG:**

```kotlin
dependencies {
    testImplementation("org.testng:testng:<version>")
}

tasks.named<Test>("test") {
    useTestNG()
}
```

### Adding test libraries (AssertJ, Mockito, ...)

- **For every module:** add them to the common conventions file next to JUnit.

  ```kotlin
  dependencies {
      testImplementation("org.assertj:assertj-core:<version>")
      testImplementation("org.mockito:mockito-core:<version>")
  }
  ```

- **For one module only:** add them to that module's `build.gradle.kts`.

### Integration tests

Use the built-in JVM Test Suite plugin in the conventions file (`testing { suites { ... } }`). It gives you a separate source set and task such as `integrationTest`.

### Running tests

```bash
./gradlew test                 # all modules
./gradlew :utilities:test      # one module
./gradlew :app:test --tests 'com.example.app.MessageUtilsTest'   # one class
```

---

## 5. Adding dependencies

### Where to put a dependency

| Who needs it | Where it goes |
| --- | --- |
| One module | That module's `build.gradle.kts` |
| Every module | `buildSrc/.../buildlogic.java-common-conventions.gradle.kts` |
| One of your own modules | `implementation(project(":name"))` in the consuming module |

Example, in `app/build.gradle.kts`:

```kotlin
dependencies {
    implementation("com.google.guava:guava:<version>")
}
```

### Choosing the configuration

| Configuration | Use when |
| --- | --- |
| `implementation` | The normal choice. Used internally, not exposed to modules that depend on this one. |
| `api` | Its types appear in this module's public methods, so dependents need it too. Library modules only. |
| `compileOnly` | Needed to compile but not at runtime (e.g. annotation-only libraries). |
| `runtimeOnly` | Needed only at runtime (e.g. a JDBC driver or logging backend). |
| `testImplementation` | Tests only. |
| `testRuntimeOnly` | Test runtime only (e.g. `junit-platform-launcher`). |

### Option A: version catalog (recommended)

Declare each version once in `gradle/libs.versions.toml`:

```toml
[versions]
guava = "<version>"
assertj = "<version>"

[libraries]
guava = { module = "com.google.guava:guava", version.ref = "guava" }
assertj-core = { module = "org.assertj:assertj-core", version.ref = "assertj" }
```

Then reference it by name in any module. Dashes in the name become dots:

```kotlin
dependencies {
    implementation(libs.guava)
    testImplementation(libs.assertj.core)
}
```

Every module uses the same version, and an upgrade is a one-line change.

**Using the catalog inside a convention plugin:** the generated `buildSrc/settings.gradle.kts` already imports the catalog, but the type-safe `libs.xxx` accessors don't work inside the precompiled `*.gradle.kts` plugins. Look entries up by name instead:

```kotlin
// buildlogic.java-common-conventions.gradle.kts
val libs = the<VersionCatalogsExtension>().named("libs")

dependencies {
    testImplementation(libs.findLibrary("assertj-core").get())
}
```

### Option B: constraints in the conventions file

The generated project also shows a second way to share versions. The common conventions file declares a version once as a **constraint**:

```kotlin
dependencies {
    constraints {
        implementation("org.apache.commons:commons-text:1.13.0")
    }
}
```

A module then adds the dependency **without** a version and gets the constrained one:

```kotlin
// app/build.gradle.kts
dependencies {
    implementation("org.apache.commons:commons-text")
}
```

A constraint only pins the version. Modules that don't declare the dependency don't get it.

After editing dependencies, reload the project in your IDE or run `./gradlew build`.

---

## 6. Command cheat sheet

```bash
./gradlew build                 # compile + test everything
./gradlew :app:run              # run the application
./gradlew test                  # run all tests
./gradlew :list:test            # test one module
./gradlew projects              # list subprojects
./gradlew :app:dependencies     # show app's dependency tree
./gradlew clean                 # delete build outputs
gradle help --task init         # all init options for your Gradle version
```

---

## References

- [Build Init Plugin](https://docs.gradle.org/current/userguide/build_init_plugin.html)
- [Sample: Building Java Applications with libraries](https://docs.gradle.org/current/samples/sample_building_java_applications_multi_project.html)
- [Version Catalogs](https://docs.gradle.org/current/userguide/version_catalogs.html)
- [JVM Test Suite Plugin](https://docs.gradle.org/current/userguide/jvm_test_suite_plugin.html)
