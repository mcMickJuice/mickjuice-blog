---
title: 'Hook(s)! Hook(s)! Hook(s)!'
date: '2019-02-10T02:40:27.765Z'
---

![Captain James Hook from Hook](./hook.jpg)

<!-- Quick intro to hooks -->

Unless you've been living under a (technology) rock, you are well aware that the React team released the long anticipated Hooks feature last week.

Hooks allow developers to express state management, effect side...effects and perform other tasks that were heretofore accomplished primarily through class components. With hooks, these tasks can be performed using React functional components (aka javascript functions that return React elements). The reasons for these are myriad, ranging from the elimination of requirement on JavaScript classes to more concise code to more reuseable code.

Whatever the reasons, the community has been buzzing with excitement regarding this new feature since their announcement at React Conf (TODO: SOURCE?) late last year.

While many React devs have been experimenting with hooks since their announcement, I have largely ignored the feature until its release.

On the morning of the 16.8 release (the one with hooks), I played around with hooks and even introduced them to an app of mine.

I can't overstate how happy I am with React hooks... (TODO: update this). As a result of my excitement, I wanted to write a few posts about my experience with hooks.

In this post, I will show how I refactored a search form a pure "class based" approach to using hooks. In a post to follow, I'll show the reusability of hooks, comparing them with other reuse patterns in React. Finally, I'll have a short post on my overall feelings on hooks, sharing sentiments on how they might change the React community's best practices and looking at difficulties in adopting them.

If you haven't done so, I suggest reading the React docs on Hooks (TODO: link) before continuing with this post. The docs are fantastic and do a good job illustrating the motivation for the feature, their uses and caveats.

## User Search

Our feature we'll look at is your typical async search:

- Search by user id to render that user's information (name, email)
- The search is an async call to our server (mocked in this exercise). As such, we want to show a loading indicator when the request is in progress
- In case of an error, render the error message

Pretty standard, right? Let's look at how we would implement this feature in React.

## [Old and Busted](https://www.youtube.com/watch?v=ha-uagjJQ9k&feature=youtu.be&t=20) - Class Component Approach

The first snippet we'll look at is how we handle state within a React Component. Here's the code:

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

- `searchTerm` state will be fully managed, so when a user types into the search box, we'll update component state `searchTerm` on each key stroke
- when the user types, we wanna reset the `errorMessage` state, if any.
- when the user submits the form, we'll use the current value of `searchTerm` to call our user service's `getUserById` method
- since `getUserById` is an async call, the component will orchestrate the `isLoading`, `errorMessage` and `searchResult` state.

That's quite a bit of responsibility, and we haven't even looked at rendering (presentation of data) yet. Let's do that now:

```jsx
class UserSearch extends React.Component {
  // previous shown component logic

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

Not much is surprising here, as we have a input element that is "attached" to the component's `handleSearchChange` callback. In addition, we have our form element's `onSubmit` handler and our button's `onClick` handler bound to the component's `handleSearchSubmit`. As far as displaying information to the end user, we have logic that pivots on the `isLoading`, `errorMessage` and `searchResult` bits of logic.

For the full code of this component, see here TODO: link to github

As previously mentioned, this component is doing a lot of work that's _not_ related to rendering content on the screen. It's handling the orchestration of loading state, error state and data fetching that is very common in applications that accept user input. If your eyes have glazed over a bit while reading this, I don't blame you.

Well, wake up cuz we're gonna refactor this component to use hooks and revel in the fact that writing component code with inputs might be fun again. TODO: better way to write this?

## New Hotness - Using Hooks

First things first, we're going to move the following responsibilities out of the React component and into a hook called `useSearch`:

- invoking `getUserInfoById` using the search term provided by the end user
- orchestration of loading state, error state and search result

So, in short, all activity related to the state of _fetching user data_ will be delegated to the `useSearch` hook. Here's the code:

```js
import { useReducer } from 'react'
import { getUserInfoById } from '../user-service'

const useSearch = () => {
  const [searchState, dispatch] = useReducer(searchReducer, {})

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

export default useSearch
```

In this code I took a couple liberties with `async/await` and `useReducer` TODO: link to useReducer to allow for better async management and state management, respectively. One could just as easily use `Promise` syntax and `if/else` constructs to achieve as much.

TODO: expand on this hook, talking specifically about the functions declared and how state and functions are returned from the function

Now let's see how this hook is used in our React Component:

```jsx
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

That's nicer! We still have nearly identical rendering of presentation logic, however we have removed all logic related to the actual fetching of data.

But we are still managing the state of the search term that the user is entering...which is absolutely correct in this case. TODO: focus on `useState`

This component now has two responsibilites:

- collect user input
- render results of searching for a user using said user input

That's it! No boilerplate `handleSearchTerm` class properties that setState on the component. No rote state management around loading or error state (which is hard to read and an infamous spot for introducing form state bugs and inconsistencies). This component is laser focused on it's purpose.

In addition, there's a little less indirection and jumping around from the render functions input elements and the class component's handler functions; the code reads a little bit better from top to bottom.

While we didn't necessarily reduce the number of lines of code, we effectively encapsulated application logic into cohesive chunks (i.e. functions). An effective refactor!

## Conclusion

Even if you are expressing logic in your app that will not be reused, splitting the logic of fetching, expressing loading state, error state etc from presentation is worthwhile.

## Next Post

Now that we looked at a basic refactoring, moving from using JS class components to using hooks with functions, the next post in this series will look at how hooks allow for easy reuse of logic, comparing them to existing patterns in the React community.
