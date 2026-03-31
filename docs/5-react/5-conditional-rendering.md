---
sidebar_position: 5
---

# Conditional Rendering

Your components will often need to display different things depending on different conditions. In React, you can conditionally render JSX using JavaScript syntax like `if` statements, `&&`, and `? :` operators.

- How to return different JSX depending on a condition
- How to conditionally include or exclude a piece of JSX
- Common conditional syntax shortcuts you'll encounter in React codebases

## Conditionally returning JSX

Let's say you have a `PackingList` component that renders several `Item` components, which can be marked as packed or not:

```js
function Item({ name, isPacked }) {
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item isPacked={true} name="Space suit" />
        <Item
          isPacked={true}
          name="Helmet with a golden leaf"
        />
        <Item isPacked={false} name="Photo of Tam" />
      </ul>
    </section>
  );
}
```

Notice that some of the `Item` components have their `isPacked` prop set to `true` instead of `false`. You want to add a checkmark (✅) to packed items if `isPacked={true}`.

You can write this as an `if`/`else` statement:

```js
if (isPacked) {
  return <li className="item">{name} ✅</li>;
}
return <li className="item">{name}</li>;
```

If `isPacked` is `true`, this returns a different JSX tree, so some items get a checkmark at the end:

```js
function Item({ name, isPacked }) {
  if (isPacked) {
    return <li className="item">{name} ✅</li>;
  }
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item isPacked={true} name="Space suit" />
        <Item
          isPacked={true}
          name="Helmet with a golden leaf"
        />
        <Item isPacked={false} name="Photo of Tam" />
      </ul>
    </section>
  );
}
```

Try editing what gets returned in either case, and see how the result changes!

### Conditionally returning nothing with `null`

In some situations, you might not want to render anything at all. For example, say you don't want to show packed items at all. In this case, a component must still return something, and you can return `null`:

```js
if (isPacked) {
  return null;
}
return <li className="item">{name}</li>;
```

If `isPacked` is true, the component returns nothing: `null`. Otherwise, it returns JSX to render:

```js
function Item({ name, isPacked }) {
  if (isPacked) {
    return null;
  }
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item isPacked={true} name="Space suit" />
        <Item
          isPacked={true}
          name="Helmet with a golden leaf"
        />
        <Item isPacked={false} name="Photo of Tam" />
      </ul>
    </section>
  );
}
```

In practice, returning `null` from a component is not common because it might surprise a developer trying to render it. More often, you would conditionally include or exclude the component in the parent component's JSX. Here's how to do that!

## Conditionally including JSX

In the previous example, you controlled which JSX tree would be returned by the `Item` component. You might have noticed duplication in the render output:

```js
<li className="item">{name} ✅</li>
```

is very similar to:

```js
<li className="item">{name}</li>
```

Both conditional branches render the same outer `<li>` element. If you want to avoid repeating yourself, you can keep the shared structure and only conditionally insert the part that changes.

### Conditional (ternary) operator (`? :`)

JavaScript has a compact syntax for writing a conditional expression -- the conditional operator or "ternary operator".

Instead of this:

```js
if (isPacked) {
  return <li className="item">{name} ✅</li>;
}
return <li className="item">{name}</li>;
```

You can write this:

```js
return (
  <li className="item">
    {isPacked ? name + " ✅" : name}
  </li>
);
```

You can read it as: "if `isPacked` is true, then (`?`) render `name + ' ✅'`, otherwise (`:`) render `name`."

#### Are these two examples fully equivalent?

Even if you're coming from an object-oriented background, you might think the two styles are subtly different because one might create different "instances". In React, JSX elements are lightweight descriptions and don't hold internal state, so these examples are equivalent. (For the deeper details, see [Preserving and Resetting State](https://react.dev/learn/preserving-and-resetting-state).)

Now say you want to wrap the completed item's text into another HTML tag, like `<del>` to strike it out. You can nest more JSX by adding parentheses:

```js
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {isPacked ? (
        <del>{name + " ✅"}</del>
      ) : (
        name
      )}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item isPacked={true} name="Space suit" />
        <Item
          isPacked={true}
          name="Helmet with a golden leaf"
        />
        <Item isPacked={false} name="Photo of Tam" />
      </ul>
    </section>
  );
}
```

This works well for simple conditions, but use it in moderation. If your components become messy with too much nested conditional markup, consider extracting child components to clean things up.

### Logical AND operator (`&&`)

Another common shortcut is the JavaScript logical AND (`&&`) operator. Inside React components, it often comes up when you want to render some JSX only when the condition is true (and render nothing otherwise).

With `&&`, you can conditionally render the checkmark only if `isPacked` is `true`:

```js
return (
  <li className="item">
    {name} {isPacked && "✅"}
  </li>
);
```

Read it like: "if `isPacked`, then (`&&`) render the checkmark, otherwise render nothing."

Here's it in action:

```js
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {name} {isPacked && "✅"}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item isPacked={true} name="Space suit" />
        <Item
          isPacked={true}
          name="Helmet with a golden leaf"
        />
        <Item isPacked={false} name="Photo of Tam" />
      </ul>
    </section>
  );
}
```

A JavaScript `&&` expression returns the value on the right side if the left side is truthy. But if the condition is false, the whole expression becomes `false`. React treats `false` (like `null` or `undefined`) as a "hole" in the JSX tree and doesn't render anything in its place.

Don't put numbers on the left side of `&&`.

For example, this is a common mistake:

```js
messageCount && New messages
```

If `messageCount` is `0`, it's easy to assume it renders nothing, but it actually renders `0`. To fix it, ensure the left side is a boolean:

```js
messageCount > 0 && New messages
```

### Conditionally assigning JSX to a variable

When shortcuts get in the way of writing plain code, you can use an `if` statement and a variable.

Start by providing a default value:

```js
let itemContent = name;
```

Then reassign it if `isPacked` is `true`:

```js
if (isPacked) {
  itemContent = name + " ✅";
}
```

Finally, embed it inside the returned JSX:

```js
<li className="item">{itemContent}</li>
```

This style is verbose, but it's the most flexible. Here's the full example:

```js
function Item({ name, isPacked }) {
  let itemContent = name;
  if (isPacked) {
    itemContent = name + " ✅";
  }
  return <li className="item">{itemContent}</li>;
}
```

Like before, this can conditionally choose arbitrary JSX too:

```js
function Item({ name, isPacked }) {
  let itemContent = name;
  if (isPacked) {
    itemContent = <del>{name + " ✅"}</del>;
  }
  return <li className="item">{itemContent}</li>;
}
```

If you're not familiar with JavaScript yet, this variety of styles might seem overwhelming at first. But learning them will help you read and write JavaScript -- not just React components.

### Refactor a series of `? :` to `if` and variables

This `Drink` example uses multiple `? :` conditions to show different information depending on `name`. You can refactor it into a single `if` statement by grouping the repeated logic:

```js
function Drink({ name }) {
  let part, caffeine, age;
  if (name === "tea") {
    part = "leaf";
    caffeine = "15-70 mg/cup";
    age = "4,000+ years";
  } else if (name === "coffee") {
    part = "bean";
    caffeine = "80-185 mg/cup";
    age = "1,000+ years";
  }
  return (
    <section>
      <h1>{name}</h1>
      <dl>
        <dt>Part of plant</dt>
        <dd>{part}</dd>
        <dt>Caffeine content</dt>
        <dd>{caffeine}</dd>
        <dt>Age</dt>
        <dd>{age}</dd>
      </dl>
    </section>
  );
}

export default function DrinkList() {
  return (
    <div>
      <Drink name="tea" />
      <Drink name="coffee" />
    </div>
  );
}
```

Another option is to remove conditions altogether by moving information into an object and looking it up based on `name`:

```js
const drinks = {
  tea: { part: "leaf", caffeine: "15-70 mg/cup", age: "4,000+ years" },
  coffee: { part: "bean", caffeine: "80-185 mg/cup", age: "1,000+ years" },
};

function Drink({ name }) {
  const info = drinks[name];
  return (
    <section>
      <h1>{name}</h1>
      <dl>
        <dt>Part of plant</dt>
        <dd>{info.part}</dd>
        <dt>Caffeine content</dt>
        <dd>{info.caffeine}</dd>
        <dt>Age</dt>
        <dd>{info.age}</dd>
      </dl>
    </section>
  );
}
```

#### Show an icon for incomplete items with `? :`

Use the conditional operator `cond ? a : b` to render a ❌ when `isPacked` isn't `true`:

```js
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {name} {isPacked ? "✅" : "❌"}
    </li>
  );
}
```

#### Show the item importance with `&&`

Use `&&` to render an importance label only when `importance` is non-zero. In particular, prefer `importance > 0 && ...` instead of `importance && ...` so that `0` does not get rendered:

```js
function Item({ name, importance }) {
  return (
    <li className="item">
      {name}
      {importance > 0 && " "}
      {importance > 0 && <i>(Importance: {importance})</i>}
    </li>
  );
}
```

---

Source: [react.dev/learn/conditional-rendering](https://react.dev/learn/conditional-rendering)

### Key takeaways

- In React, you control branching logic with JavaScript.
- You can return a JSX expression conditionally with an `if` statement.
- You can conditionally save some JSX to a variable and then include it inside other JSX by using curly braces.
- In JSX, `{cond ? a : b}` means: if `cond`, render `a`, otherwise `b`.
- In JSX, `{cond && a}` means: if `cond`, render `a` (otherwise render nothing).
- Conditional shortcuts are common, but you don't have to use them if you prefer plain `if`.

