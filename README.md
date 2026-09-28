# Todo App with React Context API

This project is a small Todo application built with React, Vite, and Tailwind CSS. It is also a practical example of the React Context API.

The application allows the user to:

- Add a todo
- Edit a todo
- Mark a todo as completed
- Delete a todo
- Keep todos after a page refresh with `localStorage`

The main learning idea is simple: `App.jsx` owns the todo state and the functions that change it, while Context makes that state and those functions available to child components without passing them through every component as props.

## What This Project Teaches

This project demonstrates:

1. `useState` for component state
2. `useEffect` for synchronizing with browser storage
3. `createContext` and `useContext` for shared values
4. A custom hook named `useTodo`
5. Controlled form inputs
6. Rendering arrays with `map`
7. Immutable state updates with `map` and `filter`
8. React event handling
9. Tailwind CSS utility classes
10. Vite's React development and production workflow

## Running the Project

### Requirements

Install Node.js and npm before starting. npm is normally included with Node.js.

### Install dependencies

Run this command from the project directory:

```bash
npm install
```

This installs the packages listed in `package.json`, including React, React DOM, Vite, Tailwind CSS, and the Vite plugins.

### Start the development server

```bash
npm run dev
```

Vite prints a local URL, usually similar to:

```text
http://localhost:5173
```

Open that URL in a browser. Vite watches the source files and updates the page after changes.

### Build the application

```bash
npm run build
```

This creates an optimized production version in the `dist` directory.

### Preview the production build

```bash
npm run preview
```

### Run the linter

```bash
npm run lint
```

The linter checks the JavaScript and JSX files for common mistakes and code-quality problems.

## Project Structure

```text
10todoContext/
|-- index.html
|-- package.json
|-- vite.config.js
|-- public/
|-- src/
|   |-- App.jsx
|   |-- index.css
|   |-- main.jsx
|   |-- components/
|   |   |-- TodoForm.jsx
|   |   `-- TodoItem.jsx
|   `-- contexts/
|       `-- TodoContext.js
`-- README.md
```

### `index.html`

This is the HTML document that Vite serves. The most important element is:

```html
<div id="root"></div>
```

React finds this element and renders the application inside it. The module script loads `src/main.jsx`, which is the React entry point.

### `src/main.jsx`

```jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'
import App from './App.jsx'

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
)
```

This file:

1. Imports React's development helper, `StrictMode`.
2. Imports `createRoot`, which creates the React rendering root.
3. Imports the global CSS file.
4. Imports the top-level `App` component.
5. Renders `App` inside the element with the id `root`.

`StrictMode` does not create a visible element. It helps React identify problematic patterns during development.

### `src/index.css`

```css
@import "tailwindcss";
```

This imports Tailwind CSS v4. Because `main.jsx` imports `index.css`, Tailwind utility classes are available in the JSX files.

### `vite.config.js`

```js
import react from '@vitejs/plugin-react'
import { defineConfig } from 'vite'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [
    react(),
    tailwindcss(),
  ],
})
```

The two plugins have different jobs:

- `react()` enables React and JSX support.
- `tailwindcss()` connects Tailwind CSS v4 to Vite.

### `package.json`

The important scripts are:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "lint": "eslint .",
    "preview": "vite preview"
  }
}
```

The project uses React 19, Vite, Tailwind CSS v4, and ESLint.

## Why the Context API Is Used

Suppose `App` owns the todos and `TodoForm` needs the `addTodo` function. Without Context, the function would need to be passed through every component between them:

```text
App
  -> Layout
    -> Main
      -> TodoForm
```

`Layout` and `Main` might not use `addTodo`, but they would still have to receive and forward it. This is called prop drilling.

Context creates a shared channel instead:

```text
App provides todos and actions
          |
          v
TodoProvider
          |
          +--> TodoForm reads addTodo
          +--> TodoItem reads updateTodo, deleteTodo, toggleComplete
```

Context is useful here because several components need the same todo data or actions. It is not necessary to put every value in Context. Small values used by only one component can remain local state.

## `src/contexts/TodoContext.js`

This file defines the Context API used by the todo components.

### Imports

```js
import { createContext, useContext } from "react";
```

- `createContext` creates the context object.
- `useContext` reads the value provided by the nearest provider.

### The context shape

```js
export const TodoContext = createContext({
  todos: [
    {
      id: 1,
      todo: "todo msg",
      completed: false,
    },
  ],
  addTodo: (todo) => {},
  updateTodo: (id, todo) => {},
  deleteTodo: (id) => {},
  toggleComplete: (id) => {},
})
```

The object describes the values that todo components can access:

- `todos` is the list of todo objects.
- `addTodo` adds a new item.
- `updateTodo` changes an existing item.
- `deleteTodo` removes an item.
- `toggleComplete` changes the completed state.

The functions in this default object are placeholders. `App.jsx` supplies the real functions through `TodoProvider`. The default object documents the expected context shape.

### The `useTodo` custom hook

```js
export const useTodo = () => {
  return useContext(TodoContext)
}
```

This wrapper means that components can write:

```js
const { addTodo } = useTodo()
```

Instead of repeating this in every component:

```js
const { addTodo } = useContext(TodoContext)
```

The hook gives the project one consistent way to access todo context values.

### The provider

```js
export const TodoProvider = TodoContext.Provider
```

`TodoProvider` is the component that places a value into Context. `App.jsx` uses it like this:

```jsx
<TodoProvider value={{ todos, addTodo, updateTodo, deleteTodo, toggleComplete }}>
  ...children...
</TodoProvider>
```

Every component inside this provider can read the value with `useTodo()`.

## `src/App.jsx`

`App.jsx` is the source of truth for the application's todo state. It creates the state, defines the actions, connects them to `TodoProvider`, and renders the page.

### Imports

```jsx
import { useState, useEffect } from 'react'
import { TodoProvider } from './contexts/TodoContext'
import TodoForm from './components/TodoForm'
import TodoItem from './components/TodoItem'
```

The components are imported without curly braces because both `TodoForm.jsx` and `TodoItem.jsx` use default exports.

### Todo state

```jsx
const [todos, setTodos] = useState([])
```

`todos` contains the current array. `setTodos` replaces that array and causes React to render the new result.

The initial value is an empty array. Existing data is loaded from `localStorage` by the first effect described below.

### Adding a todo

```jsx
const addTodo = (todo) => {
  setTodos((prev) => [{ id: Date.now(), ...todo }, ...prev])
}
```

The form sends an object such as:

```js
{
  todo: "Learn Context API",
  completed: false
}
```

`addTodo` creates a new object, adds an id based on the current timestamp, and places the new item at the beginning of the array.

The updater form is important:

```js
setTodos((prev) => nextTodos)
```

It calculates the next value from the latest previous value, which is safer when updates happen close together.

### Updating a todo

```jsx
const updateTodo = (id, todo) => {
  setTodos((prev) =>
    prev.map((prevTodo) =>
      prevTodo.id === id ? todo : prevTodo
    )
  )
}
```

`map` creates a new array. Only the todo with the matching id is replaced. Every other todo remains unchanged.

`TodoItem` preserves the id while editing:

```jsx
updateTodo(todo.id, { ...todo, todo: todoMsg })
```

The spread syntax copies the old object and changes only its text.

### Deleting a todo

```jsx
const deleteTodo = (id) => {
  setTodos((prev) => prev.filter((todo) => todo.id !== id))
}
```

`filter` creates a new array containing every todo except the item whose id matches the requested id.

### Toggling completion

```jsx
const toggleComplete = (id) => {
  setTodos((prev) =>
    prev.map((prevTodo) =>
      prevTodo.id === id
        ? { ...prevTodo, completed: !prevTodo.completed }
        : prevTodo
    )
  )
}
```

The matching item is copied and its `completed` boolean is inverted. `false` becomes `true`, and `true` becomes `false`.

### Loading from localStorage

```jsx
useEffect(() => {
  const todos = JSON.parse(localStorage.getItem("todos"))

  if (todos && todos.length > 0) {
    setTodos(todos)
  }
}, [])
```

The empty dependency array means this effect runs after the component mounts.

The browser stores strings, not arrays. Therefore:

1. `localStorage.getItem("todos")` reads the stored string.
2. `JSON.parse` converts the string into a JavaScript array.
3. `setTodos` places the restored array into React state.

If there is no saved todo or the saved array is empty, the initial empty state remains.

### Saving to localStorage

```jsx
useEffect(() => {
  localStorage.setItem("todos", JSON.stringify(todos))
}, [todos])
```

This effect runs whenever `todos` changes. `JSON.stringify` converts the array into a string that the browser can store.

Together, the effects create this flow:

```text
Page opens
  -> read localStorage
  -> restore saved todos

User changes a todo
  -> React state changes
  -> save the new array to localStorage
```

### Providing the shared value

```jsx
<TodoProvider value={{ todos, addTodo, updateTodo, deleteTodo, toggleComplete }}>
  ...
</TodoProvider>
```

This is the most important Context API line in the application. The provider makes the current array and all actions available to the form and todo items.

### Rendering todo items

```jsx
{todos.map((todo) => (
  <div key={todo.id} className="w-full">
    <TodoItem todo={todo} />
  </div>
))}
```

`map` converts each data object into a `TodoItem` component. The `key` gives React a stable identity for each item. The todo itself is passed as a prop because each item needs its own data, while shared actions come from Context.

## `src/components/TodoForm.jsx`

`TodoForm` collects text from the user and creates a new todo.

### Local input state

```jsx
const [todo, setTodo] = useState("")
```

The `todo` value contains the input text. This is a controlled input because React owns its value:

```jsx
value={todo}
onChange={(e) => setTodo(e.target.value)}
```

Every keystroke updates React state, and the input displays that state.

### Reading `addTodo` from Context

```jsx
const { addTodo } = useTodo()
```

The form does not receive `addTodo` through props. It reads it from the nearest `TodoProvider`.

### Submit handler

```jsx
const add = (e) => {
  e.preventDefault()
  if (!todo) return
  addTodo({ todo, completed: false })
  setTodo("")
}
```

The sequence is:

1. `preventDefault` stops the browser from reloading the page.
2. The empty check stops an empty todo from being added.
3. `addTodo` sends the text and initial completion state to `App` through Context.
4. `setTodo("")` clears the input after a successful submission.

Using a real form also means the user can click Add or press Enter.

## `src/components/TodoItem.jsx`

`TodoItem` displays one todo and gives the user controls for completion, editing, and deletion.

### Props and Context

```jsx
function TodoItem({ todo }) {
```

The todo data comes from `App` through a prop. The shared actions come from Context:

```jsx
const { updateTodo, deleteTodo, toggleComplete } = useTodo()
```

This is a useful combination: the item receives its own data as a prop but uses Context for application-wide actions.

### Edit state

```jsx
const [isTodoEditable, setIsTodoEditable] = useState(false)
const [todoMsg, setTodoMsg] = useState(todo.todo)
```

`isTodoEditable` controls whether the text input is read-only. `todoMsg` stores the text while the user is editing.

The input uses:

```jsx
readOnly={!isTodoEditable}
```

When edit mode is false, the input is read-only. When edit mode is true, the user can type.

### Saving an edit

```jsx
const editTodo = () => {
  updateTodo(todo.id, { ...todo, todo: todoMsg })
  setIsTodoEditable(false)
}
```

The old todo is copied, the text is replaced, and the complete object is sent to `App`. The id is preserved so `App` can find the correct item.

### Completion checkbox

```jsx
<input
  type="checkbox"
  checked={todo.completed}
  onChange={toggleCompleted}
/>
```

The checkbox is controlled by `todo.completed`. Clicking it calls:

```jsx
const toggleCompleted = () => {
  toggleComplete(todo.id)
}
```

The actual state update remains in `App`, which keeps one source of truth.

### Edit and delete buttons

The edit button has two modes:

- When the item is not editable, it turns edit mode on.
- When the item is already editable, it saves the changed text.

Completed items cannot be edited because the button is disabled:

```jsx
disabled={todo.completed}
```

The delete button calls:

```jsx
onClick={() => deleteTodo(todo.id)}
```

Only the id is needed. `App` uses it to remove the matching object with `filter`.

### Conditional styling

The item background changes based on completion:

```jsx
todo.completed ? "bg-[#c6e9a7]" : "bg-[#ccbed7]"
```

The text receives a line-through style when complete:

```jsx
todo.completed ? "line-through" : ""
```

These visual changes let the user understand the state immediately.

## Complete Data Flow

### Adding

```text
User types in TodoForm
  -> TodoForm stores the text locally
  -> User submits the form
  -> TodoForm reads addTodo with useTodo()
  -> App creates an id and a new todo object
  -> App updates todos with setTodos
  -> TodoProvider receives the new value
  -> A new TodoItem is rendered
  -> localStorage is updated
```

### Editing

```text
User clicks edit
  -> TodoItem enables its input
  -> User changes todoMsg
  -> User clicks save
  -> TodoItem calls updateTodo(id, updatedTodo)
  -> App replaces the matching item with map
  -> The updated text is rendered
  -> localStorage is updated
```

### Completing

```text
User clicks the checkbox
  -> TodoItem calls toggleComplete(id)
  -> App maps through the array
  -> The matching completed value is inverted
  -> Checkbox, color, and text styling update
  -> localStorage is updated
```

### Deleting

```text
User clicks delete
  -> TodoItem calls deleteTodo(id)
  -> App filters out the matching item
  -> The item disappears
  -> localStorage is updated
```

## Todo Data Shape

Every todo is expected to look like this:

```js
{
  id: 1710000000000,
  todo: "Learn React Context",
  completed: false
}
```

The fields mean:

- `id`: identifies the todo for React, editing, and deletion.
- `todo`: contains the user's text.
- `completed`: controls the checkbox and completed styling.

The text field is named `todo` in this project. If it is renamed to `text`, every producer and consumer must be updated consistently.

## Why `map` and `filter` Are Used

React state should be treated as immutable. The project creates new arrays instead of changing the old array directly:

```js
prev.map(...)
prev.filter(...)
```

Avoid direct mutation such as:

```js
todos.push(newTodo)
todos[0].completed = true
```

Direct mutation can prevent React from recognizing the update correctly and can lead to stale or confusing UI behavior.

## Tailwind Classes Used in the Interface

The styling is written directly in JSX. Examples include:

- `min-h-screen`: makes the page at least as tall as the viewport.
- `max-w-2xl`: limits the content width.
- `mx-auto`: centers the content.
- `flex`: creates a flexbox layout.
- `gap-y-3`: adds vertical spacing between wrapped items.
- `rounded-lg`: rounds corners.
- `bg-green-600`: styles the Add button.
- `line-through`: crosses out completed text.
- `disabled:opacity-50`: fades disabled controls.

The page does not need a separate component stylesheet because the layout and visual rules are expressed with Tailwind utility classes.

## Common Mistakes

### Default and named exports

The components use default exports:

```js
export default TodoForm
export default TodoItem
```

Therefore, import them without curly braces:

```js
import TodoForm from './components/TodoForm'
import TodoItem from './components/TodoItem'
```

Curly braces are used for named exports such as:

```js
import { useTodo, TodoProvider } from './contexts/TodoContext'
```

### Misspelling the custom hook

The context exports `useTodo`. `useTode` is a different name and will cause an import or reference error.

### Forgetting the provider

`useTodo()` must be used by a component below `TodoProvider`. Otherwise, the component will not receive the real state and actions from `App`.

### Removing the todo id during editing

Always preserve the id:

```js
{ ...todo, todo: todoMsg }
```

The id is how update and delete operations identify the correct item.

### Forgetting `preventDefault`

Form submission without `event.preventDefault()` can reload the page and interrupt the React interaction.

### Using the wrong localStorage conversion

Save arrays with `JSON.stringify` and read them with `JSON.parse`:

```js
localStorage.setItem("todos", JSON.stringify(todos))
const todos = JSON.parse(localStorage.getItem("todos"))
```

## Possible Improvements

The current project is intentionally small. Good next improvements would be:

- Move state and action functions into a dedicated provider component.
- Reject whitespace-only input with `todo.trim()`.
- Add an empty-state message.
- Add filters for all, active, and completed todos.
- Confirm before deleting.
- Handle malformed `localStorage` data with `try/catch`.
- Add tests for adding, editing, deleting, toggling, and persistence.
- Add accessible labels to the icon-only buttons.
- Use a stronger unique-id strategy for a larger application.

## Final Architecture

```text
main.jsx
  -> App.jsx
       owns todos and action functions
       |
       v
     TodoProvider
       shares todos and actions
       |
       +--> TodoForm.jsx
       |      reads addTodo
       |
       +--> TodoItem.jsx
              reads updateTodo, deleteTodo, toggleComplete
```

The project follows one clear rule: `App.jsx` owns the data, `TodoContext.js` exposes the shared access point, and child components use that access point to perform their work. This is why Context is helpful in this application: it removes unnecessary prop drilling while keeping the actual state-changing logic in one place.
