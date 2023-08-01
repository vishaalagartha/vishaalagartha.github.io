---
title: "React Redux Basics"
date: 2023-05-29
permalink: /notes/2023/05/29/react-redux
tags:
--- 

# Some definitions

React - a JavaScript frontend library that makes building UIs easier

Redux - a JavaScript libary that makes state management easier

These two work together via another library called `react-redux` to make
state management easier in an application.

# Overall Flow

React triggers action -> reducer updates store -> React updates state according to store

# Directory Structure
```
/src
  /actions
  /pages
  /components
  /reducers
```

# Actions
Actions trigger changes in state. They are triggered by React components via a call to `dispatch()`

Typically, we have 2 files `action-types.js` and the main `index.js` in the `/actions` directory. 

`action-types.js` just creates variables assigned to strings according to the action name

```
// action-types.js
export const ACTION = 'ACTION'
```

`index.js` contains action creators, which create an action with a potential payload
```
import {ACTION} from './action-types.js'

export const action = (payload) => {
  return {
    type: ACTION,
    payload: payload
  }
}
```

# Reducers
Reducers update the state when an action occurs. They can be combined using `combineReducer(reducers)`

In a reducer, we check the action's type and modify state based on the type. If it doesn't satisfy any of
the types, we return the previous state. It is important that we don't modify the original state
to make redux more functional.

It is important for reducers to have an initial state, so that react has appropriate props.


```
import {ACTION} from '../actions/action-types'

const initialState = {
  stuff: []
}

export default actionReducer = (state=initialState, action) => {
  switch(action.type){
    case ACTION:
      return {
         ...state,
         stuff: [..state.stuff, payload]
      }
      break
    default:
      return state

  }
}
```

# Connect
`connect` makes the magic happen. It allows react and redux to work in sync by telling redux to update when react triggers an action and react to update when redux changes state. How does it achieve this?

## `mapStateToProps(state)`
This function takes the redux store's state and update's the react component's props. It's a simple mapping.

```
const mapStateToProps = state => {
  return {
    stuff: state.stuff
  }
}
```

## `mapDispatchToProps(dispatch)`
This function takes the action creators and dispatches them to the reducers.

```
import {action} from '../actions'
const mapDispatchToProps = dispatch => {
  return {
    action: (payload) => dispatch(action(payload)
  }
}
```

Finally, we create a HOC using `connect(mapStateToProps, mapDispatchToProps)`

```
export default connect(mapStateToProps, mapDispatchToProps)(Component)`
```

# Some sidenotes and boilerplate code

Some necessary npm packages include: react, ReactDOM, redux, react-redux

In order to connect the App to redux, we need to provide the store to it. We do this by wrapping our app in 
a `<Provider>` tag.

Additionally, we need to create the store using `redux`.`

```
import {Provider} from 'react-redux'
import {createStore} from 'redux'
import reducers from './reducers'

const store = createStore(reducers)

const AppWrapper = () => {
  return (
    <Provider store={store}>
      <App/>
    <Provider>
  )



}
export default AppWrapper
```
