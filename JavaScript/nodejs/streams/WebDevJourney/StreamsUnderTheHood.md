### Readable and Writable streams

1. Node's ```stream.Readable``` and ```stream.Writable``` classes (and other classes that extend them, like `Duplex` and `Transform`) are either directly or indirectly derived from the ```Stream``` class. (Don't confuse them with `ReadableStream`/`WritableStream`, which are the separate **Web Streams API** classes, also available in Node.)
2. The ```Stream``` class is derived from the ```EventEmitter``` class. Hence we are able to use the same event driven way... using the ```.on()``` methods.