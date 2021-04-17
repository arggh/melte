# zodern:melte

Build [cybernetically enhanced web apps](https://svelte.dev) with Meteor and Svelte.

Based on [meteor-svelte](https://github.com/meteor-svelte/meteor-svelte/pull/30) with these added features:

- Tracker statements
- Support for hot module replacement (HMR) to update modified components without requiring a full page reload. Requires your app to use Meteor 2 and the [hot-module-replacement](https://docs.meteor.com/packages/hot-module-replacement.html) package.
- Handles syntax errors without crashing
- Supports Typescript preprocessing for script blocks

Compatible with Meteor 1.8.2 and newer.

## Installation

To use `meteor-svelte`, run the following commands:

```bash
meteor add zodern:melte
meteor npm install svelte
```

The `svelte` npm package uses newer JS syntax. To fix any errors with the cordova or legacy client, update your package.json file to tell Meteor to recompile it:

```js
{
  ...

  "meteor": {
    ...

    "nodeModules": {
      "recompile": {
        "svelte": ["legacy"]
      }
    }
  }
}
```

### Tracker Statements

In addition to the [$ reactive statements](https://svelte.dev/docs#3_$_marks_a_statement_as_reactive) Svelte normally supports, this adds `$m` tracker statements. They behave the same as normal reactive statements, but also rerun whenever any Tracker dependencies they use are invalidated. Behind the scenes, it uses [`Tracker.autorun`](https://docs.meteor.com/api/tracker.html#Tracker-autorun).

Example:

```html
<script>
const Todos = new Mongo.Collection('todos');

let sortDirection = 1;
let subReady = false;

// Tracker will unsubscribe when the Svelte component is destroyed
$m: {
  let sub = Meteor.subscribe('todos');
  subReady = sub.ready();
}

// This will rerun whenever the documents are updated or sortDirection is changed
$m: todos = Todos.find({}, { sort: { createdAt: sortDirection}}).fetch()

// You can still use normal reactive statements
$: completedCount = todos.filter(todo => todo.completed).length;
</script>

Sort by:
<select bind:value="{sortDirection}">
  <option value="-1">Oldest First</option>
  <option value="1">Newest First</option>
</select>

{#if !subReady}
  <p>Loading...</p>
{/if}

<ul>
  {#each todos as todo}
    <li>{todo.name} - {todo.createdAt}</li>
  {/each}
</ul>

<p>{completedCount} completed</p>
```

## Typescript

To use typescript with svelte files:

1) Add the typescript npm package to your app with `meteor npm i --save-dev typescript`
2) Add a `tsconfig.json` file. An example file can be found in the [Meteor docs](https://guide.meteor.com/build-tool.html#typescript)
3) In your svelte files, set the `lang="ts"` attribute for scripts:

```html
<script lang="ts">
  let count = 0;

  let doubled: number;
  $: doubled = count * 2;
</script>

<button on:click="{() => count += 1}">Click me</button>
<p>You've clicked the button {count} times (doubled: {doubled}).</p>
```

Melte does not check the types when compiling Svelte files. To check types, you can use [`svelte-check`](https://www.npmjs.com/package/svelte-check) or [`eslint-plugin-svelte3`](https://github.com/sveltejs/eslint-plugin-svelte3).

Learn more about using Typescript and Svelte from [Svelte's docs](https://github.com/sveltejs/language-tools/blob/master/docs/preprocessors/typescript.md).

If you encounter the error `The "path" argument must be of type string. Received undefined`, this is usually from Typescript not being able to find the `tsconfig.json` file.

## Options

Compiler options can be specified with a `"svelte:compiler"` property in `package.json`. For example:

```json
{
  ...
  "svelte:compiler": {
    "extensions": ["svelte", "html"],
    "hydratable": true,
    "css": false
  }
}
```

**`extensions` (default: `["svelte"]`)**

An array of file extensions to be recognized by the package.
Note that HTML files are not compiled with the Svelte compiler if they contain top-level `<head>` or `<body>` elements.
Instead, the contents of the elements are added to the respective sections in the HTML output generated by Meteor (similar to what the `static-html` package does).

**`hydratable` (default: `false`)**

By default, Svelte removes server-rendered static HTML when the application is loaded on the client and replaces it with a client-rendered version.
If you want to reuse (hydrate) server-rendered HTML, set the `hydratable` option to `true` (which generates additional code for client components) and [use the `hydrate` option when instantiating your root component](https://svelte.dev/docs#Creating_a_component).

**`css` (default: `true`)**

Svelte can [extract styles for server-side rendering](https://svelte.dev/docs#Server-side_component_API).
If you want to render CSS on the server, you might want to set the `css` option to `false` so that client-rendered components don't insert CSS into the DOM.

## Server-Side Rendering

`meteor-svelte` supports server-side rendering with minimal configuration.
If you import Svelte components on the server, they are automatically built for server-side rendering.
See the [Svelte API docs](https://svelte.dev/docs#Server-side_component_API), the [example app](https://github.com/meteor-svelte/ssr-example), and the `hydratable` and `css` options above for more details.

## Examples

* [Apollo integration](https://github.com/meteor-svelte/apollo-example)
* [Tracker integration](https://github.com/meteor-svelte/tracker-example)
* [Server-side rendering](https://github.com/meteor-svelte/ssr-example)
