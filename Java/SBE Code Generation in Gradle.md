# Gradle Code Generation: A General Guide

Many Java projects generate source code from a schema or specification during the build. Examples include SBE, Protocol Buffers, Avro, OpenAPI client generators, and parser generators. The tools differ, but the Gradle workflow is usually similar:

1. Make the generator available to a dedicated Gradle task.
2. Give the task its input schema and configuration.
3. Write generated files under `build/`.
4. Add the generated source directory to the appropriate source set.
5. Make compilation depend on generation.

Prefer an official Gradle plugin when the generator provides one; it may already configure inputs, outputs, and task dependencies. The manual pattern below is useful when you need to invoke a command-line generator yourself.

## General pattern for a Java-based generator

This Kotlin DSL template shows the main pieces. Replace the example dependency, main class, input file, system properties, and arguments with the values required by your generator.

```kotlin
val codegenTool by configurations.creating
val schemaFile = layout.projectDirectory.file("src/main/schema/input.xml")
val generatedSourcesDir = layout.buildDirectory.dir("generated/sources/my-generator/main")

dependencies {
    add(codegenTool.name, "com.example:generator-cli:1.0.0")
}

sourceSets {
    main {
        java.srcDir(generatedSourcesDir)
    }
}

tasks.register<JavaExec>("generateSources") {
    classpath = codegenTool
    mainClass.set("com.example.generator.Main")

    inputs.file(schemaFile)
    inputs.property("targetLanguage", "Java")
    outputs.dir(generatedSourcesDir)

    systemProperty("generator.output.dir", generatedSourcesDir.get().asFile.absolutePath)
    systemProperty("generator.target.language", "Java")
    args(schemaFile.asFile.absolutePath)
}

tasks.named("compileJava") {
    dependsOn("generateSources")
}
```

The example values beginning with `com.example` and `generator.` are placeholders—not real generator coordinates or options. Replace them with the dependency coordinates, main class, and command-line options documented by the specific tool.

## How the pieces fit together

```text
schema / specification
          │
          ▼
generateSources task ── runs the generator
          │
          ▼
build/generated/sources/...
          │
          ▼
compileJava compiles generated + hand-written Java
```

### 1. Put the generator on its own configuration

```kotlin
val codegenTool by configurations.creating

dependencies {
    add(codegenTool.name, "com.example:generator-cli:1.0.0")
}
```

A Gradle configuration is a named collection of dependencies. This custom configuration keeps the generator and its libraries separate from the application's dependencies:

* Use `codegenTool` as the generator task's classpath.
* Use `implementation` for libraries your application imports and needs at runtime.
* Avoid adding a build-time generator to `implementation` unless your application itself genuinely uses it.

For a non-JVM generator, a custom Java classpath may not apply. Use the tool's Gradle plugin or an `Exec` task to invoke the executable, and make sure the tool and its version are available reproducibly in your build environment.

### 2. Choose an output directory under `build/`

```kotlin
val generatedSourcesDir = layout.buildDirectory.dir("generated/sources/my-generator/main")
```

`layout.buildDirectory` provides a path inside the module's `build/` directory. Keeping generated files there separates them from hand-written files and allows them to be removed and recreated by Gradle. Use a distinct directory for each generator or source set to avoid output collisions.

### 3. Register generated files as source

```kotlin
sourceSets {
    main {
        java.srcDir(generatedSourcesDir)
    }
}
```

This adds the generated Java directory to the `main` source set, so Java compilation includes its files along with `src/main/java`.

The exact registration depends on the generated language and build plugins. For example, generated Kotlin should be registered with the Kotlin plugin's source-set mechanism; generated resources should be added to the resources source set. A Java source directory is not appropriate for every generator.

### 4. Define the generator task

For a generator that runs as a Java main class, use `JavaExec`:

```kotlin
tasks.register<JavaExec>("generateSources") {
    classpath = codegenTool
    mainClass.set("com.example.generator.Main")

    // Supply the generator's input, options, and output location.
}
```

`classpath` provides the generator and its runtime dependencies to the task process. `mainClass` selects the generator's entry point. The generator's actual options may be command-line arguments, JVM system properties, environment variables, or a combination; follow its documentation.

If the generator has a Gradle plugin, use its extension and task where practical instead of recreating the invocation manually. A plugin can manage tool versions, task wiring, and incremental behavior for you.

### 5. Declare task inputs and outputs

```kotlin
inputs.file(schemaFile)
inputs.property("targetLanguage", "Java")
outputs.dir(generatedSourcesDir)
```

Declaring inputs and outputs tells Gradle what the task depends on and where it writes results. When those values have not changed and the output still exists, Gradle can often mark the task up-to-date and skip running it. It also gives Gradle the information needed for reliable build caching where supported.

Declare every meaningful input that can change the output: schema files, configuration files, target language, generator options, and relevant version or mode settings. Declare the generated output directory (or files) as outputs. A generator should also handle obsolete generated files appropriately when schemas change or are removed.

### 6. Pass inputs and options to the generator

A typical invocation might pass an output directory as a system property and a schema as an argument:

```kotlin
systemProperty("generator.output.dir", generatedSourcesDir.get().asFile.absolutePath)
args(schemaFile.asFile.absolutePath)
```

These property names and argument formats are tool-specific. Some generators accept multiple input files, a configuration file, package names, or language options. Put schema and configuration files in the task's declared inputs as well as passing them to the process.

### 7. Make compilation wait for generation

```kotlin
tasks.named("compileJava") {
    dependsOn("generateSources")
}
```

`dependsOn` establishes task order: Gradle completes `generateSources` before `compileJava` begins. This matters because Gradle must have the generated files before compiling them.

This task name is for Java projects using the Java plugin (the `application` plugin also applies Java support). Other languages or source sets may need to depend on a different compilation task. When using a generator plugin, task wiring may already be handled for you.

## SBE example

In an Aeron project using SBE, the same pattern is configured with SBE's dependency coordinates, main class, schema, and system properties:

```kotlin
val sbeTool by configurations.creating
val generatedSbeDir = layout.buildDirectory.dir("generated/sbe/main")
val sbeSchema = layout.projectDirectory.file("src/main/resources/sbe/messages.xml")

dependencies {
    add(sbeTool.name, "uk.co.real-logic:sbe-tool:1.40.2")
}

sourceSets {
    main {
        java.srcDir(generatedSbeDir)
    }
}

tasks.register<JavaExec>("generateSbe") {
    classpath = sbeTool
    mainClass.set("uk.co.real_logic.sbe.SbeTool")

    inputs.file(sbeSchema)
    inputs.property("sbe.target.language", "Java")
    inputs.property("sbe.validation.stop.on.error", true)
    outputs.dir(generatedSbeDir)

    systemProperty("sbe.output.dir", generatedSbeDir.get().asFile.absolutePath)
    systemProperty("sbe.target.language", "Java")
    systemProperty("sbe.validation.stop.on.error", "true")
    args(sbeSchema.asFile.absolutePath)
}

tasks.named("compileJava") {
    dependsOn("generateSbe")
}
```

Here, `sbeTool` is the build-time tool classpath; `messages.xml` is the schema input; `generatedSbeDir` is the generated Java source directory; and `generateSbe` runs SBE before Java compilation. The SBE-specific property names and main class are not general Gradle conventions—other generators will use their own.

The same structure applies to other generators, but their invocation details vary. Check the generator's documentation for the Gradle plugin or CLI, required inputs, output paths, options, and supported languages. For generators that run in another ecosystem (for example, a Node-based CLI), use an appropriate plugin or `Exec` task rather than `JavaExec`.

## Running the tasks

Run only the generator:

```bash
./gradlew generateSources
```

Compile Java sources; Gradle runs the generator first because of the task dependency:

```bash
./gradlew compileJava
```

In a multi-project build, qualify the task with the subproject path, for example:

```bash
./gradlew :task1:generateSbe
./gradlew :task1:compileJava
```

Replace `task1` with the actual subproject name. A full application build or run will also reach generation through the compile-task dependency, provided the generator is wired to the compilation task used by that application.