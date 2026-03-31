---
sidebar_position: 6
---

# Lists and Keys

You will often need to display multiple similar components from a collection of data. You can use JavaScript array methods like `filter()` and `map()` to transform arrays of data into arrays of components.

Source: [react.dev/learn/rendering-lists](https://react.dev/learn/rendering-lists)

## Rendering Data from Arrays

Use the `map()` method to transform an array of data into an array of JSX elements:

```js
const people = [
  { id: 1, name: 'Creola Katherine Johnson' },
  { id: 2, name: 'Mario Jose Molina-Pasquel Henriquez' },
  { id: 3, name: 'Mohammad Abdus Salam' },
];

export default function List() {
  const listItems = people.map((person) => (
    <li key={person.id}>{person.name}</li>
  ));

  return <ul>{listItems}</ul>;
}
```

## Filtering Arrays

Use `filter()` to narrow down the data before mapping:

```js
const people = [
  { id: 1, name: 'Creola Katherine Johnson', profession: 'mathematician' },
  { id: 2, name: 'Mario Jose Molina-Pasquel', profession: 'chemist' },
  { id: 3, name: 'Mohammad Abdus Salam', profession: 'physicist' },
  { id: 4, name: 'Percy Lavon Julian', profession: 'chemist' },
];

export default function ChemistList() {
  const chemists = people.filter((person) => person.profession === 'chemist');

  return (
    <ul>
      {chemists.map((person) => (
        <li key={person.id}>{person.name}</li>
      ))}
    </ul>
  );
}
```

## Keeping List Items in Order with key

Each item in a list needs a `key` -- a string or number that uniquely identifies it among its siblings. Keys tell React which array item each component corresponds to so it can match them up between renders.

### Rules for Keys

- **Keys must be unique among siblings.** Different arrays can use the same keys.
- **Keys must not change.** Don't generate them during rendering.

### Where to Get Keys

| Data source | Key |
|---|---|
| Database | Use the database ID (primary key) |
| Locally generated data | Use an incrementing counter or `crypto.randomUUID()` at creation time |

```js
const [todos, setTodos] = useState([]);
let nextId = 0;

function addTodo(text) {
  setTodos([...todos, { id: nextId++, text }]);
}
```

### Why React Needs Keys

Keys let React identify which items changed, were added, or were removed. Without proper keys, React may incorrectly reuse component state when items are reordered, leading to subtle bugs.

### Pitfalls

**Don't use array index as a key** if the list can be reordered, filtered, or have items inserted in the middle. Index keys cause items to "inherit" the wrong state:

```js
// Bad: using index as key
{items.map((item, index) => <ListItem key={index} item={item} />)}

// Good: using a stable, unique ID
{items.map((item) => <ListItem key={item.id} item={item} />)}
```

**Don't generate keys on the fly** with `Math.random()` or `crypto.randomUUID()` during rendering. This creates new keys every render and destroys all component state and DOM:

```js
// Bad: key changes every render
{items.map((item) => <ListItem key={Math.random()} item={item} />)}
```

## Rendering Nested Lists

For nested structures, use `map` at each level:

```js
const sections = [
  {
    id: 'basics',
    title: 'Basics',
    items: [
      { id: 'jsx', name: 'JSX' },
      { id: 'components', name: 'Components' },
    ],
  },
  {
    id: 'advanced',
    title: 'Advanced',
    items: [
      { id: 'hooks', name: 'Hooks' },
      { id: 'context', name: 'Context' },
    ],
  },
];

function Outline() {
  return (
    <div>
      {sections.map((section) => (
        <div key={section.id}>
          <h2>{section.title}</h2>
          <ul>
            {section.items.map((item) => (
              <li key={item.id}>{item.name}</li>
            ))}
          </ul>
        </div>
      ))}
    </div>
  );
}
```

## Extracting List Item Components

When list items become complex, extract them into a separate component. The `key` goes on the component itself, not on the root JSX element inside it:

```js
function RecipeList({ recipes }) {
  return (
    <div>
      {recipes.map((recipe) => (
        <Recipe key={recipe.id} recipe={recipe} />
      ))}
    </div>
  );
}

function Recipe({ recipe }) {
  return (
    <div>
      <h2>{recipe.name}</h2>
      <ul>
        {recipe.ingredients.map((ingredient) => (
          <li key={ingredient}>{ingredient}</li>
        ))}
      </ul>
    </div>
  );
}
```

Source: [react.dev/learn/rendering-lists](https://react.dev/learn/rendering-lists)
