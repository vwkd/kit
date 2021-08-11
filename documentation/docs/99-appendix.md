---
title: Appendix
--- 

### Server-side rendering (SSR)

Server-side rendering (SSR) is the generation of the page contents on the server. The content is contained in the HTML as markup and to show the page the browser can build the DOM directly from the markup. This is how all web pages worked initially. Don't get confused, the page can still contain JavaScript, and therefore personalise the contents on the client-side. This is called hydration.

For the initial page load, SSR improves performance since it can show content faster. Also it makes the page accessible to users without JavaScript, which can fail or be disabled [more often than you might think](https://kryogenix.org/code/browser/everyonehasjs.html). Also SSR is generally preferred for SEO. While some search engines can index content that is dynamically generated on the client-side (see CSR) it may take longer even in these cases. SSR is the preferred way for serving the initial page.

#### Dynamic SSR

Dynamic SSR is the generation of a server-side rendered page at request-time. Such a web page is often also called a dynamic web page, although the term may be confusing because client-side JavaScript can make it appear dynamic as well.

Dynamic SSR can make sense for pages that need to personalise the contents of the page on the server for different requests, and can't use JavaScript on the client.

Dynamic SSR can make sense for certain types of web sites, where pages can't be prerendered. For example, if pages change too quickly like on social networks or news sites, or if pages are simply too many to store like in encyclopedias. But in most cases even those sites can have certain pages prerendered that change less often, making it valuable to be able to specify prerendering on a page-to-page basis.

#### Prerendering

Prerendering is the generation of a server-side rendered page at build-time. Such a web page is often also called a static web page, although the term may be confusing because client-side JavaScript can make it appear dynamic. Web sites with only prerendered pages are called static sites. The tool for generating a static site is called a Static Site Generator (SSG).

Prerendering is more performant and efficient than Dynamic SSR, since it scales independently of the number of requests. Hosting for static sites is as simple as it gets and many static site hosts offer it even for free. This delivery model is often referred to as JAMstack.

As a tradeoff, prerendering allows for less frequent updates to the page and also requires more storage.

Prerendered pages don't personalise the contents of the page on the server for different requests but can use JavaScript on the client. The rule of thumb is: for content to be prerenderable, any two requests must get the same content. Note that you can still prerender content if it is personalised using the data contained in the request itself like the parameters in the query string of the URL.

### Client-side rendering (CSR)

Client-side rendering (CSR) is the generation of the page contents on the client. The content is contained in the HTML as JavaScript instead of as markup and to show the page the browser needs to execute the JavaScript to build the DOM.

For the initial page load, CSR is slower than SSR because of the additional size of the supporting JavaScript engine and the time needed to execute it. However, for subsequent pages it can be more efficient if used together with client-side routing (see Routing).

### Routing

Routing is the navigation to a new page of the same website on the client.

Server-side routing uses the built-in browser navigation. The browser loads the new page as initial page, the same as if the previous page had never been there. No state is preserved and the DOM is built again from scratch. This is how navigation clasically works in SSR pages. The advantage of this is that it works out of the box. A user interaction with the navigation elements like the anchor element or the forward and back buttons, the history, the accessibility like moving focus back the top and reading the new title.

Client-side routing uses JavaScript for navigation. JavaScript loads the new contents and modifies the DOM of the existing page. To the browser it seems like there is only ever the single initial page. JavaScript just manipulates that single page to make it look like distinct pages.

For subsequent navigation, this is more efficient since it only needs to load the changed contents without the page boilerplate. It can also preserve state, intelligently prefetch pages in advance, and animate page transitions. The navigation can be initiated by JavaScript, or by the user interacting with the built-in browser navigation elements which JavaScript needs to intercept. JavaScript also needs to change the URL in the browser bar, update the browser history, and preserve accessibility. Client-side routing requires CSR and therefore both are almost always used together.

### SPA

A Single-Page App (SPA) is a web site that uses client-side routing without SSR the initial page. Initially, frameworks didn't include support for SSR but nowadays most frameworks have good support for SSR.

There are certain cases in which a SPA still makes sense where the tradeoff of simplicity against performance and efficiency is worth it, like a complex business application behind a login where SEO isn't important and users are accessing the application from a consistent computing environment.
