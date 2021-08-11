---
title: Loading
---

The `load` function in a component runs before the component is rendered. It can be used to initialise the component by preloading data using `fetch` that is always ready when the component is rendered.

It can be exported from the `<script context="module">` in [page](#routing-pages) and [layout](#layouts) components. Note child components imported by a page or layout compopnent don't have a `load` function.

Our example blog page `/src/blog/[slug].svelte` might contain a `load` function that fetches from our endpoint (or any API) like this...

```html
<script context="module">
	export async function load({ page, fetch }) {
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

You might wonder why we use a `fetch` function in the argument. The `fetch` function is a custom wrapper around the native `fetch` Web API that allows SvelteKit to do some Svelte magic.

The `load` function runs both on the server during SSR and on the client during CSR. During SSR the responses to any `fetch` requests in `load` are saved in the generated page. After hydration on the client, the client-side runtime returns the saved responses to the `fetch` requests in the `load` function when it runs. On the client there is no network request. The only actual network request is on the server. To the component it only looks like the request happens on the client.

This also explains why when rendering the component on the client the data is always ready, no matter how long the fetch takes. This is because there is no fetch. The data is already there on the client because the fetch took place on the server earlier. To the client the response is always instant, even if the API is slow since the waiting is done on the server. And even better for prerendered pages since the waiting is done during build-time. The amount of fetch requests doesn't matter either since they are all included in the same generated page.

Make sure you don't fetch APIs using secrets directly from `load`, since the code is transferred to the client. Instead use [endpoints](#routing-endpoints).

The `load` function is the preferable way to fetch data. The `onMount` function runs only on the client meaning the client has to wait for the network.

> Make sure to use the custom `fetch` wrapper instead of the global `fetch`. Otherwise the client makes an ordinary request to the network and fails when fetching endpoints since they don't exist on the client. This would make `load` behave like `onMount`.

> Since the `load` function runs on also on the server, make sure it doesn't use any browser-specific objects like `window` or `document`.

It is recommended that you not store pre-request state in global variables, but instead use them only for cross-cutting concerns such as caching and holding database connections.

> Mutating any shared state on the server will affect all clients, not just the current one.









??? What happens if SSR is off?


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

The return value is an object with several properties, all optional. A promise will be awaited and delay rendering until it resolves.

If nothing is returned, SvelteKit will [fall through](#routing-advanced-fallthrough-routes) to other routes until it finds a `load` function that responds. If no route responds it will respond with a generic 404.

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
