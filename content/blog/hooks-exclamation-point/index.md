---
title: 'Hook(s)! Hook(s)! Hook(s)!'
date: '2019-02-10T02:40:27.765Z'
---

![Captain James Hook from Hook](./hook.jpg)

Unless you've been living under a (technology) rock, you are well aware that the React team released the long anticipated Hooks feature last week.

Hooks allow developers to express state management and perform side effects that were heretofore accomplished primarily through class components. The benefits from using hooks are myriad, ranging from the elimination of the dependency on JavaScript classes to more concise and reuseable code.

Whatever the reasons, the community has been buzzing with excitement regarding this new feature since their announcement at [React Conf last year](https://www.youtube.com/watch?v=dpw9EHDh2bM).

While many React devs have been experimenting with hooks since their announcement, I have largely ignored the feature until its release. On the morning of the 16.8 release ([the one with hooks](https://reactjs.org/blog/2019/02/06/react-v16.8.0.html)), I played around with hooks and even introduced them to an app of mine.

And I fell in love.

As a result of my excitement for this new feature, I wanted to write a few posts about my experience with React hooks.

In this post, I will show how I refactored a search form from a class based approach to one that utilizes hooks. In a post to follow, I'll show the reusability of hooks, comparing them with other reuse patterns in React. Finally, I'll have a short post on my overall feelings on hooks, sharing sentiments on how they might change the React community's best practices and looking at difficulties in adopting them.

If you haven't done so, I suggest reading the [React docs on hooks](https://reactjs.org/docs/hooks-intro.html) before continuing with this post. The docs are fantastic and do a good job illustrating the motivation for the feature, their uses and caveats you should be aware of.

## User Search

The use case we'll look at is your typical async search:

- A user can search by user id to render the corresponding user's information (name, email).
- The search is an async call to our server (mocked in this exercise). As such, we want to show a loading indicator when the request is in progress.
- In case of an error, we want to display an error message.

Pretty standard, right? Let's look at how we would implement this feature in React, starting with a class-based approach.

## [Old and Busted](https://www.youtube.com/watch?v=ha-uagjJQ9k&feature=youtu.be&t=20) - Component Approach

The first snippet we'll look at is how we will handle state within our React component. Here's the code:

```jsx
class UserSearch extends React.Component {
  constructor(props) {
    super(props)

    this.state = {
      isLoading: false,
      searchTerm: '',
    }
  }

  handleSearchChange = evt => {
    this.setState({
      searchTerm: evt.currentTarget.value,
      errorMessage: undefined,
    })
  }

  handleSearchSubmit = evt => {
    const { searchTerm } = this.state
    evt.preventDefault()

    this.setState({
      isLoading: true,
      searchResult: undefined,
    })

    getUserInfoById(searchTerm)
      .then(userInfo => {
        this.setState({
          searchResult: userInfo,
          isLoading: false,
        })
      })
      .catch(err => {
        this.setState({
          errorMessage: err.message,
          isLoading: false,
        })
      })
  }

  render() {
    // rendering code
  }
}
```

Those experienced with React will find this old hat:

- The `searchTerm` state will be fully managed, so when a user types into the search box, we'll update component state `searchTerm` on each key stroke. This piece of state will be passed to the input element on each update.
- When the user types, we want to reset the `errorMessage` state, if any.
- When the user submits the form, we'll use the current value of `searchTerm` to call our user service's `getUserById` method.
- Since `getUserById` is an async call, the component will orchestrate the `isLoading`, `errorMessage` and `searchResult` state using Promise's `then` and `catch` API.

That's quite a bit of responsibility, and we haven't even looked at rendering (i.e. the presentation of data in the UI) yet. Let's do that now:

```jsx
class UserSearch extends React.Component {
  // previous shown state management logic

  render() {
    const { searchTerm, isLoading, errorMessage, searchResult } = this.state
    return (
      <div>
        <h2>Basic Search State</h2>
        <div>
          <form onSubmit={this.handleSearchSubmit}>
            <div>
              <label htmlFor="user-id">User ID</label>
              <input
                type="text"
                value={searchTerm}
                onChange={this.handleSearchChange}
              />
              <button
                disabled={searchTerm.length === 0}
                onClick={this.handleSearchSubmit}
              >
                Search
              </button>
            </div>
          </form>
          <div>
            {isLoading ? <div>Loading...</div> : null}
            {errorMessage != null ? (
              <div>
                Error searching for {searchTerm}: {errorMessage}
              </div>
            ) : null}
            {searchResult != null ? (
              <div>
                <h2>User Info</h2>
                <div>
                  {searchResult.firstName} {searchResult.lastName}
                </div>
                <div>{searchResult.email}</div>
              </div>
            ) : null}
          </div>
        </div>
      </div>
    )
  }
}
```

To manage the `searchTerm` state, we have an input element that is bound to the component's `handleSearchChange` property method. In addition, we have our form element's `onSubmit` handler and our button's `onClick` handler bound to the component's `handleSearchSubmit`. As far as displaying information to the end user, we have logic that pivots on the `isLoading`, `errorMessage` and `searchResult` state to render the appropriate content based on the current state.

As previously mentioned, this component is doing a lot of work that's _not_ related to rendering content on the screen. It's handling the orchestration of loading state, error state and data fetching that is very common in applications that accept user input.

If your eyes have glazed over a bit while reading this, I don't blame you.

Well, wake up cuz we're gonna refactor this component to use hooks and revel in the fact that writing component code with inputs might be ~~fun again~~ less bad.

## New Hotness - Using Hooks

First things first, we're going to move the following responsibilities out of the React component and into a hook called `useSearch`:

- Invoking the user service's `getUserInfoById` using the search term provided by the end user
- Orchestration of loading state, error state and search result

In short, all activity related to the state of _fetching user data_ will be delegated to the `useSearch` hook. Here's the code:

```js
import React from 'react'
import { getUserInfoById } from '../user-service'

const useSearch = () => {
  const [searchState, dispatch] = React.useReducer(searchReducer, {})

  function resetErrorState() {
    dispatch({ type: 'Initial' })
  }

  async function search(userId) {
    dispatch({ type: 'Search' })

    try {
      const result = await getUserInfoById(userId)
      dispatch({ type: 'Success', result })
    } catch (e) {
      dispatch({ type: 'Error', error: e.message })
    }
  }

  const { isLoading, searchResult, error } = searchState

  return [isLoading, error, searchResult, search, resetErrorState]
}
const searchReducer = (initialState, action) => {
  switch (action.type) {
    case 'Initial':
      return {}
    case 'Search':
      return {
        isLoading: true,
      }
    case 'Error':
      return {
        error: action.error,
      }
    case 'Success':
      return {
        searchResult: action.result,
      }
    default:
      return initialState
  }
}
```

The interesting bits of this hook are the following:

- `useReducer` - similar to `setState`, `useReducer` allows us to manage non-arbitrary state in a React functional component. Using a similar pattern that we see in Redux, we can express our state as a switch statement (or mutually exclusive manner) that pivots on an action's type and data tied to that action. The result of the `useReducer` expression is the current state and a dispatch function (which is used to update state). In code, this is the `searchState` and `dispatch` variables, respectively.
- `resetErrorState` and `search` - these functions that are declared in the hook function body [close over](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures) the dispatch function. When invoked, they dispatch an action that updates this hook's state.
- Hook return value - both the state and the hook's aforementioned updater functions are then returned as a [tuple](https://en.wikipedia.org/wiki/Tuple). These values can then be consumed by the calling function (i.e. React component) to render or update this hook's state.

In this code I took a couple liberties with `async/await` and [`useReducer`](https://reactjs.org/docs/hooks-reference.html#usereducer) to allow for better async and state management, respectively. One could just as easily use `Promise` syntax and `if/else` constructs to achieve as much.

Now let's see how this hook is used in our React Component:

```jsx
import React from 'react'
import useSearch from './useSearch'

const SearchForm = () => {
  const [userId, updateUserId] = React.useState('')
  const [isLoading, error, searchResult, searchUserId, reset] = useSearch()

  return (
    <div>
      <form
        onSubmit={evt => {
          evt.preventDefault()
          searchUserId(userId)
        }}
      >
        <div>
          <label htmlFor="user-id">User ID</label>
          <input
            type="text"
            value={userId}
            onChange={evt => {
              updateUserId(evt.currentTarget.value)
              reset()
            }}
          />
          <button
            disabled={userId.length === 0}
            onClick={() => searchUserId(userId)}
          >
            Search
          </button>
        </div>
      </form>
      <div>
        {isLoading ? <div>Loading...</div> : null}
        {error != null ? (
          <div>
            Error searching for {userId}: {error}
          </div>
        ) : null}
        {searchResult != null ? (
          <div>
            <h2>User Info</h2>
            <div>
              {searchResult.firstName} {searchResult.lastName}
            </div>
            <div>{searchResult.email}</div>
          </div>
        ) : null}
      </div>
    </div>
  )
}
```

That's a little nicer! We still have nearly identical rendering of presentation logic, however we have removed all logic related to the actual fetching of data. That logic is now encapsulated in the `useSearch` hook, which exposes its state and update functions by simply returning them as a tuple. These values are then used how we would normally handle passed in props.

In addition, the `SearchForm` is now expressed as a React function component, instead of a class component.

Note also that, though we removed the logic around managing search state, this component is still managing it's own bit of state related to the the text input to collect a user id. This is accomplished using the `useState` hook which returns the current state value and an updated function.

After this refactor, this component now has two responsibilities:

- Collect user input
- Render results of searching for a user using said user input

That's it! No boilerplate `handleSearchTerm` class properties that setState on the component. No rote state management code around loading or error state (which is hard to read and an infamous spot for introducing form state bugs and inconsistencies). This component is laser focused in it's purpose.

In addition, there's a little less indirection and jumping around from the render functions input elements and the class component's handler functions; the code reads a little bit better from top to bottom.

While we didn't necessarily reduce the number of lines of code, we effectively encapsulated application logic into cohesive chunks (i.e. functions). An effective refactor!

## Conclusion

My initial impression when doing the above refactor was that I was thinking about state in terms of functions and closures and not in terms of class state, lifecycle methods or React patterns such as higher-order components. When implementing `useSearch`, it felt like I was writing normal JavaScript code without regard for how the function would be used.

I believe this is the main goal of hooks: allow the developer to refactor and (therefore reuse) code how they normally would in JavaScript by pulling common pieces of logic into a function.

While the above example didn't reduce the number of lines of code and the code in the `useSearch` hook will not (yet) be reused, splitting the logic of fetching, expressing loading state, error state etc. from presentation is a worthwhile endeavor.

In the next post in this series, we'll look at how hooks allow for easy reuse of logic by comparing them to existing patterns in the React community.

<!--
## Next Post

Now that we looked at a basic refactoring, moving from using JS class components to using hooks with functions,  -->
