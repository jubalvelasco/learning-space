## Pipes

Sometimes, you need to read data from a stream simply to turn around and write that same data to another stream.

-----

Imagine, for example, that you are writing a simple HTTP server that serves a directory of static files. In this case, you will need to read data from a file input stream and write it out to a network socket. But instead of writing your own code to handle the reading and writing, you can instead simply connect the two streams together as a “pipe” and let Node handle the complexities (including backpressure) for you. Simply pass the Writable stream to the pipe() method of the Readable stream:

```javascript
const fs = require('fs');

function pipeFileToSocket(filename, socket) {
    fs.createReadStream(filename).pipe(socket);
}

// pipes one stream to another and invokes a callback
// when done or when an error occurs
function pipe(readable, writable, callback) {
    
    // First, setup an error handling function
    // (.pipe() does NOT clean up the other stream when one of them fails, so we do it ourselves)
    function handleError(err) {
        readable.destroy();
        writable.destroy();
        callback(err);
    }

    // Next define the pipe and handle the normal termination
    readable
        .on('error', handleError)
        .pipe(writable)
        .on('error',handleError)
        .on('finish',callback);
}

// a function that compresses a file
const zlib = require('zlib');

function gzip(filename, callback) {
    // Create the streams
    let source = fs.createReadStream(filename);
    let destination = fs.createWriteStream(filename + '.gz');
    let gzipper = zlib.createGzip();

    // Setup the pipeline
    source
        .on('error',callback) // call callback on read error
        .pipe(gzipper)
        .pipe(destination)
        .on('error',callback) // call callback on write error
        .on('finish', callback); // call callback when writing is complete
}

// Modern alternative: stream.pipeline() forwards errors from every stream
// and destroys all of them on failure (there is also a promise version in 'stream/promises')
const { pipeline } = require('stream');

function gzip2(filename, callback) {
    pipeline(
        fs.createReadStream(filename),
        zlib.createGzip(),
        fs.createWriteStream(filename + '.gz'),
        callback // called with an error, or with no arguments on success
    );
}
```