---
title: useInsertionEffect
---

<Pitfall>

`useInsertionEffect` is aimed at CSS-in-JS library authors. Unless you are working on a CSS-in-JS library and need a place to inject the styles, you probably want [`useEffect`](/apis/react/useEffect) or [`useLayoutEffect`](/apis/react/useLayoutEffect) instead.

</Pitfall>

<Intro>

`useInsertionEffect` is a version of [`useEffect`](/apis/react/useEffect) that fires before any DOM mutations.

```js
useInsertionEffect(setup, dependencies?)
```

</Intro>

<InlineToc />

---

## Usage {/*usage*/}

### Injecting dynamic styles from CSS-in-JS libraries {/*injecting-dynamic-styles-from-css-in-js-libraries*/}

Traditionally, you would style React components using plain CSS.

```js
// In your JS file:
<button className="success" />

// In your CSS file:
.success { color: green; }
```

Some teams prefer to author styles directly in JavaScript code instead of writing CSS files. This usually requires using a CSS-in-JS library or a tool. There are three common approaches to CSS-in-JS you might encounter:

1. Static extraction to CSS files with a compiler
2. Inline styles, e.g. `<div style={{ opacity: 1 }}>`
3. Runtime injection of `<style>` tags

If you use CSS-in-JS, we recommend a combination of the first two approaches (CSS files for static styles, inline styles for dynamic styles). **We don't recommend runtime `<style>` tag injection for two reasons:**

1. Runtime injection forces the browser to recalculate the styles a lot more often.
2. Runtime injection can be very slow if it happens at the wrong time in the React lifecycle.

The first problem is not solvable, but `useInsertionEffect` helps you solve the second problem.

Call `useInsertionEffect` to insert the styles before any DOM mutations:

```js {4-11}
// Inside your CSS-in-JS library
let isInserted = new Set();
function useCSS(rule) {
  useInsertionEffect(() => {
  	// As explained earlier, we don't recommend runtime injection of <style> tags.
  	// But if you have to do it, then it's important to do in useInsertionEffect.
    if (!isInserted.has(rule)) {
      isInserted.add(rule);
      document.head.appendChild(getStyleForRule(rule));
    }
  });
  return rule;
}

function Button() {
  const className = useCSS('...');
  return <div className={className} />;
}
```

Similarly to `useEffect`, `useInsertionEffect` does not run on the server. If you need to collect which CSS rules have been used on the server, you can do it during rendering:

```js {1,4-6}
let collectedRulesSet = new Set();

function useCSS(rule) {
  if (typeof window === 'undefined') {
    collectedRulesSet.add(rule);
  }
  useInsertionEffect(() => {
    // ...
  });
  return rule;
}
```

[Read more about upgrading CSS-in-JS libraries with runtime injection to `useInsertionEffect`.](https://github.com/reactwg/react-18/discussions/110)

<DeepDive>

#### How is this better than injecting styles during rendering or useLayoutEffect? {/*how-is-this-better-than-injecting-styles-during-rendering-or-uselayouteffect*/}

If you insert styles during rendering and React is processing a [non-blocking update,](/apis/react/useTransition#marking-a-state-update-as-a-non-blocking-transition) the browser will recalculate the styles every single frame while rendering a component tree, which can be **extremely slow.**

`useInsertionEffect` is better than inserting styles during [`useLayoutEffect`](/apis/react/useLayoutEffect) or [`useEffect`](/apis/react/useEffect) because it ensures that by the time other Effects run in your components, the `<style>` tags have already been inserted. Otherwise, layout calculations in regular Effects would be wrong due to outdated styles.

</DeepDive>

---

## Reference {/*reference*/}

### `useInsertionEffect(setup, dependencies?)` {/*useinsertioneffect*/}

<Pitfall>

`useInsertionEffect` is aimed at CSS-in-JS library authors. Unless you are working on a CSS-in-JS library and need a place to inject the styles, you probably want [`useEffect`](/apis/react/useEffect) or [`useLayoutEffect`](/apis/react/useLayoutEffect) instead.

</Pitfall>

Call `useInsertionEffect` to insert the styles before any DOM mutations:

```js
function useCSS(rule) {
  useInsertionEffect(() => {
  	// As explained earlier, we don't recommend runtime injection of <style> tags.
  	// But if you have to do it, then it's important to do in useInsertionEffect.
  	// ... inject <style> tags here ...
  });
  return rule;
}
```


[See more examples above.](#examples-connecting)

#### Parameters {/*parameters*/}

* `setup`: The function with your Effect's logic. Your setup function may also optionally return a *cleanup* function. Before your component is first added to the DOM, React will run your setup function. After every re-render with changed dependencies, React will first run the cleanup function (if you provided it) with the old values, and then run your setup function with the new values. Before your component is removed from the DOM, React will run your cleanup function one last time.
 
* **optional** `dependencies`: The list of all reactive values referenced inside of the `setup` code. Reactive values include props, state, and all the variables and functions declared directly inside your component body. If your linter is [configured for React](/learn/editor-setup#linting), it will verify that every reactive value is correctly specified as a dependency. The list of dependencies must have a constant number of items and be written inline like `[dep1, dep2, dep3]`. React will compare each dependency with its previous value using the [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) comparison algorithm. If you don't specify the dependencies at all, your Effect will re-run after every re-render of the component.

#### Returns {/*returns*/}

`useInsertionEffect` returns `undefined`.

#### Caveats {/*caveats*/}

* Effects only run on the client. They don't run during server rendering.
* You can't update state from inside `useInsertionEffect`.
* By the time `useInsertionEffect` runs, refs are not attached yet, and DOM is not yet updated.
