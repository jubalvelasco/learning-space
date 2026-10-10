# Debugging using GDB

---

## Compile with debug symbols

To debug a C/C++ program with gdb, compile with -g and, for best debugging reliability, disable optimizations with -O0.

```bash
gcc -g -O0 main.c -o main   # for C
g++ -g -O0 main.cpp -o main # for C++
```

---

## Run the program with GDB

Open the program with gdb.

 ```bash
 gdb <program>.exe
 ```

Then run it inside gdb

```bash
run
```

Run with redirected stdin / stdout.
First save the inputs to a file, for example `input.txt`,
then run the program. You could also save the output in
output.txt.

```bash
run < input.txt
run > output.txt
run < input.txt > output.txt
```

Run with arguments:

```bash
run arg1 arg2 ...  argN
```

Alternatively set the arguments before running:

```bash
set args arg1 arg2
run
```

You can also start gdb with the program arguments already set:

```bash
gdb --args ./program arg1 arg2
# then inside gdb:
run
```

---

## Display Modes (TUI)

GDB includes a TUI (text user interface) with several layouts:

```bash
# display assembly only
layout asm

# display source + assembly (split view)
layout split

# display source only
layout src

# display registers window
layout regs

# cycle to the next layout
layout next

# show/list source (not a layout command)
list # or shorthand: l
```

---

## Break Points

Breakpoints are where the running program will automatically
pause, usually you should display the source code to see the line
number where you want to pause the program.

Here's how you can **add breakpoints** to your program.

```bash
b <line-number>   # break at line in current source file
b <file>:<line>   # e.g., b worker.c:120
b <function-name> # break at the start of a function
b *0x401234       # break at address
```

To **show breakpoint** and **watchpoint** numbers use the command
below, you could also use `info watchpoints` for displaying watchpoints
only.

```bash
info break
```

**Delete breakpoints**

```bash
delete <n> # delete breakpoint number n
delete     # delete all breakpoints (removes them)

# alternative: clear <file>:<line> # remove breakpoints at that line
```

**Enable/Disable breakpoints**

Disable is useful if you just want to temporarily disable a
breakpoint, not completely delete it so you can use it later.

```bash
disable <n> # disable breakpoint number n
enable <n>  # enable breakpoint number n
disable all # disable all breakpoints/watchpoints
enable all  # enable all breakpoints/watchpoints

# you can disable a list or range: disable 2 3 5 or disable 2-5
```

---

## Conditions

#### set a conditional breakpoint

```bash
break <func> if <expr>
break <file>:<line> if <expr>
```

#### apply a condition to an existing breakpoint

```bash
condition <breakpoint-number> <expr>
```

#### create a watchpoint with a condition

```bash
watch <expression>  # break when expression is written (value changes)
rwatch <expression> # break when expression is read
awatch <expression> # break when expression is read or written

# watchpoint that only triggers when the condition is true
watch <expression> if <condition>   # e.g., watch total if total > 100
```

---

## Controlling Program Execution

#### Pause the program

just press `ctrl+c`.

useful when debugging an infinite loop, you could find out why it is happening.

#### `step`

This command executes the next statement. If the next statement is a function call, `step` 
will enter (step into) the function and execute its first line.

```bash
s <optional-number-of-steps>
```

#### `next`

This command also executes the next statement. However, if the next statement is a function call, 
`next` will execute the entire function and then stop at the line immediately following the function
call in the current frame (it **"steps over"** the function).

```bash
n <optional-number-of-next-lines>
```

For assembly level next (executes one machine instruction, stepping over calls), use the command below.
Use `stepi` (`si`) if you want to step into calls at the instruction level.

```bash
nexti <optional-number-of-instructions>
```

#### `continue`

This command resumes program execution until a breakpoint is hit, the program terminates, or a 
signal is received that GDB is configured to stop for.

```bash
c
```

#### `finish`

Used to continue the execution of the current function until it returns to its caller. When the 
function returns, GDB will stop execution and display the return value of the function, if applicable.

This command is particularly useful when you are inside a function and want to quickly step out of it
without having to repeatedly use next or step through every line of code within that function.

```
finish
```

---

## Displaying Values and Information

#### Print a Variable using `print` or `p`.

It also works with function calls in your program, e.g. `print foo(3)` or `print obj.method()`
(as long as the function was compiled into the binary and not completely inlined away).

```bash
print <variable-name-of-code or expression>
```

#### `print`/`display` Formats

```bash
p /x var      # print var in hexadecimal
p /t var      # print var in binary
p /d var      # signed decimal
p /u var      # unsigned decimal
p /o var      # octal
p /c var      # as a character
```

Note: 0x0 is numeric zero; for pointers that means `NULL`.

See `help x` for more info on the FMT (format) switches.

#### Print Array Elements

```bash
# print 10 elements starting at pointer p (assumes p points to int)
p *p@10

# if arr is an array object, `p arr` prints the whole array,
# and `p arr[0]@10` (or `p *arr@10`) prints the first 10 elements
p arr[0]@10
```

you could also print std::vector to see its value `print vec_name`, and even call the methods of std::vector and see the result `print vec_name.size()`.

Printing STL containers nicely works because of GDB's Python "pretty printers" (shipped with libstdc++). Your own classes/structs can also be printed (all members are shown), and their methods can be called too, but only if the method exists in the binary (template methods or inline methods that were never used/instantiated, or were inlined away, cannot be called).

#### Low-Level memory examine

```bash
x/10dw p   # examine 10 words as signed decimal starting at p
x/20bx p   # examine 20 bytes in hex
x/10x p    # 10 words in hex
```

#### Print Array Values
 
```bash
print *ptr@number-of-elements
```

#### Watch / Delete / Display

Use with `info break`.

```bash
watch <expr>    # stop when expr is written
delete <n>      # delete breakpoint/watchpoint number n
# you can use shorthand: d <n> in many gdb setups

# auto-display expression each time program stops
display <expr>       # e.g., display i
display /x <expr>    # show in hex

# stop auto-display
undisplay <n>    # where n is the display number shown by 'info display'
```

#### Print all function calls (call stack)

`bt` alone prints a summary of the stack frames. Each line typically includes the frame number, the function name,
and sometimes the line number and source file where the call originated. It provides a concise overview of the call stack.

`bt full` or `backtrace full` provides a more detailed backtrace. In addition to the information provided by the basic `bt`
command, it also prints the values of local variables and function arguments within each stack frame. This can be extremely 
helpful for understanding the state of the program at each step of the call chain.

```bash
backtrace full

# or

bt full
```

#### print all source files known to GDB for the currently loaded program

```bash
info sources    # list all source files known to the program
info source     # info about the current source file
```

---

## Manipulating Values

#### Change variable value

```bash
set var <variable-name> = <value>

# or
set variable x = 42 # recommended form

# or shorthand:
set var x = 42

# older form sometimes used (avoid it, it can clash with gdb's own `set` sub-commands, e.g. a variable named `width`):
set x = 42
```

---

## Working with Threads

#### Display threads

```bash
info threads
```

#### Display information only from specific threads

For example to show info on threads 1 and 3 only we can use the command below.

```bash
info threads 1 3
```

#### Jump to a thread

```bash
thread <gdb-thread-id-number>
```

#### Scheduler Locking

- `set scheduler-locking off` (default), all threads are allowed to run when the program is resumed,
if another thread hits a breakpoint, GDB will stop all threads and switch to the thread that hit the breakpoint.


- When `set scheduler-locking on` is enabled, GDB ensures that only the currently selected thread 
is allowed to execute when the program is resumed (e.g., using continue, next, step). 
All other threads in the program are prevented from running.

- `set scheduler-locking step`, This mode optimizes for single-stepping. It locks other threads only
during single-stepping operations (like step or next) to prevent them from seizing the prompt by
preempting the current thread. Other threads will rarely get a chance to run when you step,
but they can run during continue.

#### Thread specific breakpoint

Sets a breakpoint on line 32 that is exclusive to thread 3

```bash
b 32 thread 3
```

This command sets a breakpoint on line 120 of the file worker.c,
but only for thread 2. 

```bash
b worker.c:120 thread 2
```

Set **conditional breakpoint** at line 5 but only when i == 34

```bash
break test.c:5 if i==34
```

#### Thread specific command examples

- Display the backtrace (or call stack) for all threads in a
multi-threaded program.

  ```bash
  thread apply all bt
  ```

- This command applies the `print <variable>` command to threads
with IDs 1 and 3.
 
  ```bash
  thread apply 1 3 print <variable>
  ```

- This command displays the register values for all threads.

  ```
  thread apply all info registers
  ```

- The `-c` flag (continue) ensures that if an error occurs while 
applying print my_variable to a thread, the thread apply command
continues to execute for other threads instead of aborting
(the error messages are still printed).
 
  ```bash
  thread apply all -c print <variable>
  ```

- The `-s` flag (silent) suppresses error messages or empty output
from the applied command, which can be useful when you expect some
commands to only apply to a subset of threads or to produce no
output in certain cases.

  ```bash
  thread apply all -s print my_variable
  ```

- By default `thread apply all` goes through the threads in descending
order of their thread IDs. The `-ascending` flag makes it go in ascending
order instead (there is no `-descending` flag since that is already the default).

  ```bash
  thread apply all -ascending bt
  ```

---

## Working with Processes

* `set follow-fork-mode child|parent` - choose which process gdb follows after `fork`/`vfork`.
* `set detach-on-fork on|off` - `off` keeps both parent and child attached as inferiors; `on` detaches the non-followed one.
* `set follow-exec-mode same|new` - control behavior after `exec`: keep same inferior or treat the exec as a new image/inferior.
* `catch fork` / `catch vfork` / `catch exec` - stop when those events occur.
* `info inferiors` - list processes gdb controls.
* `inferior <n>` - switch to inferior number n.
* `attach <pid>` / `detach` - attach to or release an external process.
* `info breakpoints` / pending (deferred) breakpoints - set breakpoints before the new image loads; gdb will resolve them at load time.

#### Fork - follow the child

```bash
set follow-fork-mode child
set detach-on-fork off   # keep both attached if you want to inspect parent later
catch fork
run
# on fork hit: use `info inferiors`, `inferior <n>` to switch, inspect, then `continue`
```

#### Fork - follow parent, inspect child later

gdb session:

```bash
set follow-fork-mode parent
set detach-on-fork off
catch fork
run
# after fork: `info inferiors` -> note child's inferior number -> `inferior 2` to inspect child
```

#### Exec - new image loaded by same process

- An `exec`-family call (for example `execve`, `execv`, `execl`, etc.) does not create a new OS process.

- Instead, `exec` replaces the current process's memory image, symbol tables, and program text with a different program.

- The OS PID stays the same, but the running program is now a different binary - i.e., a "new image" inside the same process.

```bash
catch exec
set follow-exec-mode new    # or `same` depending on whether you want a new inferior
run
# when catch exec fires:
# - if `new`: use `info inferiors`, `inferior N`, then set breakpoints for new image
# - if `same`: set breakpoints for the loaded image (e.g., `break main`) then `continue`
```

#### Fork + Exec - child execs a different program - follow child across exec

combined pattern:

```bash
set follow-fork-mode child
set detach-on-fork off
catch fork
catch exec
run
# workflow:
# - fork catch: inspect return value / inferiors
# - exec catch (usually on child): set breakpoints for the new image (or rely on deferred breakpoints)
# - use `info inferiors` and `inferior N` to confirm which inferior holds the exec'd image
```

* Tips: use `set detach-on-fork off` when you must inspect both parent and child in one session - otherwise the non-followed process may be detached. After `exec`, prefer `catch exec` + deferred breakpoints or set breakpoints once the new image loads.

---

## Others

### Refresh the Screen if the TUI display gets garbled

```bash
refresh   # or press ctrl+l
```

### Quit

```bash
quit
```
