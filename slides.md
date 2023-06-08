---
theme: apple-basic
highlighter: shiki
lineNumbers: false
rawings:
  persist: false
transition: slide-left
css: unocss
title: Svelte workshop
---
## Svelte!!!
Introshop to svelte and sveltekit

### Agenda

1. Basic svelte
2. Things you asked about
3. Your questions

> Get a copy of the code at https://github.com/iv-ar/svelte-workshop

---
title: Data binding
---

# Data binding

Data binding in svelte refers to the ability to have a two-way data flow between a reader and writer, typically form inputs

The most basic form of data binding:

```svelte
<script>
    // This variable will always have the latest value of the input field
    let value
    // This variable will have a reference to the DOM object which is the input
    let input
</script>

<input bind:value bind:this={input} />
```
There are alot of different rules regarding which properties can be bound and not, therefore I suggest using [the docs](https://svelte.dev/docs#template-syntax-element-directives-bind-property) for reference.
<!--
Since the variable is named the same as the thing we want to bind, we don't need the ={value}
Take a look at the group bindings!
-->

---
title: Conditionally apply classes
---

# Conditionally apply classes

Svelte has builtin support to conditionally apply classes
```svelte
<script>
    let yes = true
    let no = false
</script>
<h1 class:big={yes} class:upsideDown={no} class="h1">
Heading
</h1>
```
<!--
So long as you don't have the need for a lot of conditional classnames, cn can be switched out with the built-in functionality
-->
---
title: Preprocessing
---

# Preprocessing
- vitePreprocess
  - Comes out of the box with support for typescript, scss, postcss, etc., just install `typescript` and `sass` from npm
- svelte-preprocess
  - Normally not needed anymore, unless you need things like pug
<!--
Historcally we were depedent on svelte-preprocess, but now we get vitePreprocess out of the box
-->

---
title: The .svelte file
---

# The .svelte file

The .svelte file denotes a svelte component (or page in kit), and usually contains three sections
- scripts
- styles
- template (aka markup)

`<script>` and `<style>` tags accept a `lang` attribute, which tells the preprocessor to preprocess the contents, for example `lang=ts` for typescript and `lang=scss` for scss.

You can also have `context=module` on a seperate `<script>` tag, this is handy for functions without side-effetcs, and for type definitions.

Everything in the `context=module` is available in the "main" script, and can also be exported as a part of the component itself.

---
title: Reactivity
---
# Reactivity

Any `let`'s in a svelte component is reactive by default, all you need is to assign the variable and everything that references that variable will update

To react to changes we have reactive statements, they are denoted with the `$:` syntax.

Example:
```svelte
<script>
let value
// This will evaluate every time the value variable changes
$: canSubmit = value?.length > 7
</script>
<input type="tel" bind:value>
<button disabled={!canSubmit}>submit</button>
```

---
title: Component props
---

# Component props

Any exported `let` is a settable property to that component.

Any exported `const` is a readable property for that component.

```svelte
<script lang="ts">
export let prop1: Typed | undefined;
export let propWithDefault = false
export const id = crypto.randomUUID();
</script>
```
<!--
    let's are most common for obvious reasons
    const are handy if you for example generate an id for your component that the parent should know about, or for callbacks.
-->

---
title: Events
---

# Component events

Any event can be hooked onto by using the `on:eventname={handler}` syntax.
```svelte
<script>
function handle(event) {}
</script>
<h1 on:mouseover={handle}>Heading</h1>
```

Components can dispatch their own events by creating a eventDispatcher:
```svelte
<script>
	import { createEventDispatcher } from 'svelte';
	const dispatch = createEventDispatcher();
</script>
<button on:click="{() => dispatch('notify', 'detail value')}">Fire Event</button>
```
Components can also pass their dom events along by omitting a handler
```svelte
<!--CoolButton.svelte-->
<button on:click><slot/></button>
```

---
title: Stores
---

# Stores

A store in svelte gives you the ability to store data in a central place and subscribe and update its value from anywhere.

---
title: Images
---

# Display images in svelte

> There is no `next/image` for svelte

Svelte leaves image rendering up to the you and the browser, this means that we don't get anything for free ootb.

Alternatives:
- Use `<picture>` and `<img>` directly, if you can
- Write your own; You can leverage svelte components to write complex markup for images
- Use https://github.com/zerodevx/svelte-img

<!--
In svelte and sveltekit we don't have a preffered way to handle images (like `next/image`)
Usually you can get away with writing a image component yourself but if you need niceties like cropping, srcsets, placeholders and optimized formats then svelte-img can be a great companion.
-->

---
title: Data fetching in svelte
---

# Data fetching in svelte

In svelte (not sveltekit) we will almost always be in the browser and therefore can use any library that uses `fetch` or `XMLHttpRequest` to get our data.

> Pure `fetch` is the best option!

Normally, we use `onMount` to populate variables with the data that we use in our templates

Example:
```svelte
<script>
    import { onMount } from "svelte"
    let data
    onMount(async () => {
        data = await (await fetch("https://jsonplaceholder.typicode.com/users/1")).json()
    })
</script>
<pre>{JSON.stringify(data, null, 2)}</pre>
```
---
title: Data fetching in svelte (await block)
---
# Data fetching in svelte

But we also have the `{#await}` block availabe, this syntax enables us to conveniently handle the different states of data fetching.

Example:
```svelte
<script>
	async function theData() {
		const request = await fetch("https://jsonplaceholder.typicode.com/users/1");
		return request.json();
	}
</script>
{#await theData()}
  <!-- promise is pending -->
  <p>Loading users...</p>
{:then value}
  <!-- promise was fulfilled -->
  <pre>{JSON.stringify(value,null,2)}</pre>
{:catch error}
  <!-- promise was rejected -->
  <p>Something went wrong: {error.message}</p>
{/await}
```

---
title: Data fetching in svelte (summary)
---

# Data fetching in svelte, summary

- Use `fetch`
- Initial data should be loaded in `onMount` or in a `{#await}` block
- Eventual data can be loaded seperate from the svelte lifetime (i.e. using variables and functions)

Eventual example:
```svelte
<script>
let data
async function loadData() {
    data = await (await fetch("//data.com")).json();
}
</script>
<pre>{JSON.stringify(data,null,2)}</pre>
<button on:click={loadData}>load</button>
```


---
title: Data fetching in SvelteKit
---

# Data fetching in SvelteKit

- In SvelteKit we can use `onMount` and `{#await}` as with normal svelte.
- Load page data with `+page.ts` and `+page.server.ts`

Let's look at `+page.ts` first...

<!--

1. Use the normal svelte way to load data on a component basis, consider if the data should be coming in as a prop from the page data.
   1. Example of component data is current weather for a given lat and long
   2. Example of component data that could be a prop to a component, is the metadata for a given blog article
2. Use page.ts and page.server.ts to load page data before it is mounted.
   1. Example of this is the content of a blog article
   2. Images that should be shown
   3. Basically most cms data

-->

---
title: +page.ts
---

# Loading data with `+page.ts`

- Exports a function that you use to return an object with your page data
- Normally runs at runtime on the server, unless the page option `ssr` is set to `false`, then it will run on the client.
- Ran at build time if your page is prerendered

---
title: +page.ts example
---

# Loading data with `+page.ts`

```ts
import type { PageLoad } from './$types';

export const load = (async ({ data, fetch }) => {
    const dataFromApi = await (await fetch("//data.example")).json()
    return {
        article: {
            title: "Hello...",
            body: "World"
        },
        dataFromApi,
        ...data
    };
}) satisfies PageLoad;
```
<!--
1. Getting some data from an external api, that does not require authentication
2. We are passing the data object along, this object contains the data from +paige.server.ts, data is not passed automatically
-->

---
title: +page.ts example with params
---

# Loading data with `+page.ts`

```ts
import type { PageLoad } from './$types';

export const load = (async ({ params, parent, data, url }) => {
    // data from your +layout.ts (and in this case +layout.server.ts because we pass it from +layout.ts)
    const layout = await parent()
    layout.layoutData;
    layout.serverLayoutData;
    // url is a web api URL object you can for example get search params from, example: /?q=asdf
    const query = url.searchParams.get("q"); // query = asdf;
    // params contains the params availabe from the current route, in this case we have on called aparam (since we are in a folder named [aparam])
    const {aparam} = params
    return {
        slug: aparam,
        query: query,
        slugFromServer: data.param,
        dataFromLayout: layout
    };
}) satisfies PageLoad;
```
<!--
1. Getting the nearest layout data by using parent
2. Getting a query param from the url object
3. Getting aparam from the params object
-->

---
title: +page.ts vs +layout.ts
---

# +page.ts vs +layout.ts

The `+page.ts` and `+layout.ts` files behave largely in a similar matter

<!--
The main difference being that layout files apply to their respective child/sibling pages, and expose their data to the +page.ts files via the `parent` function we saw earlier
-->

---
title: +page.server.ts
---

# .server.ts

The `+page|layout.server.ts` files work mostly in the same way as `+page|layout.ts`, but they are guaranteed to run on the server and does not rerun without a page load.

These files (mainly the `+page.server.ts`) is where you will be getting sanity data from.

> If you only have a +page.svelte and a +page.server.ts, SvelteKit will pass the data from +page.server.ts automatically

---
title: +page.server.ts
---

# Getting a blog article from sanity

Route: "/blog/[slug]"
File: +page.server.ts
```ts
import type { PageServerLoad } from './$types';
import { sanity } from '$lib/sanity-client';

export const load = (async ({ params }) => {
    const client = sanity();
    const { slug } = params;
    return {
        content: client.blog.find(slug)
    };
}) satisfies PageServerLoad;
```

---
title: Page options
---

# Page options

SvelteKit has six page options, but the most important are: `ssr`, `csr` and `prerender`.
They are most commonly set by exporting the config in a `+page.ts` or `+layout.ts`.
```ts
export const ssr = false;
export const csr = false;
export const prerender = true;
```
When set in a `+layout.ts`, the config will apply to all child pages unless otherwise configured with a child `+layout.ts` or `+page.ts`.


---
title: SSR
---

# Server side rendering (`ssr`)

This option tells SvelteKit to allow or disallow server side rendering the page(s).

In practice this means that users will get a blank page initially and sveltekit will fetch content from `+page.server.ts` after the page has loaded.

It also tells `+page.ts` to run it's load function on the client.

---
title: CSR
---

# Client side rendered (`csr`)

If you don't need javascript you can safely disable csr, as this option tells sveltekit to hydrate your page or not.

> If both csr and ssr are false, nothing will render.

---
title: Prerender
---

# Prerendering (`prerender`)

- Generate html/page at build time
- .server.ts will not run
- Load function in +page.ts is ran at build time

<!--

By setting `prerender` to true you are telling sveltekit to generate the html for your page at build time, meaning that you can't have any .server.ts associated with that page, and the load function in +page.ts is ran at build time.

-->

---
title: Hooks
---

# Hoooks

The docs say it best:

> 'Hooks' are app-wide functions you declare that SvelteKit will call in response to specific events, giving you fine-grained control over the framework's behaviour.

Example use cases:
- Load data you will need as early as possible
  - Set user specific locals like date time, localised strings, session
- Intercept sveltekits fetch calls
  - Authorize fetch calls based on user cookies
  - Set shared headers
- Log errors to a third party
- Transform resulting html with transformPageChunk

<!--
I will touch on the server side hooks that we can use to intercept http requests, but i encourge you to check out the docs to get a more wide understanding and inspiration for things to do with hooks.
-->


---
title: hooks.server.ts
---
# `hooks.server.ts`

This file is placed at the root of your `routes` folder and it's `handle` function is ran every time sveltekit receives an http request.

```ts
import type { Handle } from "@sveltejs/kit";

export const handle: Handle = ({ resolve, event }) => {
    // event.locals is availabe as a parameter to the load function in all subsequent .server.ts files
    // All properties on locals should be defined in your app.d.ts file, assuming you are using typescript
    event.locals.handled = true;
    // With event.cookies you have full control over the cookies the client sends with the user's request
    const session = event.cookies.get("session");
    // you are also free to set and update the cookies
    event.cookies.delete("session");
    event.cookies.set("guid", crypto.randomUUID());
    // event.request gives you the web native Request object for the given request, that you can use to read headers for example
    const isAdmin = event.request.headers.get("X-IsAdmin");
    return resolve(event);
}
```
