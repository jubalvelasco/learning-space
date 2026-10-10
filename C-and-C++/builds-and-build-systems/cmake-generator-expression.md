# **CMAKE GENERATOR EXPRESSION**

### Get Target Build Directory

Usually targets are the first argument of `add_library` and `add_executable`
(including imported targets, e.g. the ones created by `find_package` or by a project
pulled in with `FetchContent_MakeAvailable`). It must be a target that produces a file
(not an `INTERFACE` library or a custom target, e.g. the one created by `ExternalProject_Add`).

```cmake
$<TARGET_FILE_DIR:TARGET_NAME>
```

### Equality

Returns `1` if the (integer) arguments are equal and `0` if not.

```cmake
# Format (do not put spaces after the commas, spaces break generator expressions)
$<EQUAL:ARG1,ARG2>

# Example - check if pointer size is equal to 8 to determine architecture 8 = 64bit, 4 = 32bit
$<EQUAL:${CMAKE_SIZEOF_VOID_P},8>
```

### If Conditions

Returns `VALUE_IF_TRUE` or `VALUE_IF_FALSE` depending on the condition argument (which must evaluate to `1` or `0`).

```cmake
# Format
$<IF:CONDITION,VALUE_IF_TRUE,VALUE_IF_FALSE>

# Example
$<IF:$<EQUAL:${CMAKE_SIZEOF_VOID_P},8>,x64,x86>
```
