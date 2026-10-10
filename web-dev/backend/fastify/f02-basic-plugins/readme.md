## Fastify FormBody
Enables parsing `application/x-www-form-urlencoded` request bodies, which is what a plain HTML `<form>` sends when it is submitted (the parsed data is available in `request.body`).

- **Install**
    ```
    npm install @fastify/formbody
    ```

- **Sample**
    ```js
    // register by conventional way
    const fastifyFormBody = require('@fastify/formbody');
    fastify.register(fastifyFormBody);

    // register directly
    fastify.register(require('@fastify/formbody'));
    ```

## Fastify Static

Enables serving static files, like html, css, and javascript to clients.

- **Install**
    ```
    npm install @fastify/static
    ```

- **Sample**
    ```js
    // import fastify static
    const fastifyStatic = require('@fastify/static');
    const path = require('path');

    // register the fastify static plugin.
    // this also sets a root path where
    // all the static files are located
    fastify.register(fastifyStatic, {
      root: path.join(__dirname, '/static-folder')
    });

    // serve the file to the clients
    // (the 2nd parameter is fastify's `reply` object, named `res` here)
    fastify.get('/', (req,res)=>{
      return res.sendFile('static-form.html'); // path is relative to `root`
    });
    ```
