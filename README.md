# ECMAScript Pattern Matching

## [Status](https://tc39.github.io/process-document/)

**Stage**: 1

**Author**: Kat Marchán (npm, [@maybekatz](https://twitter.com/maybekatz))

**Champions**: Brian Terlson (Microsoft, [@bterlson](https://twitter.com/bterlson)), Sebastian Markbåge (Facebook, [@sebmarkbage](https://twitter.com/sebmarkbage)), Kat Marchán (npm, [@maybekatz](https://twitter.com/maybekatz))

## Introduction

This proposal adds a pattern matching expression to the language, based on the
existing [Destructuring Binding
Patterns](https://tc39.github.io/ecma262/#sec-destructuring-binding-patterns).

There's many proposals potentially related to this one, and other proposals
might mention interaction with this. This file includes casual, example-based
discussion of the proposal, and there's also a document [describing the core
semantics in more formal language](CORE.md), which will be iterated over into
the final Spec-ese.

There's also a document including suggestions for [other future
proposals](TO_INFINITY_AND_BEYOND.md), which are dependent on this one, but do
not directly affect the main behavior of the feature.

This proposal was approved for Stage 1 in the May 2018 TC39 meeting, and [slides
for that presentation are
available](https://docs.google.com/presentation/d/1WPyAO4pHRsfwGoiIZupz_-tzAdv8mirw-aZfbxbAVcQ/edit?usp=sharing).

This proposal draws heavily from corresponding features in
[Rust](https://doc.rust-lang.org/1.6.0/book/patterns.html),
[F#](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/pattern-matching),
[Scala](http://www.scala-lang.org/files/archive/spec/2.11/08-pattern-matching.html),
and
[Elixir/Erlang](https://elixir-lang.org/getting-started/pattern-matching.html).

## Motivating Examples

Matching `fetch()` responses:

```javascript
const res = await fetch(jsonService)
case (res) {
  let {status: 200, headers: {'Content-Length': s}} ->
    console.log(`size is ${s}`),
  let {status: 404} -> {
    console.log('JSON not found'),
  let {status} if (status >= 400) -> {
    throw new RequestError(res)
  },
}
```

Terser, more functional handling of Redux reducers. Compare with [this same
example in the Redux
documentation](https://redux.js.org/basics/reducers#splitting-reducers):

```js
function todoApp (state = initialState, action) {
  return case (action) {
    let {type: 'set-visibility-filter', filter: visFilter} ->
      ({...state, visFilter}),
    let {type: 'add-todo', text} ->
      ({...state, todos: [...state.todos, {text}]}),
    let {type: 'toggle-todo', index} -> (
      {
        ...state,
        todos: state.todos.map((todo, idx) => idx === index
          ? {...todo, done: !todo.done}
          : todo
        )
      }
    )
    let _ -> state // ignore unknown actions
  }
}
```

Or mixed in with JSX code for quick props handling, using the expression
version:

```js
<Fetch url={API_URL}>{
  props => case (props) {
    let {loading} -> <Loading />
    let {error} -> <Error error={error} />
    let {data} -> <Page data={data} />
  }
}
</Fetch>
```
(via [Divjot Singh](https://twitter.com/bogas04/status/977499729557839873))

General structural duck-typing on an API for vector-likes.

```js
const getLength = vector => case (vector) {
  const { x, y, z } -> Math.sqrt(x ** 2 + y ** 2 + z ** 2)
  const { x, y } -> Math.sqrt(x ** 2 + y ** 2)
  const [...etc] -> vector.length
}
getLength({x: 1, y: 2, z: 3}) // 3.74165
```

## Implementations

* [Babel Plugin](https://github.com/babel/babel/pull/9318)
* [Sweet.js macro](https://github.com/natefaubion/sparkler) (NOTE: this isn't based on the proposal, this proposal is partially based on it!)
