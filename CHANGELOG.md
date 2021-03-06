# Changelog

## 8.1.0 (Not released)

### Noticeable changes

- Stores no longer return `this` from `register()` by default. **This is
  a potentially breaking change**, however should not pose a problem to
  projects using idiomatic Store registration.

### Internal Changes

- Added plugin class to manage defaults
- `tag` now includes the name of the function in `toString()`

## 8.0.0

### Noticeable changes

- Stores now contain the logic for how it should receive an action.
  logic is contained under `send`.
- Stores now contain the logic to determine what method should resolve
  an action sent to it. This is defined in `register`
- `Microcosm::deserialize` will now only operate on the keys provided
  by the seed object. This means that data passed into `replace`
  will only blow way keys provided in the data object.
- The signaling logic for dispatching actions will throw an error if
  the action provided is not a function
- Internalized tag, it will now lazy evaluate as actions are fired
- Upgraded Foliage, Microcosm now contains `subscribe`, `unsubscribe`, and `publish` aliases for `listen`, `ignore`, and `publish`

### Breaking Changes

- Remove all uses of the `tag` module.

#### Changes to Stores

Before this release, stores would listen to actions using the
stringified value of their functions:

```javascript
var MyStore = {
  [Action.add](state, params){}
}
```

This was terse, however required actions to be tagged with a special
helper method. It also required any module that needed access to a
Store's method to also know what actions it implemented.

To address these concerns, Stores now communicate with a Microcosm
using the `register` method:

```javascript
var MyStore = {
  register() {
    return {
      [Action.add]: this.add
    }
  },
  add(state, params){}
}
```

Under the hood, Microcosm tags functions automatically.

## 7.1.1

- Bumped Foliage to a newer version

## 7.1.0

### Noticeable changes

- `Microcosm::start` will return itself

### Internal improvements

- Replaced all uses of ES6 modules with CommonJS. This was causing
  issues in non-ES6 module projects.
- Microcosm publishes as separate modules now. Ideally, this will make
  internal pieces easier to reuse and help with debugging.

## 7.0.0

- Internally, Microcosm now uses
  [Foliage](https://github.com/vigetlabs/foliage) for state
  management.
- `pull` is now `get`, as it is inherited from Foliage
- Microcosm is actually an extension of Foliage, so it now includes
  all Foliage methods.
- Microcosm no longer uses toString() to get the key for Stores. This
  was decided upon so that it is easier to reason about what a Store
  is responsible for when hooking it into a Microcosm.

## 6.2.1

- Externalize some methods to fix extension

## 6.2.0

- Microcosm's event system has been replaced with
  [Diode](https://github.com/vigetlabs/diode). The APIs are the
  same. This should not lead to any breaking changes.

## 6.1.0

- `Microcosm::pull` can now accept an array of keys for the first
  argument. This will traverse the nested keys of state to calculate value.

## 6.0.0

6.0.0 is the second effort to reduce the surface area of the Microcosm API.

- Removed `Upstream` and `Downstream` mixins. They used the
  undocumented context API and introduced some complexity in testing
- `Microcosm::send` is now `Microcosm::push`
- `Microcosm::push` is now `Microcosm::replace`
- `Microcosm::dispatch` and `Microcosm::commit` are now private. These
  are important methods that should not be overridden

## 5.2.0

- `Microcosm::pull` accepts a callback that allows you to modify the
result. This should help to make data queries more terse.
- Removed `Microcosm::clone`, the functionality is not gone, but it
  has been internalized to mitigate the cost of future changes
- Removed mixins from main payload to improve size

## 5.1.1

- Fix build process mistake :-/

## 5.1.0

- Removed fallback from `Microcosm::pull` which returns all state
- Added an `Upstream` and `Downstream` mixin, however it is
  experimental. More details will come as the feature develops.
- `Microcosm::send` will throw an error if given an undefined
  action parameter

## 5.0.0

Version 5 represents an attempt to address some growth pains from
rapidly adding new features to Microcosm. Names have been changed to
improve consistency and internal APIs have been refactored. The
overall surface area of the app has been reduced and more opinions have
been made.

- Renamed `Microcosm::seed` to `Microcosm::push`
- Renamed `Microcosm::get` to `Microcosm::pull`
- Removed `Microcosm::has`
- Removed `Microcosm::getInitialState`. the `Store` API still provides
  this function, however it is the expectation of the system that
  value of state is a primitive object. This is so that Microcosm
  always knows how to smartly clone its state, regardless of if
  another data library is used for its values.
- Removed `Microcosm::swap`, this was an internal API that is no
  longer required
- Renamed `Microcosm::reset` to `Microcosm::commit`
- Removed `Microcosm::shouldUpdate`. If no stores respond to an
  action, a change event will not fire anyway. Placing this concern in
  the view layer keeps React's `shouldComponentUpdate` as the single
  responsibility for this task.
- Added `Microcosm::toObject`
- Internal function `mapBy` has been renamed to `remap`. It now
  operates primarily upon objects.
- `Microcosm::pump` is now `Microcosm::emit`, this is to better match
  existing event emitter libraries (including the one in Node's
  standard library)

As an additional illustration, the Microcosm API has been logistically
sorted within `./cheatsheet.md`

## 4.0.0

- Added concept of plugins. Plugins provide a way to layer on
  additional functionality. This has specifically been added so that
  environment specific behavior may be added to an app.
- Added `Microcosm::start`. Calling `start()` will bootstrap initial
  state, run all plugins, then execute a callback.

## 3.3.0

- `mapBy` internal function now accepts an initial value
- Changed `Microcosm::dispatch` copy strategy. Instead of merging a
  change set, it now directly modifies a clone of the previous
  state.
- Added `Microcosm::clone`. This method defines how state is copied
  before dispatching an action.

## 3.2.0

- Changed default shouldUpdate algorithm

## 3.1.0

- `Microcosm::getInitialState()` now accepts an `options`
  argument. This argument is passed down from the constructor.

## 3.0.0

- Changed data update pattern to more closely match
  [Om](https://github.com/omcljs/om/wiki/Basic-Tutorial). This means
  that `Microcosm::merge` has been replaced with
  `Microcosm::swap`. Additionally, `Microcosm::reset` has been added
  to completely obliterate old state.
- `Microcosm::addStore` now only accepts one store at a time. It was
  not being utilized, gives poorer error handling, and makes let less
  clear the order in which Stores will process data.
- The internal class `Heartbeat` was replaced with `pulse`. Pulse is a
  function that can act as a factory or decorator. When given an
  argument, it extends an object with emitter functionality, otherwise
  it returns a new object that implements the same API. This
  eliminates the possibility that the private `_callbacks` member of
  `Heartbeat` was overridden. It also reduces the use of classical
  inheritance, which yields some minor file size benefits by
  polyfilling less of the `class` API.

## 2.0.1

- Fix issue where empty arguments would break deserialize

## 2.0.0

- Replace default `Microcosm::send` currying with partial application
  using `Microcosm::prepare`
- Throw an error if a store is added that does not have a unique identifier
- `Microcosm::set` has been replaced with `Microcosm::merge`, so far
  `set` has only been used internally to `Microcosm` and `merge` dries
  a couple of things up

### More info on removing currying

Currying has been removed `Microcosm::send`. This was overly clever
and somewhat malicious. If an action has default arguments, JavaScript
has no way (to my knowledge) of communicating it. One (me) could get into a
situation where it is unclear why an action has not fired properly
(insufficient arguments when expecting fallback defaults).

In a language without static typing, this can be particularly hard to debug.

In most cases, partial application is sufficient. In light of this,
actions can be "buffered up" up with `Microcosm::prepare`:

```javascript
// Old
let curried = app.send(Action)

// New
let partial = app.prepare(Action)
```

`Microcosm::prepare` is basically just `fn.bind()` under the
hood. Actions should not use context whatsoever, so this should be a
reasonable caveat.

## 1.4.0

- `Store.deserialize` returns the result of `getInitialState` if no
  state is given
- Added `Microcosm.swap` to perform diffing and emission on change
- `Microcosm.seed` will now trigger a change event
- `Heartbeat.js` now invokes callbacks with `callback.call(this)`

## 1.3.0

- Microcosms will `set` the result of `getInitialState` when adding a store
- Microcosms will execute `deserialize` on stores when running `seed`
- Adding a store will now fold its properties on top of a default set
  of options. See `./src/Store.js` for details.

## 1.2.1

- Fix bug introduced with Tag by exposing ES6 module

## 1.2.0

- All stores can implement a `serialize` method which allows them to
  shape how app state is serialized to JSON.

## 1.1.0

- Better seeding. Added `Microcosm::seed` which accepts an
object. For each known key, Microcosm will the associated store's
`getInitialState` function and set the returned value.
- Exposed `Microcosm::getInitialState` to configure the starting value
  of the instance. This is useful for those wishing to use the
  `immutable` npm package by Facebook.
- Microcosm will not emit changes on dispatch unless the new state
  fails a shallow equality check. This can be configured with
  `Microcosm::shouldUpdate`
- `Microcosm::send` is now curried.

## 1.0.0

This version adds many breaking changes to better support other
libraries such as
[Colonel Kurtz](https://github.com/vigetlabs/colonel-kurtz) and [Ars
Arsenal](https://github.com/vigetlabs/ars-arsenal).

In summary, these changes are an effort to alleviate the cumbersome
nature of managing unique instances of Actions and Stores for each
Microcosm instance. 1.0.0 moves away from this, instead relying on
pure functions which an individual instance uses to operate upon a
global state object.

- Actions must now be tagged with `microcosm/tag`. For the time being,
  this is to provide a unique identifier to each Action. It would be
  nice in future versions to figure out a way to utilize `WeakMap`.
- Stores are plain objects, no longer inheriting from `Store` base
  class.
- Stores must implement a `toString` method which returns a unique id.
- State for a store must now be accessed with: `microcosm.get(Store)`
- Microcosms no longer require `addActions`, actions are fired with
  `microcosm.send(Action, params)`
- Removed `Microscope` container component. Just use `listen`

## 0.2.0

- Remove `get all()` from `Store`. This is to reduce namespace collisions. Stores should define their own getters.

## 0.1.0

- Added a `pump` method to `Microcosm` instances. This exposes the heartbeat used to propagate change.
