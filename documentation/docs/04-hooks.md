---
title: Hooks
---

Hooks allow to customise the behavior of SSR.

Hooks are functions exported from a `src/hooks.js` (or `src/hooks.ts`, or `src/hooks/index.js`) file. The location of this file can be configured using [`config.kit.files.hooks`](#configuration).

### handle

The `handle` function allows to determine the response of pages and endpoints.

It receives the `request` object and a function called `resolve`, which invokes SvelteKit's router and generates a response accordingly. This allows you to modify response headers or bodies, or bypass SvelteKit entirely (for implementing endpoints programmatically, for example).

If unimplemented, defaults to `({ request, resolve }) => resolve(request)`.

To add custom data to the request, which is passed to endpoints, populate the `request.locals` object, as shown below.

```js
export async function handle({ request, resolve }) {
	request.locals.user = await getUserInformation(request.headers.cookie);

	const response = await resolve(request);

	return {
		...response,
		headers: {
			...response.headers,
			'x-custom-header': 'potato'
		}
	};
}
```

### getSession

The `getSession` function determines the initial value of [session](#modules-$app-stores) accessible on the client on a page.

It takes the `request` object and returns a `session` object. The `session` object must be serializable, which means it must not contain things like functions or custom classes, just built-in JavaScript data types.

If unimplemented, session is `{}`.

```js
export function getSession(request) {
	return request.locals.user ? {
		user: {
			// only include properties needed client-side —
			// exclude anything else attached to the user
			// like access tokens etc
			name: request.locals.user.name,
			email: request.locals.user.email,
			avatar: request.locals.user.avatar
		}
	} : {};
}
```

> The `session` object must be safe to expose to users.

### serverFetch

The `serverFetch` allows you to modify (or replace) a `fetch` request for an **external resource** inside a `load` function on a page.

For example, your `load` function might make a request to a public URL like `https://api.yourapp.com` when the user performs a client-side navigation to the respective page, but during SSR it might make sense to hit the API directly (bypassing whatever proxies and load balancers sit between it and the public internet).

```js
export async function serverFetch(request) {
	if (request.url.startsWith('https://api.yourapp.com/')) {
		// clone the original request, but change the URL
		request = new Request(
			request.url.replace('https://api.yourapp.com/', 'http://localhost:9999/'),
			request
		);
	}

	return fetch(request);
}
```
