---
title: Routing
---

There are two types of routes: **pages** and **endpoints**.

### Pages

Pages typically generate HTML to display to the user (as well as any CSS and JavaScript needed for the page). By default, pages are rendered on both the client and server, though this behaviour is [configurable](#rendering).

Pages are Svelte components with a `.svelte.` extension in the `src/routes` directory of your project. You can customise the extension in [`config.extensions`](#configuration) and the directory in [project config](#configuration).

SvelteKit uses a _filesystem-based router_. This means that the structure of your application is defined by the structure of your codebase — specifically, the contents of `src/routes`. The filename determines the route.

For example, `src/routes/index.svelte` is the root of your site:

```html
<!-- src/routes/index.svelte -->
<svelte:head>
	<title>Welcome</title>
</svelte:head>

<h1>Hello and welcome to my site!</h1>
```

A file called either `src/routes/about.svelte` or `src/routes/about/index.svelte` would correspond to the `/about` route:

```html
<!-- src/routes/about.svelte -->
<svelte:head>
	<title>About</title>
</svelte:head>

<h1>About this site</h1>
<p>TODO...</p>
```

Dynamic parameters are encoded using `[brackets]`. For example, a blog post might be defined by `src/routes/blog/[slug].svelte`. Soon, we'll see how to access that parameter in a [load function](#loading) or the [page store](#modules-$app-stores).

A file or directory can have multiple dynamic parts, like `[id]-[category].svelte`. (Parameters are 'non-greedy'; in an ambiguous case like `x-y-z`, `id` would be `x` and `category` would be `y-z`.)

### Endpoints

Endpoints run only on the server at request-time or at build-time if [prerendering](#rendering-prerender). This means it's the place to do things like access databases or APIs that require private credentials or return data that lives on a machine in your production network. Pages can request data from endpoints. Endpoints return JSON by default, though may also return data in other formats.

Endpoints are modules written in `.js` (or `.ts`) files that export functions corresponding to HTTP methods, like `get` for `GET` requests and `post` for `POST` requests. Since `delete` is a reserved word in JavaScript, DELETE requests are handled with a `del` function. These functions return

???? All server-side code, including endpoints, has access to `fetch` in case you need to request data from external APIs. ??? WHAT IS OTHER SS CODE EXCLUDING ENDPOINTS?

For example, we might have an endpoint `src/routes/blog/[slug].json.js` that returns a blog post as JSON. Then a hypothetical blog page like `/blog/cool-article` would request data from `/blog/cool-article.json`, which calls the endpoint with the slug `cool-article`

```js
// Don't worry about `$lib`, we'll get to that [later](#modules-$lib).
import db from '$lib/database';

export async function get({ params }) {
	// the `slug` parameter is available because this file
	// is called [slug].json.js
	const { slug } = params;

	const article = await db.get(slug);

	if (article) {
		return {
			body: {
				article
			}
		};
	}
}
```

#### Response

The job of an endpoint function is to return a `{ status, headers, body }` object representing the response.

We don't interact with the `req`/`res` objects you might be familiar with from Node's `http` module or frameworks like Express, because they're only available on certain platforms. Instead, SvelteKit translates the returned object into whatever's required by the platform you're deploying your app to.

The property `status` is an [HTTP status code](https://httpstatusdogs.com) which defaults to `200`. Returning nothing is equivalent to an explicit 404 response.

???? For successful responses, SvelteKit will generate 304s automatically.

If the returned `body` is an object, and no `content-type` header is returned, it will automatically be turned into a JSON response.

To set multiple cookies in a single set of response headers, you can return an array:

```js
return {
	headers: {
		'set-cookie': [cookie1, cookie2]
	}
};
```

#### Body parsing

The `body` property of the request object will be provided in the case of POST requests:
- Text data (with content-type `text/plain`) will be parsed to a `string`
- JSON data (with content-type `application/json`) will be parsed to a `JSONValue` (an `object`, `Array`, or primitive).
- Form data (with content-type `application/x-www-form-urlencoded` or `multipart/form-data`) will be parsed to a read-only version of the [`FormData`](https://developer.mozilla.org/en-US/docs/Web/API/FormData) object.
- All other data will be provided as a `Uint8Array`

### Private routes

A filename that has a segment with a leading underscore, such as `src/routes/foo/_Private.svelte` or `src/routes/bar/_utils/cool-util.js`, is hidden from the router, but can be imported by files that are not.

### Advanced

#### Rest parameters

A route can have multiple dynamic parameters, for example `src/routes/[category]/[item].svelte` or even `src/routes/[category]-[item].svelte`. If the number of route segments is unknown, you can use rest syntax — for example you might implement GitHub's file viewer like so...

```bash
/[org]/[repo]/tree/[branch]/[...file]
```

...in which case a request for `/sveltejs/kit/tree/master/documentation/docs/01-routing.md` would result in the following parameters being available to the page:

```js
{
	org: 'sveltejs',
	repo: 'kit',
	branch: 'master',
	file: 'documentation/docs/01-routing.md'
}
```

> `src/routes/a/[...rest]/z.svelte` will match `/a/z` as well as `/a/b/z` and `/a/b/c/z` and so on. Make sure you check that the value of the rest parameter is valid.

#### Fallthrough routes

Finally, if you have multiple routes that match a given path, SvelteKit will try each of them until one responds. For example if you have these routes...

```bash
src/routes/[baz].js
src/routes/[baz].svelte
src/routes/[qux].svelte
src/routes/foo-[bar].svelte
```

...and you navigate to `/foo-xyz`, then SvelteKit will first try `foo-[bar].svelte` because it is the best match, then will try `[baz].js` (which is also a valid match for `/foo-xyz`, but less specific), then `[baz].svelte` and `[qux].svelte` in alphabetical order (endpoints have higher precedence than pages). The first route that responds — a page that returns something from [`load`](#loading) or has no `load` function, or an endpoint that returns something — will handle the request.

If no page or endpoint responds to a request, SvelteKit will respond with a generic 404.
