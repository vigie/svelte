---
title: Template syntax
---


### Tags

---

A lowercase tag, like `<div>`, denotes a regular HTML element. A capitalised tag, such as `<Widget>`, indicates a *component*.

```html
<script>
	import Widget from './Widget.svelte';
</script>

<div>
	<Widget/>
</div>
```


### Attributes

---

By default, attributes work exactly like their HTML counterparts.

```html
<div class="foo">
	<button disabled>can't touch this</button>
</div>
```

---

As in HTML, values may be unquoted.

```html
<input type=checkbox>
```

---

Attribute values can contain JavaScript expressions.

```html
<a href="page/{p}">page {p}</a>
```

---

Or they can *be* JavaScript expressions.

```html
<button disabled={!clickable}>...</button>
```

---

An expression might include characters that would cause syntax highlighting to fail in regular HTML, in which case quoting the value is permitted. The quotes do not affect how the value is parsed:

```html
<button disabled="{number !== 42}">...</button>
```

---

When the attribute name and value match (`name={name}`), they can be replaced with `{name}`.

```html
<!-- These are equivalent -->
<button disabled={disabled}>...</button>
<button {disabled}>...</button>
```

---

*Spread attributes* allow many attributes or properties to be passed to an element or component at once.

An element or component can have multiple spread attributes, interspersed with regular ones.

```html
<Widget {...things}/>
```


### Text expressions

* `{expression}`

---

Text can also contain JavaScript expressions:

```html
<h1>Hello {name}!</h1>
<p>{a} + {b} = {a + b}.</p>
```


### HTML expressions

* `{@html expression}`

---

In a text expression, characters like `<` and `>` are escaped. With HTML expressions, they're not.

> Svelte does not sanitize expressions before injecting HTML. If the data comes from an untrusted source, you must sanitize it, or you are exposing your users to an XSS vulnerability.

```html
<div class="blog-post">
	<h1>{post.title}</h1>
	{@html post.content}
</div>
```


### If blocks

* `{#if expression}...{/if}`
* `{#if expression}...{:else if expression}...{/if}`
* `{#if expression}...{:else}...{/if}`

---

Content that is conditionally rendered can be wrapped in an if block.

```html
{#if answer === 42}
	<p>what was the question?</p>
{/if}
```

---

Additional conditions can be added with `{:else if expression}`, optionally ending in an `{:else}` clause.

```html
{#if porridge.temperature > 100}
	<p>too hot!</p>
{:else if 80 > porridge.temperature}
	<p>too cold!</p>
{:else}
	<p>just right!</p>
{/if}
```


### Each blocks

* `{#each expression as name}...{/each}`
* `{#each expression as name, index}...{/each}`
* `{#each expression as name, index (key)}...{/each}`
* `{#each expression as name}...{:else}...{/each}`

---

Iterating over lists of values can be done with an each block.

```html
<h1>Shopping list</h1>
<ul>
	{#each items as item}
		<li>{item.name} x {item.qty}</li>
	{/each}
</ul>
```

---

An each block can also specify an *index*, equivalent to the second argument in an `array.map(...)` callback:

```html
{#each items as item, i}
	<li>{i + 1}: {item.name} x {item.qty}</li>
{/each}
```

---

If a *key* expression is provided, Svelte will diff the list when data changes, rather than adding or removing items at the end.

```html
{#each items as item, i (item.id)}
	<li>{i + 1}: {item.name} x {item.qty}</li>
{/each}
```

---

You can freely use destructuring patterns in each blocks.

```html
{#each items as { id, name, qty }, i (id)}
	<li>{i + 1}: {name} x {qty}</li>
{/each}
```

---

An each block can also have an `{:else}` clause, which is rendered if the list is empty.

```html
{#each todos as todo}
	<p>{todo.text}</p>
{:else}
	<p>No tasks today!</p>
{/each}
```


### Await blocks

* `{#await expression}...{:then name}...{:catch name}...{/await}`
* `{#await expression}...{:then name}...{/await}`
* `{#await expression then name}...{/await}`

---

Await blocks allow you to branch on the three possible states of a Promise — pending, fulfilled or rejected.

```html
{#await promise}
	<!-- promise is pending -->
	<p>waiting for the promise to resolve...</p>
{:then value}
	<!-- promise was fulfilled -->
	<p>The value is {value}</p>
{:catch error}
	<!-- promise was rejected -->
	<p>Something went wrong: {error.message}</p>
{/await}
```

---

The `catch` block can be omitted if no error is possible, and the initial block can be omitted if you don't care about the pending state.

```html
{#await promise then value}
	<p>The value is {value}</p>
{/await}
```


### DOM events

* `on:eventname={handler}`
* `on:eventname|modifiers={handler}`

---

Use the `on:` directive to listen to DOM events.

```html
<script>
	let count = 0;

	function handleClick(event) {
		count += 1;
	}
</script>

<button on:click={handleClick}>
	count: {count}
</button>
```

---

Handlers can be declared inline with no performance penalty. As with attributes, directive values may be quoted for the sake of syntax highlighters.

```html
<button on:click="{() => count += 1}">
	count: {count}
</button>
```

---

Add *modifiers* to DOM events with the `|` character.

The following modifiers are available:

* `preventDefault` — calls `event.preventDefault()` before running the handler
* `stopPropagation` — calls `event.stopPropagation()`, preventing the event reaching the next element
* `passive` — improves scrolling performance on touch/wheel events (Svelte will add it automatically where it's safe to do so)
* `capture` — fires the handler during the *capture* phase instead of the *bubbling* phase
* `once` — remove the handler after the first time it runs

Modifiers can be chained together, e.g. `on:click|once|capture={...}`.

```html
<form on:submit|preventDefault={handleSubmit}>
	<!-- the `submit` event's default is prevented,
	     so the page won't reload -->
</form>
```

---

If the `on:` directive is used without a value, the component will *forward* the event, meaning that a consumer of the component can listen for it.

```html
<button on:click>
	The component itself will emit the click event
</button>
```


### Component events

* `on:eventname={handler}`

---

Components can emit events using [createEventDispatcher](#docs/createEventDispatcher), or by forwarding DOM events. Listening for component events looks the same as listening for DOM events:

```html
<SomeComponent on:whatever={handler}/>
```



### Element bindings

* `bind:property={value}`

---

Data ordinarily flows down, from parent to child. The `bind:` directive allows data to flow the other way, from child to parent. Most bindings are specific to particular elements.

The simplest bindings reflect the value of a property, such as `input.value`.

```html
<input bind:value={name}>
<textarea bind:value={text}></textarea>

<input type="checkbox" bind:checked={yes}>
```

---

If the name matches the value, you can use a shorthand.

```html
<!-- These are equivalent -->
<input bind:value={value}>
<input bind:value>
```

---

Numeric input values are coerced; even though `input.value` is a string as far as the DOM is concerned, Svelte will treat it as a number.

```html
<input type="number" bind:value={num}>
<input type="range" bind:value={num}>
```

---

Inputs that work together can use `bind:group`.

```html
<script>
	let tortilla = 'Plain';
	let fillings = [];
</script>

<!-- grouped radio inputs are mutually exclusive -->
<input type="radio" bind:group={tortilla} value="Plain">
<input type="radio" bind:group={tortilla} value="Whole wheat">
<input type="radio" bind:group={tortilla} value="Spinach">

<!-- grouped checkbox inputs populate an array -->
<input type="checkbox" bind:group={fillings} value="Rice">
<input type="checkbox" bind:group={fillings} value="Beans">
<input type="checkbox" bind:group={fillings} value="Cheese">
<input type="checkbox" bind:group={fillings} value="Guac (extra)">
```

---

A `<select>` value binding corresponds to the `value` property on the selected `<option>`, which can be any value (not just strings, as is normally the case in the DOM).

```html
<select bind:value={selected}>
	<option value={a}>a</option>
	<option value={b}>b</option>
	<option value={c}>c</option>
</select>
```

---

A `<select multiple>` element behaves similarly to a checkbox group.

```html
<select multiple bind:value={fillings}>
	<option value="Rice">Rice</option>
	<option value="Beans">Beans</option>
	<option value="Cheese">Cheese</option>
	<option value="Guac (extra)">Guac (extra)</option>
</select>
```

---

When the value of an `<option>` matches its text content, the attribute can be omitted.

```html
<select multiple bind:value={fillings}>
	<option>Rice</option>
	<option>Beans</option>
	<option>Cheese</option>
	<option>Guac (extra)</option>
</select>
```

---

Media elements (`<audio>` and `<video>`) have their own set of bindings — four *readonly* ones...

* `duration` (readonly) — the total duration of the video, in seconds
* `buffered` (readonly) — an array of `{start, end}` objects
* `seekable` (readonly) — ditto
* `played` (readonly) — ditto

...and three *two-way* bindings:

* `currentTime` — the current point in the video, in seconds
* `paused` — this one should be self-explanatory
* `volume` — a value between 0 and 1

```html
<video
	src={clip}
	bind:duration
	bind:buffered
	bind:seekable
	bind:played
	bind:currentTime
	bind:paused
	bind:volume
></video>
```

---

Block-level elements have readonly `clientWidth`, `clientHeight`, `offsetWidth` and `offsetHeight` bindings, measured using a technique similar to [this one](http://www.backalleycoder.com/2013/03/18/cross-browser-event-based-element-resize-detection/).

```html
<div bind:offsetWidth={width} bind:offsetHeight={height}>
	<Chart {width} {height}/>
</div>
```

---

To get a reference to a DOM node, use `bind:this`.

```html
<script>
	import { onMount } from 'svelte';

	let canvasElement;

	onMount(() => {
		const ctx = canvasElement.getContext('2d');
		drawStuff(ctx);
	});
</script>

<canvas bind:this={canvasElement}></canvas>
```


### Component bindings

* `bind:property={value}`

You can bind to component props using the same mechanism.

```html
<Keypad bind:value={pin}/>
```

---

Components also support `bind:this`, allowing you to interact with component instances programmatically.

(Note that we can do `{cart.empty}` rather than `{() => cart.empty()}`, since there is no `this` inside a component's `<script>` block.)

```html
<ShoppingCart bind:this={cart}/>

<button on:click={cart.empty}>
	Empty shopping cart
</button>
```


### Classes

* `class:name={value}`
* `class:name`

---

A `class:` directive provides a shorter way of toggling a class on an element.

```html
<!-- These are equivalent -->
<div class="{active ? 'active' : ''}">...</div>
<div class:active={active}>...</div>

<!-- Shorthand, for when name and value match -->
<div class:active>...</div>
```


### Actions

* `use:action`
* `use:action={parameters}`

```js
action = (node: HTMLElement, parameters: any) => {
	update?: (parameters: any) => void,
	destroy: () => void
}
```

---

Actions are functions that are called when an element is created. They must return an object with a `destroy` method that is called after the element is unmounted:

```html
<script>
	function foo(node) {
		// the node has been mounted in the DOM

		return {
			destroy() {
				// the node has been removed from the DOM
			}
		};
	}
</script>

<div use:foo></div>
```

---

An action can have parameters. If the returned value has an `update` method, it will be called whenever those parameters change, immediately after Svelte has applied updates to the markup.

> Don't worry about the fact that we're redeclaring the `foo` function for every component instance — Svelte will hoist any functions that don't depend on local state out of the component definition.

```html
<script>
	export let bar;

	function foo(node, bar) {
		// the node has been mounted in the DOM

		return {
			update(bar) {
				// the value of `bar` has changed
			},

			destroy() {
				// the node has been removed from the DOM
			}
		};
	}
</script>

<div use:foo={bar}></div>
```



### TODO

* transitions
* animations
* slots
* special elements