# Create a Node.js Fastify.js project using fastify-cli

## Install Fastify CLI if not installed yet.

```bash
sudo npm install fastify-cli --global
```

## generate fastify project

```bash
fastify generate <app-name>
cd <app-name>
npm install
```

## generate fastify project on current directory

```bash
fastify generate . --integrate
npm install
```

## generate fastify project with additional flags

```bash
fastify generate <app-name> --lang=ts --standardlint --esm
cd <app-name>
npm install
```

## Production

```
npm start
# or
fastify eject
node server.js
```

## See full guide

https://github.com/fastify/fastify-cli
