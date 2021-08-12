---
title: Adapters
---

An adapter is a small plugin that takes the built app as input and generates output for a deployment platform. It is specified in [`config.kit.adapter`](#configuration) and is run using the [svelte-kit build](#command-line-interface-svelte-kit-build) command.

For example, `adapter-static` generates a static site that can be hosted on any static site hosting provider. Other adapters are more specific to the deployment platform, like generating cloud functions and configuration.

### Serverless platforms

- [`adapter-cloudflare-workers`](https://github.com/sveltejs/kit/tree/master/packages/adapter-cloudflare-workers) — for [Cloudflare Workers](https://developers.cloudflare.com/workers/)
- [`adapter-netlify`](https://github.com/sveltejs/kit/tree/master/packages/adapter-netlify) — for [Netlify](https://netlify.com)
- [`adapter-vercel`](https://github.com/sveltejs/kit/tree/master/packages/adapter-vercel) — for [Vercel](https://vercel.com)

### Traditional platforms

- [`adapter-node`](https://github.com/sveltejs/kit/tree/master/packages/adapter-node) — for creating self-contained Node apps
- [`adapter-static`](https://github.com/sveltejs/kit/tree/master/packages/adapter-static) — for prerendering your entire site as a collection of static files

### Other

- [community-provided adapters](https://sveltesociety.dev/components#category-SvelteKit%20Adapters)
- [write your own adapter](#writing-an-adapter)
