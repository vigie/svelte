# Fork of Svelte

https://github.com/sveltejs/svelte

# What's different?

This is a minor bug fix to Svelte to make it compatible with the ShadyDom pollyfill. It also continues to work with native shadow DOM.

# What was the issue?

If you try to use regular Svelte with the ShadyDom polyfill to create custom elements, you will encounter

`Uncaught DOMException: "Failed to construct 'CustomElement': The result must not have children"`

this patch fixes that by moving some of the init logic from the constructor to the `connectedCallback()` method, as recommended by the custom elements spec.

# Why would I want to use the ShadyDom polyfill?

If you want to get access to the shadow DOM APIs (such as `attachShadow()` and `<slot>`) but you don't want the style encapsulation, you can swap your browsers native implementation of shadow DOM for ShadyDOM. Shadydom only polyfills the API but not style encapsulation. For example, if you want to create custom elements that you can still style using a global stylesheet you can force the browser to use Shadydom with:

```
npm i @webcomponents/shadydom
```

then

```
	<script>
		ShadyDOM = {force: true}
	</script>
	<script src="shadydom.min.js"></script>
```

# How to use this patch

Git clone this repo then

```
cd ~/path/to/your-svelte-project
npm install ~/path/to/this/repo
```

To restore your app to regular Svelte:

```
npm install svelte@latest
```