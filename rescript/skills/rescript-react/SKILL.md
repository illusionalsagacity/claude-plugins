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

# Styles

In `@rescript/react` 0.14+, `ReactDOM.Style.make` is removed. `ReactDOM.Style.t` is a record type (aliased to `JsxDOMStyle.t`) with optional fields. Use record literals:

```rescript
// Correct — record literal
let style: ReactDOM.Style.t = {padding: "16px", margin: "4px 0"}

// Wrong — ReactDOM.Style.make does not exist
let style = ReactDOM.Style.make(~padding="16px", ~margin="4px 0", ())
```

# JSX Spread Props

The spread must be the **first** set of props, and only one spread is allowed per element:

```rescript
// Correct — spread first
<div {...spread} style role="button" />

// Wrong — spread after explicit props
<div style role="button" {...spread} />

// Wrong — multiple spreads
<div {...a} {...b} />
```

# Callback Refs

`ReactDOM.Ref.callbackDomRef` expects:

```rescript
Js.nullable<Dom.element> => option<unit => unit>
```

Third-party libraries often provide ref setters with a different signature (e.g., `Nullable.t<Dom.htmlElement> => unit`). Common mismatches:

- `Dom.htmlElement` vs `Dom.element` — `:>` won't work here because the ref receives a supertype (`element`) while the binding expects a subtype (`htmlElement`)
- Return type `unit` vs `option<unit => unit>` (cleanup function)

Bridge with `%identity`:

```rescript
external toCallbackRef: (Nullable.t<Dom.htmlElement> => unit) => ReactDOM.Ref.callbackDomRef = "%identity"
```

# JSX Prop Compatibility

## Spreading bound values as DOM props

When a JS library returns an object of event handlers meant to be spread onto elements (e.g., `{onPointerDown: ..., onKeyDown: ...}`), you can't spread a `Dict.t` directly — JSX spread expects `JsxDOM.domProps`.

Prefer binding these as records with optional fields rather than `Dict.t`, since the set of keys is typically a known subset of React event handlers:

```rescript
// Preferred — record with optional fields
type syntheticListeners = {
  onPointerDown?: JsxEvent.Pointer.t => unit,
  onKeyDown?: JsxEvent.Keyboard.t => unit,
}
```

When the keys are truly dynamic and a record isn't feasible, use `%identity` as an escape hatch:

```rescript
external toJsxDOMProps: option<Dict.t<JsxEvent.Synthetic.t => unit>> => JsxDOM.domProps = "%identity"
```

## ARIA and HTML attribute types

When binding attributes that will be spread onto DOM elements, match the types in `JsxDOM.domProps` rather than using simpler ReScript types:

```rescript
// Wrong — JsxDOM.domProps.ariaPressed is option<[#"true" | #"false" | #mixed]>
type draggableAttributes = {ariaPressed: option<bool>}

// Correct — matches JsxDOM.domProps, no conversion needed when spreading
type draggableAttributes = {ariaPressed: option<[#"true" | #"false" | #mixed]>}
```

Check `JsxDOM.domProps` source for the canonical types of ARIA and HTML attributes.
