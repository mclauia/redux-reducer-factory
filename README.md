
# redux-reducer-factory
> A configurable createReducer factory for adding custom reducer enhancers.

A configurable createReducer factory (i.e. it is a factory for making a createReducer function) that essentially allows projects to insert extra generic reducers (enhancers) before and after individual domain reducers are invoked, without muddying those individual domains.

Meant to be used underneath redux's `combineReducers`; use `combineReducers` for reducer composition, and `createReducer` for your action-handling reducers.

Multiple configurations of `createReducerFactory` allows different reducers to use only the enhancers that apply to them.

NOTE: Obviously, reducers must still be pure, and reducer enhancers should have no side effects; they're just common, chained reducers.

This is for logic that should happen within each reducer's domain, and not at the store level. Specifically this was written to fulfill two needs: firstly to find a place to meld a partially hydrated state tree (e.g. from a local storage storeEnhancer) with any missing properties from initialState (which belongs to the reducer domain), and secondly to provide helpful runtime logging and validation for myself and other developers new to working with redux & pure reducers. Rather than lump these two totally different tasks together into one reducerFactory, it made sense to break it apart into a configurable module.

## installation

```
@todo npm install or whatever
```

## usage

### createReducerFactory(beforeReduceEnhancers, afterReduceEnhancers)
`=> Function` Returns a createReducer function with enhancements.

Expects `beforeReduceEnhancers` and `afterReduceEnhancers` to be arrays of reducer/enhancer functions with the signature below. Enhancers are invoked in array order before and after the reducer invocation with the following parameters:
#### beforeEnhancer(handler, initialState, previousState, chainedEnhancedState)
`=> enhancedState` Returns enhanced state, which will be passed as `enhancedPreviousState` to the next enhancer in the chain.

`previousState` will get the original previous state for this dispatch; `chainedEnhancedState` will get the latest state from the enhancer chain.
#### afterEnhancer(handler, initialState, enhancedPreviousState, chainedEnhancedState)
`=> enhancedState` Returns enhanced state, which will be passed as `chainedEnhancedState` to the next enhancer in the chain.

`chainedEnhancedState` will get the latest next state from the enhancer chain.

`enhancedPreviousState` will get the (enhanced) state before action handling.

Enhancers are given the handler function for the dispatched action in case they want to parse it, or if they need to change enhancements based on whether there is a handler.

### configure

Somewhere in your project utilities, add a module that configures the reducer factory:

```
// File: e.g. utils/createReducer.js

// import the reducer-enhancer library:
import createReducerFactory from 'redux-reducer-factory';

// import any enhancers you desire, for example:
import meldPartialState from 'redux-reducer-meld';
import { reducerBotBefore, reducerBotAfter } from 'redux-reducer-bot';

// set up before/after enhancers:
const beforeReduce = [
	meldPartialState
];
const afterReduce = [];

if (__DEV__) {
	beforeReduce.push(reducerBotBefore);
	afterReduce.push(reducerBotAfter);
}

// then create and export your reducer creators:
export default createReducerFactory(beforeReduce, afterReduce);
```


### createReducer(initialState, actionHandlers)
`=> Function`
Returns an enhanced reducer function that directs actions to the appropriate handler based on type and provides initialState.

#### initialState
The reducer's initial domain state. Any native JavaScript object, including ES6 collections.

Note: currently no support for Immutable.js.

#### actionHandlers
A plain object with action types as keys, and handler functions as values.


```
// File: e.g. reducers/fooReducer.js
import createReducer from '../utils/createReducer';

const initialState = {
	foo: 'bar',
    foos: 'ball'
};

const actionhandlers = {
	FOO_ACTION_TYPE: (state, action) => Object.assign({}, state, { foo: action.payload })
};

export default createReducer('foo', initialState, actionHandlers);
```

Or, to use action type constants, use the computed property syntax:
```
import { UPDATE_FOO } from '../constants';
const actionHandlers = {
    [UPDATE_FOO]: (state, action) => ({ foo: action.payload })
}

```

### without configuration

This module also exports a plain createReducer function that uses no enhancers.
```
import { createReducer } from 'redux-reducer-factory';

export default createReducer('foo', initialState, actionHandlers);
```

## License

MIT © [Ian McLaughlin](mclauia.com)

Built on top of yeoman generator-node