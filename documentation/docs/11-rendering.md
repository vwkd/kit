---
title: Rendering
---

If a setting is specified both per-page and per-app, the per-page setting takes preference. `ssr` and `hydrate` cannot both be `false` since that would result in nothing being rendered at all. Note that each of the per-page settings use [`context="module"`](https://svelte.dev/docs#script_context_module), and only apply to page components, _not_ [layout](#layouts) components.

### ssr

> In most situations disabling [SSR](#appendix-ssr) is not recommended!

Disabling [SSR](#appendix-ssr) turns your SvelteKit app into a [SPA](#appendix-csr-and-spa).

You can disable [SSR](#appendix-ssr) on the page-level using the `ssr` export:

```html
<script context="module">
	export const ssr = false;
</script>
```

To disable SSR app-wide you can use the [`ssr` config option](#configuration-ssr).

### prerender

Your [adapter](#adapters) determines if pages are by default [prerendered](#appendix-prerendering) or dynamically server-side rendered. Currently, only [`adapter-static`](https://github.com/sveltejs/kit/tree/master/packages/adapter-static) prerenders pages by default, while all others adapters default to dynamic SSR.

You can enable/disable [prerendering](#appendix-prerendering) on the page-level using the `prerender` export:

```html
<script context="module">
	export const prerender = true;
</script>
```

To enable/disable prerendering app-wide you can use the [`prerender.enabled` config option](#configuration-ssr).

The prerenderer will crawl your app starting from the root and following all `<a>` elements that point to other pages of the same app. Therefore you generally don't need to specify which pages should be accessed by the prerenderer. If you _do_ have pages which aren't linked to, you can specify them in the `pages` option in the [prerender configuration](#configuration-prerender).

A prerendered page will generate a HTML file, plus any additional files requested by the `load` function. Note that you can still prerender pages that load data based on the page's parameters, like our `src/routes/blog/[slug].svelte` example from earlier, since the prerenderer will intercept requests made inside `load`, so the data served from `src/routes/blog/[slug].json.js` will also be captured.

#### Route conflicts

Because prerendering writes to the filesystem, it isn't possible to have two endpoints that would cause a directory and a file to have the same name. For example, `src/routes/foo/index.js` and `src/routes/foo/bar.js` would try to create `foo` and `foo/bar`, which is impossible.

For that reason among others, it's recommended that you always include a file extension — `src/routes/foo/index.json.js` and `src/routes/foo/bar.json.js` would result in `foo.json` and `foo/bar.json` files living harmoniously side-by-side.

For _pages_, we skirt around this problem by writing `foo/index.html` instead of `foo`.

### router

> In most situations disabling [client-side routing](#appendix-routing) is not recommended!

Disabling [client-side routing](#appendix-routing) will use [server-side routing](#appendix-routing) for any navigation from the page, regardless of whether the router is already active.

You can disable [client-side routing](#appendix-routing) on the page-level using the `router` export:

```html
<script context="module">
	export const router = false;
</script>
```

To disable client-side routing app-wide you can use the [`router` config option](#configuration-router).

### hydration

> In most situations disabling [hydration](#appendix-hydration) is not recommended!

??? Skipping [hydration](#appendix-hydration) disables any interactivity due to JavaScript if it's the initial page.

You can skip [hydration](#appendix-hydration) on the page-level using the `hydrate` export:

```html
<script context="module">
	export const hydrate = false;
</script>
```

If `hydrate` and `router` are both `false`, SvelteKit will not add any JavaScript to the page at all.

You can skip hydration app-wide when the app boots using the [`hydrate` config option](#configuration-hydrate).
