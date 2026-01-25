---
name: rescript-react
description: ReScript and React conventions and patterns. Use when writing ReScript and React in projects.
---

# Local State

If a component has several `useState` state atoms, combine them into a `useReducer`.

# Reducer Initialization

For reducers that need initial data, prefer `React.useReducerWithMapState` over `useReducer` + `useEffect0`:

```rescript
// Prefer this (lazy initialization):
let (state, dispatch) = React.useReducerWithMapState(
  reducer,
  (),
  () => {loadingState: Loading, error: None}
)

// Over this (effect-based):
let (state, dispatch) = React.useReducer(reducer, {loadingState: Idle, error: None})
React.useEffect0(() => {
  dispatch(Load)
  None
})
```

Note: For async initialization (e.g., localStorage), the effect approach may be necessary, but consider whether the initial state should be `Loading` rather than `Idle`.
