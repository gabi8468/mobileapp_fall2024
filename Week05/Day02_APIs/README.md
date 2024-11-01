## Data Persistence with JSON Server

We will still have our todos as a piece of state in `App.js`. But now the App can make initial immediate request to server to populate initial state.

Our child components will make their own requests to CREATE EDIT and DELETE

## Rest API Server Setup

Next up we will:

1- create API

2- make initial request

3- create edit delete requests to server, THEN update local data

First we need a new node package in our project. We will be using JSON Server

JSON Server

- (Open source project for learning and dev).
- Pretty close to a production API interaction.
- Stores all info inside of a plain text file.

We will be running 2 different commands in 2 different tab of terminal for the rest of this project. Remember the first tab is for the usual dev server. Let’s open up a second tab and make sure you cd into the current project directory.

```bash
npm install json-server@0
```

_Note: You will notive this time we have requested a specific version of a node package. This is due to the fact that a recent V1 was released in a somewhat broken state. Rememeber, if the big number bumps in a project, that means breaking changes. Just a reminder to pay attention to these things and pin versions if neccessary in your code and your documentation._

Create db.json file inside the root of your project (this is where the data will be stored).

Add in and save:

```js
{
“todos”: []
}
```

Open up `package.json`
add the command

```js
"server": "json-server -p 3001 --watch db.json"
```

to run the command ( second tab)

```bash
npm run server
```

you will get a resources and a home server address
you should see something like

```
Resources
http://localhost:3001/todos

Home
http://localhost:3001
```

#### Troubleshooting and Running JSON Server

Starting our server and testing out routes may surface some errors for some of you.

1. If you get the ECONNREFUSED error

restart json server (command C to kill server at any time)

update your scripts in package.json to:

`"server": "json-server —port 3001 --watch db.json --host 127.0.0.1",`

2. If you get a TypeError [ERR_INVALID_ARG_TYPE]

you probably need to run your server as admin

`sudo npm run server `

You may also need to move the db.json file into a folder named server/db.json

update your scripts in package.json to:

`"server": "json-server --no-cors --port 3001 --watch server/db.json --host 127.0.0.1",`

Don’t forget we need to run two commands to start our project, one for the dev server and one for the api server; 3000 and 3001. For todays class, our api calls will go to localhost 3001 (this would be swapped out by a real server in a production project) but the general workflow is basically identical aside from this fake serverness.

NOTE:

- you can change the port at any time in the script if your port 3001 is in use by something else.
- `db.json` knows we want to store a list named todos because we added that starter code in db.json

## Adding/Testing Route with Rest Client

REST Client for today is a stand alone API client built in to vs code. To add/enable it go to the extensions tab and search for REST Client. Make sure it is enabled.

Now we can test out some requests and responses inside of VS Code. We will need 4 total for today

CREATE:  
POST :3001/todos
ID will be created automatically
response returned with both title and id! (just to let us know it was successful)

FETCH:
`GET :3001/todos`
(This is to grab the initial todos from our DB)
response will be all todos

CREATE:
`POST :3001/todos`
response will be the newly created todo

EDIT:
`PUT :3001/todos/{id}`
response will be the edited object

DELETE
`DELETE :3001/books/{id}`
response will be the deleted object

---

Create a file in the root of your project called `api.http` and add:

This file can be used to test new routes and stick around as documentation for future you or others

NOTE: A button appears about each request. click to send a request and see the response.

```
GET http://localhost:3001/todos HTTP/1.1
Content-Type: application/json

###
POST http://localhost:3001/todos HTTP/1.1
Content-Type: application/json

{
  "title": "Make the bed"
}

###
PUT http://localhost:3001/todos/1 HTTP/1.1
Content-Type: application/json

{
  "title": "Walk the dog"
}

###
DELETE http://localhost:3001/todos/1 HTTP/1.1
Content-Type: application/json
```

NOTE: IDs here are hardcoded as 1. Once we test our post request we will have an entry in our db.json file with the ID of 1. We need to make sure we have at least one TODO before we test EDIT or DELETE.

## Updating our App.js

We will need to make an initial GET request to our server to populate state, but first, let’s wire up our CREATE aka POSTtodo method in react. Remember the TODO object will be returned as the response when successful.

First, we are making API calls again, so we need to install Axios in our project.
nom install axios

Find the create function inside our App.js
Remember we no longer need a random ID, the DB is going to take care of that for us

```js
import axios from ‘axios’
```

```jsx
const createTodo = async (title) => {
  const response = await axios.post('http://localhost:3001/todos', {
    title: title,
  })

  console.log(response)
  // const updatedTodos = [
  //   ...todos,
  //   {id: Math.round(Math.random() * 100000), title: title},
  // ]
  // setTodos(updatedTodos)
  // console.log(todos)
}
```

We can check in networks tab or db.json file at any time. Cool. now what to do with it? Our old code is useful here, but with one little update.

```jsx
const createTodo = async (title) => {
  const response = await axios.post('http://localhost:3001/todos', {
    title: title,
  })

  // console.log(response)
  const updatedTodos = [...todos, response.data]
  setTodos(updatedTodos)
  console.log(todos)
}
```

Ok but if we refresh, we are still left with an empty todo. lets go back an make a request whenever the application starts up!

Lets declare a new function called fetchBooks right after we declare our todos state in App.js
We already know we are going to have to wait for our response so let’s label it async right off the bat.

```jsx
const fetchTodos = async () => {}
```

This function is even more simple since it is a GET request that returns our list in response.data. Don;t forget to set it with our setTodos function at the end

```jsx
const fetchTodos = async () => {
  const response = await axios.get('http://localhost:3001/todos')

  setTodos(response.data)
}
```

But when do we call/invoke this function? We know we don’t want directly call fetchTodos() because it will immediately initiate an app to re-render in an infinite loop of calling fetchTodos, re-rendering and immediately calling it again! How do we call it just one time on load? Our old friend, `useEffect()`. This difference this time is this time we will leave the second argument empty `[]`.

Remember useEffect is always called when a component initially re-renders and sometimes called for others (like when the pieces of state we are watching update). The reason we are leaving the second argument [] blank here is we ONLY want it to fire when App.js initially renders aka we aren’t watching for any changes in state.

Don’t forget to import useEffect at the top before continuing.

```jsx
useEffect(() => {
  fetchTodos()
}, [])
```

OK only UPDATE and DELETE to go! Remember, we need to pass in an ID in our request for these.

First, let’s edit. These should be a breeze because of our API file we wrote as both a test to make sure the api routes work, but also as documentation for ourselves in the future, or other developers that might be working on this project.

Since we have to pass in the ID to our URL at the very end, we need use a template string .

- wrap our route in back ticks
- pass in the ID of the appropriate TODO

```jsx
const response = await axios.put(`http://localhost:3001/todos/${id}`, {
  title: newTitle,
})
```

Food for thought, what if multiple users are editing a todo/hitting our API at the same time?
Remember we get the entire updated record as a response. Currently are not making use of it:

```jsx
const editTodoById = async (id, newTitle) => {
  const response = await axios.put('http://localhost:3001/todos', {
    title: newTitle,
  })

  if (todo.id === id) {
    return {...todo, title: newTitle}
  }

  setTodos(response.data)
}
```

So let’s change that. We are now updating the current todo with the api response. This also comes in handy if we had more than one field to update at once in this TODO object.

```jsx
    const updatedTodos = todos.map((todo) => {
      //if the current todo matches the ID given, we want to edit it
      if (todo.id === id) {
        return {...todo, ...response.data}
      }
      // otherwise just return the other todos as they are
      return todo
    })
    setTodos(updatedTodos)

}
```

Updating our delete function is even simpler! We simple add a DELETE request to our server before updating state.

```jsx
await axios.delete(`http://localhost:3001/todos/${id}`)
```

## An Alternative to Props Alone: Context

Things to remember as we proceed:

- Context is not a replacement for props!
- Context is not a replacement for centralized state stores like Redux.
- Context by itself is a communication channel

Before we do a considerable refactor to use Context inside of our app. Let’s do a small simple exercise to understand Context and what it is doing for us.

1. Create the context.

- `createContext` has 2 properties: Provider and Consumer (more on this in later classes). The Provider is the component used to specify what data we want to share across our application.
  `<Context.Provider>`

2. Specify the data to be shared.
   The `value` prop here is super special, whatever is provided here will be shared with the rest of the application. can be a number string, array with a list or an object with a bunch of stuff including functions.
   Anything wrapped in the provider can access the values shared in context

3. “Consume” the data in a component
   `useContext` is the function for accessing value in `context`

Let’s write some code and see the steps above in action

Create folder called `context/` inside src

- create a new file `todos.js`
- import, create and export

```jsx
import {createContext} from 'react'
// use create context to create our context for our todos
const TodosContext = createContext()

//export it
export default TodosContext
```

Open index.js and wrap the app component with the context provider

```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'
import TodosContext from './context/todos'

const root = ReactDOM.createRoot(document.getElementById('root'))
root.render(
  <TodosContext.Provider value={5}>
    <App />
  </TodosContext.Provider>
)
```

Try and get access in TodoList using useContext

```jsx
import {useContext} from 'react'
import TodosContext from '../context/todos'
import TodoItem from './TodoItem'
export default function TodoList(props) {
  const {todos, onDelete, onEdit} = props
  const num = useContext(TodosContext)
  const renderedTodos = todos.map((todo) => {
    return (
      <TodoItem key={todo.id} todo={todo} onDelete={onDelete} onEdit={onEdit} />
    )
  })

  return (
    <div>
      {num}
      {renderedTodos}
    </div>
  )
}
```

Now we have the value shared in context, but it is static aka won’t change
We want it to change over time
and we want to content on the screen to update accordingly… STATE like count setCount, how do we pass this in to our context provider? Something like:

`value={{count,setCount}}`

Here’s the confusing part: another provider. same word, but it does different things.

The second provider wraps the first. We can do that in our todos.js

```jsx
import {createContext, useState} from 'react'

// use create conext to create our contect for our todos
const TodosContext = createContext()

function Provider({children}) {
  const [count, setCount] = useState(5)

  const valuesToShare = {
    count: count,
    incrementCount: () => {
      setCount(count + 1)
    },
  }

  return (
    <TodosContext.Provider value={valuesToShare}>
      {children}
    </TodosContext.Provider>
  )
}

// named export
export {Provider}
//export as default
export default TodosContext

// import TodosContext, {Provider} from './content/....'
```

Now in `index.js`

```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'
import {Provider} from './context/todos'

const root = ReactDOM.createRoot(document.getElementById('root'))
root.render(
  <Provider>
    <App />
  </Provider>
)
```

NOTE: the `App` component shows up as the prop `children` inside Provider!

Now inside `TodoList` we need to access multiple properties using destructuring. Lets also add a button to call our incrementCount function

```jsx
import {useContext} from 'react'
import TodosContext from '../context/todos'
import TodoItem from './TodoItem'

export default function TodoList(props) {
  const {todos, onDelete, onEdit} = props
  const {count, incrementCount} = useContext(TodosContext)
  const renderedTodos = todos.map((todo) => {
    return (
      <TodoItem key={todo.id} todo={todo} onDelete={onDelete} onEdit={onEdit} />
    )
  })

  return (
    <div>
      {count}
      <button onClick={incrementCount}>Add to Count</button>
      {renderedTodos}
    </div>
  )
}
```

What’s happening: Provider has some state, every time our count updates Provider and everything wrapped in it (app, etc) updates!

Before we update our Provider with everything App.js was handling we need to think about Application State vs Component State for example `todos` vs `showEdit`. Many components consume our list of todos, only our individual `TodoItem`s care about `showEdit`. What about to titles inside the forms? those are probably component state too right?

_Application State_

- Todos
- Fetch
- Create
- Edit
- Delete

_Component State_

- Title(s) bound to input elements
- showEdit

## In Class Exercise: Refactor!

Let’s do a big refractor and move all of the TODOs as well as Fetch, Create, Edit, Delete functions to our context provider and consume them differently inside each component.

Final `src/` folder will me posted inside of this weeks notes at the end of class.

Together we will:

- move everything from `App` state to context provider, leave our useEffect function
- update value from empty object to share all the things!
- reach up to context to grab the fetchTodos function
- peep our child components and remove the props

Then

- update all children components to consume the proper functions and values from context instead of props. There will be some instances where we use both props and context!
