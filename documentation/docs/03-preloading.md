---
title: Preloading
---

The `load` function
[page](#routing-pages) and [layout](#layouts) components can export a `load` function from `<script context="module">` which runs right before the component is rendered.

During SSR, all `fetch` responses are cached and baked into the rendered page. After hydration on the client, the client-side runtime takes the responses and injects it into `fetch` calls in the `load` function.

When `fetch` runs on the server, the resulting response will be serialized and inlined into the rendered HTML. This allows the subsequent client-side `load` to access identical data immediately without an additional network request.

- it has access to cookies on the server
- it can make requests against the app's own endpoints

This is why `load` receives a custom `fetch`.
> You must use the `fetch` wrapper, otherwise `load` won't work as expected during SSR!


it doesn't mean you're making unnecessary network requests.
Anything you fetch in your load function is baked into the server-rendered HTML
So the actual network request happens on the server. It just seems like it happens on the client.


It runs both during SSR on the server and during CSR on the client. During SSR, all `fetch` responses are baked into the SSR rendered page. Then during CSR the client-side runtime serves.  can only be called through the load function.






Note child components imported by a page or layout compopnent don't have a `load` function.

Note if you instead load your data in a normal `<script>` then ???

Code that is per-component instance should go into the normal `<script>` tag. 

It can be used to already have the data ready when the component is created.



This replaces fetching data on the client in the `onMount` function and showing a loading spinner.


Load has access to a `fetch` in case you need to request data from external APIs.





Code called inside `load` blocks:

- should use the SvelteKit-provided [`fetch`](#loading-input-fetch) wrapper rather than using the native `fetch`
- should not reference `window`, `document`, or any browser-specific objects
- should not directly reference any API keys or secrets, which will be exposed to the client, but instead call an endpoint that uses any required secrets
You shouldn't be talking directly to the database in load, you should be creating an endpoint and requesting data with fetch.



///

An endpoint is a private API that is only available to your pages. Endpoints are the place to do things like access databases or APIs that require private credentials or return data that lives on a machine in your production network.

An endpoint doesn't exist as a resource at the URL on the WWW. Its responses are precomputed during SSR along with the pages themselves. The runtime on the client then emulates the responses to your pages as if they were resources that actually existed at a URL.

You can think of an endpoint as a page with a `load` function that gets SSR but never ends up on the client.

What actually happens during SSR is the SvelteKit compiler calls the endpoint function for every `fetch` call in a page and stores the results in the generated page that will be served to the client. After hydration on the client, the client-side runtime intercepts `fetch` requests of the pages to the endpoint and returns the precomputed results. There is no actual HTTP call going on.

??? can fetch only in `load` or also in normal `script`?

Endpoints return JSON by default, though may also return data in other formats.

??? What happens if SSR is off?

Endpoints are modules with a `.js` or `.ts` extension and filenames similar to pages. A module exports functions corresponding to HTTP methods, like `get` for `GET` requests and `post` for `POST` requests. Since `delete` is a reserved word in JavaScript, DELETE requests are handled with a `del` function. 

Endpoints return JSON by default, though may also return data in other formats.

Endpoints has access to a `fetch` in case you need to request data from external APIs.
??? HOW `this.fetch` or through argument like in `load` ?

/////



If `load` returns nothing, SvelteKit will [fall through](#routing-advanced-fallthrough-routes) to other routes until it finds a `load` function that responds. If no route responds it will respond with a generic 404.

Our example blog page might contain a `load` function like the following. Note the actual `fetch` request takes place on the server.

```html
<script context="module">
	export async function load({ page, fetch, session, context }) {
		const url = `/blog/${page.params.slug}.json`;
		const res = await fetch(url);

		if (res.ok) {
			return {
				props: {
					article: await res.json()
				}
			};
		}

		return {
			status: res.status,
			error: new Error(`Could not load ${url}`)
		};
	}
</script>
```

It is recommended that you not store pre-request state in global variables, but instead use them only for cross-cutting concerns such as caching and holding database connections.

> Mutating any shared state on the server will affect all clients, not just the current one.



### Input

The `load` function receives an object containing four fields — `page`, `fetch`, `session` and `context`. The `load` function is reactive, and will re-run when its parameters change, but only if they are used in the function. Specifically, if `page.query`, `page.path`, `session`, or `context` are used in the function, they will be re-run whenever their value changes. Note that destructuring parameters in the function declaration is enough to count as using them. In the example above, the `load({ page, fetch, session, context })` function will re-run every time `session` or `context` is changed, even though they are not used in the body of the function. If it was re-written as `load({ page, fetch })`, then it would only re-run when `page.params.slug` changes. The same reactivity applies to `page.params`, but only to the params actually used in the function. If `page.params.foo` changes, the example above would not re-run, because it did not access `page.params.foo`, only `page.params.slug`.

#### page

`page` is a `{ host, path, params, query }` object where `host` is the URL's host, `path` is its pathname, `params` is derived from `path` and the route filename, and `query` is an instance of [`URLSearchParams`](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams).

So if the example above was `src/routes/blog/[slug].svelte` and the URL was `https://example.com/blog/some-post?foo=bar&baz&bizz=a&bizz=b`, the following would be true:

- `page.host === 'example.com'`
- `page.path === '/blog/some-post'`
- `page.params.slug === 'some-post'`
- `page.query.get('foo') === 'bar'`
- `page.query.has('baz')`
- `page.query.getAll('bizz') === ['a', 'b']`

#### fetch

`fetch` is a wrapper around the native `fetch` web API, and can make credentialed requests. It can be used across both client and server contexts.


> Cookies will only be passed through if the target host is the same as the SvelteKit application or a more specific subdomain of it.



#### session

`session` can be used to pass data from the server related to the current request, e.g. the current user. By default it is `undefined`. See [`getSession`](#hooks-getsession) to learn how to use it.

#### context

`context` is passed from layout components to child layouts and page components. For the root `__layout.svelte` component, it is equal to `{}`, but if that component's `load` function returns an object with a `context` property, it will be available to subsequent `load` functions.

### Output

If you return a Promise from `load`, SvelteKit will delay rendering until the promise resolves. The return value has several properties, all optional:

#### status

The HTTP status code for the page. If returning an `error` this must be a `4xx` or `5xx` response; if returning a `redirect` it must be a `3xx` response. The default is `200`.

#### error

If something goes wrong during `load`, return an `Error` object or a `string` describing the error alongside a `4xx` or `5xx` status code.

#### redirect

If the page should redirect (because the page is deprecated, or the user needs to be logged in, or whatever else) return a `string` containing the location to which they should be redirected alongside a `3xx` status code.

#### maxage

To cause pages to be cached, return a `number` describing the page's max age in seconds. The resulting cache header will include `private` if user data was involved in rendering the page (either via `session`, or because a credentialed `fetch` was made in a `load` function), but otherwise will include `public` so that it can be cached by CDNs.

This only applies to page components, _not_ layout components.

#### props

If the `load` function returns a `props` object, the props will be passed to the component when it is rendered.

#### context

This will be merged with any existing `context` and passed to the `load` functions of subsequent layout and page components.

This only applies to layout components, _not_ page components.
