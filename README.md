# Environment Variables

Define the way JS reads environment variables across different runtimes:

```js
const myvar = import.meta.env.MY_VARIABLE;
```

## Status

Authors: Francisco Presencia

Champions: Francisco Presencia

~This proposal is at Stage 0 of The TC39 Process.~ This proposal is for WinterCG.

## Motivation

Javascript has grown a lot and now it's used in a large variety of runtimes, environments and systems ("runtimes" from now on). While ECMASCript has ensured the language is the same for all of them, there are many parts that are left to each runtime to decide. This includes things like lifecycle of the application, interaction with the host OS, some globals, etc. This makes writing universal code (also called isomorphic) to be a challenge some times; tc39 has made great advances with the standards and so today in virtually all environments you can use `fetch()`, while this was very fragmented just 5 years ago and you needed to write or import large amounts of code to achieve the same thing.

Which brings us to this proposal. Today, to use environment variables (which can come from a variety of places) you have to write different code for different environments, some examples:

```js
// Node, the classic (+many others), but this depends on `process` which is not a standard
process.env.MY_VARIABLE;

// React (like Node, with mandatory prefix)
process.env.REACT_APP_MY_VARIABLE;

// Bun (though the Node ones also work)
Bun.env.MY_VARIABLE;

// Netlify
Netlify.env.get('MY_VARIABLE');

// Cloudflare Workers
async fetch(request, env, ctx) {
  env.MY_VARIABLE; // ??
}
```

## Proposal

My proposal is to create a single place where the Environment Variables are all already loaded from the environment:

```js
import.meta.env.MY_VARIABLE;
```

Note: props to Vite, since they are the ones who started using this first AFAIK.

This is the way that Vite does it, and it seems to be a great way to use `import.meta`, a place that has already been used in the past to load things that were left to `process` or other meta-variables in the past.

Specifically:

- Define the read-only property `env` in `import.meta` as a plain object.
  - You cannot change it with `import.meta.env = '...';` (INVALID).
  - However, individual keys and values can be overwritten, e.g. `import.meta.env.MY_DB = 'hello';` (VALID).
- The keys are the environment variable keys, which are strings since they are object keys (usually in uppercase, but this spec does not force it).
- The values are all defined as **strings**. The environment should not attempt to parse them or cast them into different types.
- If there is none, then `import.meta.env` should be an empty object `{}`.
- We purposefully do not specify where these variables come from, that is left to the specific runtime.

## Example

If we are working with Node and a single `index.js` file, this should happen:

```js
console.log(import.meta.env);
// {}
```

However if we create the file `.env` as this:

```text
HELLO=world
DB=https://mydb.com/connect
```

And then load the same index.js as above, but with the flag `node --env-file=.env index.js`:

```js
console.log(import.meta.env);
// { HELLO: "world", DB: "https://mydb.com/connect" }
```

## Definitions

- Environment Variable: a variable set by the runtime (Node.js, Bun, Cloudflare Worker, Netlify Function, etc) that is accessible by Javascript. This is usually different in different environments on purpose, e.g. in a local dev server you'd have a different DB access than in a remote production server.


