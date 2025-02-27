---
title: Installation and Setup
---

## Installing Relay

Relay comes with quite a number of dependencies that don't involve Next.js.
We'll set those up first before moving on to `relay-nextjs`.

First install Relay's runtime dependencies:

```
npm install react-relay relay-runtime
```

TypeScript users should install appropriate `@types` packages:

```
npm install --save-dev @types/react-relay @types/relay-runtime
```

## Configuring Relay

Install `relay-config` to provide a single configuration file to the rest of
Relay:

```
npm install --save-dev relay-config
```

Create `relay.config.js`. For Next.js projects using TypeScript this should look
something like this:

```js
module.exports = {
  src: './src',
  schema: './src/schema/__generated__/schema.graphql',
  exclude: ['**/node_modules/**', '**/__mocks__/**', '**/__generated__/**'],
  extensions: ['ts', 'tsx'],
  language: 'typescript',
  artifactDirectory: 'src/queries/__generated__',
};
```

### Configuring `artifactDirectory`

Next.js's `/pages` directory cannot include non-React components as default
export.

By default, the relay-compiler generates `*.graphql.ts` files that are
co-located with the corresponding files containing graphql tags. To fix this
configure `artifactDirectory` to emit to `src/queries/__generated__`:

```js:relay.config.js
module.exports = {
  // ...
  artifactDirectory: 'src/queries/__generated__',
}
```

For more information please see the Relay
[type emission documentation](https://relay.dev/docs/guides/type-emission/#single-artifact-directory).
Alternatively you can keep `*.graphql.ts` files in `/pages` directory with
[`pageExtensions`](https://nextjs.org/docs/api-reference/next.config.js/custom-page-extensions).

## Installing Relay Compiler

The Relay Compiler statically analyzes and optimizes GraphQL queries in your
application. To install the dependencies run:

```
npm install --save-dev relay-compiler relay-compiler-language-typescript babel-plugin-relay graphql
```

For convenience create a `package.json` to run the compiler:

```json
{
  "scripts": {
    "generate:relay": "relay-compiler"
  }
}
```

Then configure Babel to compile away `graphql` strings:

`.babelrc`:

```json
{
  "presets": ["next/babel"],
  "plugins": ["relay"]
}
```

`relay-nextjs` is designed to run on both the server and client. To avoid
pulling in server dependencies to the client bundle configure Webpack to ignore
any files in `src/lib/server`. In `next.config.js`:

```js
module.exports = {
  webpack: (config, { isServer, webpack }) => {
    if (!isServer) {
      // Ensures no server modules are included on the client.
      config.plugins.push(new webpack.IgnorePlugin(/lib\/server/));
    }

    return config;
  },
};
```

If your path to server-only files is different please adjust the above RegExp
properly.
