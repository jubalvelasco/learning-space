# IT'S A LIE
we cannot truly make a custom asynchronous function in javascript (me: or maybe in one javascript runtime?).

```callbacks```, ```promises``` and ```async``` & ```await``` are all just ways to deal with built in asynchronous functions and methods, they provide ways to **run natively asynchronous functions** either **sequentially** or **simultaneously but waiting for each other**.

Those methods and functions are builtin and they run natively on our machine, probably written using another language, (me: or maybe using multiple javascript runtimes communicating with each other?), we can expect them as a separate thread but javascript provides us interface to work with those.

[an SO thread](https://stackoverflow.com/questions/9516900/how-can-i-create-an-asynchronous-function-in-javascript)
If you want to simulate an asynchronous function, you will eventually need to leverage on technology provided natively, such as

- ```setInterval```
- ```setTimeout```
- ```requestAnimationFrame```
- ```XMLHttpRequest```
- ```WebSocket```
- ```Worker```
- Some HTML5 APIs such as the File API, Web Database API
- technologies that support ```onload```
- ... and many others

**Note (clarification):** JavaScript code itself runs on a single thread (per realm/worker) driven by the **event loop**. `callbacks`, `promises` and `async`/`await` don't make your own code run in parallel, they only control **when** code runs: `async` functions and promise callbacks still run on the same thread, just later (as microtasks), so a heavy loop inside an `async` function still blocks everything. The real concurrent work (timers, network and file I/O, etc.) is done by the runtime/host (the browser, or Node.js with libuv and the OS), not necessarily by separate threads (e.g. network I/O mostly uses the OS's non-blocking I/O, while Node uses a thread pool for things like file system operations and crypto). If you need real parallel execution of your own JavaScript, use a `Worker` (Web Workers / Node.js `worker_threads`).

