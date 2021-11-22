---
title: Anchor options
---

### sveltekit:prefetch

When navigating to one of our pages the client-side router currently doesn't automatically prefetch the new page. To enable prefetching add a `sveltekit:prefetch` attribute to a link...

```html
<a sveltekit:prefetch href="blog/what-is-sveltekit">What is SvelteKit?</a>
```

With prefetching enabled, the client-side router loads the target page's `load` function as soon as the user hovers over a link on desktop or touches it on mobile, rather than waiting for the `click` event to fire. Note that prefetching has no effect if the target page doesn't export a `load` function or if the [`router`](#rendering-router) setting is `false`.

You can also programmatically invoke `prefetch` from `$app/navigation`.

### sveltekit:noscroll

When navigating to one of our pages the client-side router scrolls to the top of the page or to the element with a matching ID if the link includes a `#hash` to mirror the browser's default behavior.

You can disable scrolling after the link is clicked by adding a `sveltekit:noscroll` attribute to a link...

```html
<a href="path" sveltekit:noscroll>Path</a>
```

### rel=external

By default, further navigation to one of our pages is routed by the [client-side router](#rendering-routing).

You can disable client-side routing for a specific link by adding a `rel=external` attribute...

```html
<a rel="external" href="/path">Path</P>
```
