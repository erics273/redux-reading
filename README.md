# Redux

Redux is a state container for JavaScript applications, meaning that it is a wrapper for application state and it implements a clear pattern for how to modify and access that state. Redux was modeled after the Flux pattern and creates a single data store that acts as the sole source of truth for the application.

Redux doesn't require React; the two technologies aren't inherently bound together. Non-React applications can utilize Redux and React applications can function just fine without Redux. However, Redux has become the defacto state container for non-trivial React applications and you will commonly see them used together.

Redux doesn't implement a strict version of the flux pattern. Instead, it incorporates the basic concept of one-way data flow to decouple immutable state from the view layer of an application. To understand all of this, we'll first talk about the flux pattern.


### Flux
The most basic Flux pattern incorporates four things: Actions, Dispatchers, The Store, Views. The idea is simple. Actions get triggered by the application. These Actions Dispatch messages to update The Store. Once The Store is updated, the View rerenders.

### Action->Dispatcher->Store->View

- NOTE: Flux isn't a thing, it's a pattern for managing data flow and comes in multiple flavors. Don't get too bogged down in understanding the theory of Flux. What matters is that you should try to understand *how data moves through your application.*


### Redux Flux
The Redux version of Flux is different in a couple of ways.

- Redux doesn't implement dispatcher objects. Instead, the dispatch function sends actions directly to the store. Then, additional functions called "reducers" create a new app state by cloning the current app state and merging changes from the action.
- Redux places the entire app state in one single object. However, components never change that global state directly. The only way to update the app state is to dispatch actions to reducers. The reducers then update the state.

This general pattern provides a couple of benefits:
- Application components can have restricted access to application state by restricting which actions they can dispatch.
- Because reducers always return a new application state, the app state is immutable which eliminates a whole class of bugs and technical overhead for application structure.
- The architecture of the application is clear and consistent:
    - Actions are merely simple objects.
    - Reducers are functions that return a new (possible updated) application state depending on the actions passed to them.
    - Dispatching an action to the store requires access to the dispatch function.

To sum up this introduction, Redux provides a structured way to access and modify a single data store. It abstracts access to that data store through the flux pattern and provides a single source of truth for the application state as a whole. It can take some time to understand all the moving parts of a Redux application in the context of a React application, but that is mostly done to the way that Redux abstracts messaging and data flow. Redux only introduces a few concepts: Dispatching, Actions, Action Creators, Reducers, and the store.

This article discusses the major components of Redux and how to implement them in a React application. To demonstrate the concepts, we will start with a React application that does not implement Redux and then modify it to move a component's local state to the Redux store. Then, we will implement actions, reducers, and a container component to access that store.


## Redux Moving Parts
To begin, we should discuss each of the Redux patterns in a little more detail. Talking about these patterns will help to frame up how they all fit together and why they are part of Redux. It will also provide a little background for the motivations behind why React/Redux applications follow a specific structure.


### Actions
Actions are simple JavaScript objects that get passed to Reducers. These objects have no hidden or automagical features. They are simply objects used to pass data about the action that has taken place. Reducers then use the data contained in Actions to determine how to build a new application state.

Actions should have a "type" attribute, and the value of that attribute should be an all caps, snake case string. For example:

```js
{
    type: "UPDATE_THING"
}
```
Actions can also include additional arbitrary data that might be required by the Reducer to which it's passed. For instance, the Reducer that processes the "UPDATE_THING" Action might need the id and a new value for the thing it is updating:

```js
{
    type: "UPDATE_THING",
    id: 27,
    value: "icecream"
}
```
The amount of data passed to Reducers with Actions is up to you and what you are attempting to accomplish in your application.


### Action Creators
Action Creators are functions that return Actions. There is nothing special about these functions, they are simply JS functions that return an object. The idea behind Action Creators is that the are reusable and portable functions that can be shared by components in your application.

The following Action creator returns a "UPDATE_THING" Action but can be passed arbitrary data for the "id" and "value" properties. Our application might have 1000 Thing objects, all with different "ids" and "values." Action Creators can also define default values for some (or all) of the data attributes that get passed along to the reducer.

```js
function updateThing( id, value = "icecream" ){
    return {
        type: "UPDATE_THING",
        id,
        value
    };
}

updateThing( 27 );
/*{ type: "UPDATE_THING",
    id: 27,
    value: "icecream" }*/

updateThing( 12, "hotdogs" );
/*{ type: "UPDATE_THING",
    id: 12,
    value: "hotdogs" }*/
```


### Dispatch
The `dispatch()` function is provided by Redux as part of the Store and expects to be passed an Action. It will then dispatch that Action to the Store which will process that action with a Reducer.

You can pass dispatch an Action directly like so:

```js
store.dispatch( {
    type: "UPDATE_THING",
    id,
    value
} );
```

However, typically dispatch will be passed the return value of an Action Creator. The Action Creator will build and return an Action based on the properties passed to it. That Action is then passed to the dispatch function, which dispatches it to the store. Dispatch will also ensure that the correct Reducer processes the Action.

```js
store.dispatch( updateThing( 27 ) );
store.dispatch( updateThing( 12, "hotdogs" ) );
```

### Reducers
Reducers are the functions that Redux uses to process actions and then create a new application state. Reducers are passed two things: The current application state and the Action object.

The Reducer then utilizes the action type value to determine how to respond. In the following example, when "UPDATE_THING" is dispatched, the `thing()` reducer will merge `action.id` and `action.value` with the current state. If a different action is dispatched, let's say "DELETE_THING," then the `thing()` will simply return the current state and do nothing.

```js
const thing = ( state = {}, action )=>{

    switch( action.type ){
        case 'UPDATE_THING':
            return Object.assign( {}, state, {
                id: action.id,
                value: action.value
            } );

        default:
            return state;
    }
}
```

One common point of confusion is to understand where the state "lives." So far we haven't defined a global object to hold state. We haven't talked about the data at all. The reason we haven't talked about manipulating or creating data objects is that the reducers handle those tasks. When the application is booted up, the store is empty. It isn't until the reducers have been run that the state takes shape.

In the previous example, the signature for `thing()` is `const thing = ( state = {}, action )`. The default value of state is an empty string. The "UPDATE_THING" reducer is the only part of our application that can build application state. When run, it will merge an id and a value with the state.

```js
// Application starts up...
// Application state = {}
store.dispatch( updateThing( 12, "hotdogs" ) );
// Initial application state { id: 12, value: "hotdogs" }
```

Reducers can process multiple action types, simply add cases to the switch statement and respond appropriately to the Action that gets passed in.

Eventually, you will see how Reducers are used to create the store and to tie everything together, but for now, you only need a conceptual understanding of what is happening within Redux.

## Building A Redux Clock
To demonstrate all of these new concepts we will start with a functioning React Application that displays a simple clock. The initial version of this app won't use Redux. Instead, the state is stored locally in the `Clock` component. Once, we've got the React version working; we'll modify the code by adding Redux and all the necessary wiring to interact with the store.

Admittedly, this example is trivial. It doesn't matter whether or not the state is stored locally or in a Redux store, but simple examples at least let us see all the moving parts and how they fit together. Once we've completed the Redux version of our Clock, I'll leave it to you to determine whether local state or Redux state is more appropriate in this application.

### Setting up the Application with Local State
Use `create-react-app` to spin up a new application called "clock-time" and open the app in your favorite text editor. Your initial project structure should look like this:

```
my-app
├── README.md
├── node_modules
├── package.json
├── .gitignore
├── public
│   └── favicon.ico
│   └── index.html
│   └── manifest.json
└── src
    ├── components
    │   └── clock
    │       └── Clock.css
    │       └── Clock.js
    └── App.css
    └── App.js
    └── App.test.js
    └── index.css
    └── index.js
    └── logo.svg
    └── registerServiceWorker.js
```

Then, create a *clock* folder under */src/components* with `Clock.css` and `Clock.js` inside.**

Copy the following code into `Clock.js`. Read through the comments to understand how the Clock component works:

```javascript
import React, { Component } from 'react';
// import css for Clock
// right now it's empty,
// so you can pretty much ignore it ;)
import './Clock.css';

class Clock extends Component {

    // Override the Component constructor
    // Create the initial local state for the Component
    constructor(props) {
        super(props);
        this.state = {date: new Date()};
    }

    // When the component is added to the view
    // Start a timer that will run the tick function
    // every second
    componentDidMount() {
        this.timerID = setInterval(
            () => this.tick(),
        1000
        );
    }

    // When the component is removed from the view
    // stop the timer
    componentWillUnmount() {
        clearInterval(this.timerID);
    }

    // Update the local state.date to a new Date value
    // each time tick() is called
    tick() {
        this.setState({
            date: new Date()
        });
    }

    // Whenever this component is rendered
    // format the date and print it to screen
    render() {
        return (
            <div>
                <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
            </div>
        );
    }
}
export default Clock;
```
Now, update `App.js` with the following code, which includes the Clock component and then renders it within the "App-header" div, beneath the logo:

```js
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';
import Clock from './coomponenets/Clock';

class App extends Component {
    render() {
        return (
            <div className="App">
                <div className="App-header">
                    <img
                        src={logo}
                        className="App-logo"
                        alt="logo"
                    />

                    {/* Render the clock component */}
                    <Clock />

                </div>
            </div>
        );
    }
}
export default App;

```

You should be able to run this application with `npm start` and see the clock counting up.

### Adding Redux to the Application
To Add Redux to the clock application, we'll need to install Redux and React-Redux.

```bash
$ npm install redux react-redux  --save
```

Once the dependencies are installed, we need to make some new directories and some new files:

1. Within the "src" directory, create three additional directories: "actions", "containers", and "reducers".
2. Within "src/actions" create a file called "index.js."
3. Within "src/containers" create a file called "ClockContainer.js."
4. Within "src/reducers" create two files, "clock.js" and "index.js."

Your final folder structure should look as follows:

```
my-app
├── README.md
├── node_modules
├── package.json
├── .gitignore
├── public
│   └── favicon.ico
│   └── index.html
│   └── manifest.json
└── src
    ├── actions
    │   └── index.js
    ├── components
    │   └── clock
    │       └── Clock.css
    │       └── Clock.js
    ├── containers
    │   └── ClockContainer.js
    ├── reducers
    │   └── clock.js
    │   └── index.js
    └── App.css
    └── App.js
    └── App.test.js
    └── index.css
    └── index.js
    └── logo.svg
    └── registerServiceWorker.js
```

Now that all of the files are in place let's start adding code to the files and discussing what each file is doing.

### "src/actions/index.js"
This actions file contains one Action Creator, `updateTime()`, that returns an Action "UPDATE_TIME". Update time doesn't accept any properties, so it will always return the same Action.

```js
export function updateTime(){
    return {
        type : 'UPDATE_TIME'
    }
}
```

### "src/reducers/clock.js"
The clock Reducer responds to one action type, "UPDATE_TIME," and generates a new date whenever that Action is passed in. That date value is formatted using `toLocaleTimeString()` and then stored in the `store.clock.time`. If you aren't familiar with `Object.assign,` read [about it here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign). In layman's terms, `Object.assign` is combining the current state with the new state and returning a brand new object that is the combination of both.

If an Action other than "UPDATE_TIME" is passed in, the clock Reducer returns the current state.

```js
const clock = ( state = {}, action )=>{
    switch( action.type ){
        case 'UPDATE_TIME':
            const date = new Date();
            return Object.assign( {}, state, {
                time: date.toLocaleTimeString()
            } );

        default:
            return state;
    }
}
export default clock;
```

### "src/reducers/index.js"
Also included in the reducers folder is "index.js." This file imports `combineReducers()` from Redux and also imports the clock Reducer. Then, the clock Reducer is passed (in an object) to `combineReducers`, which then returns the combined set of reducers for the application. This step may seem superfluous, but combining reducers is essential for larger application with multiple reducers.

If this application had multiple reducers, then they would be imported into this file and passed into combineReducers along with clock.

```js
import { combineReducers } from "redux";
import clock from "./clock.js";

const clockApp = combineReducers( { clock } );

export default clockApp;
```

### "src/containers/ClockContainer.js"
Next, add the following code to "ClockContainer.js." ClockContainer uses `connect()` to send properties down to Clock, but doing so requires a couple of steps.

`connect()` is expecting to be passed a couple functions `mapStateToProps()` and `mapDispatchToProps`.

`mapStateToProps()` will be passed the state of the application. Within the body of `mapStateToProps(),` the state can be filtered and formatted so that Clock is passed only the state it needs. In this case, `state.clock.time` is mapped to `time`. `time` will be passed down to Clock as one of its props.

The benefit of this processing is that application state is available only where it is needed. Imagine a larger application with tons of state. You might not want the Clock component to have access to the entire application state when it is only going to use one of the values. `mapStateToProps()` allows us to grab that one value and pass it on to Clock without passing on anything else.

`mapDispatchToProps` is passed the `dispatch()`. Notice that we imported the `iupdateTime()` Action Creator from `actions/index.js`. We want Clock to be able to dispatch that action without direct access to the dispatch function. To make that functionality available, we create an object with one property "updateTime" and then wrap our dispatch statement in an anonymous function. "updateTime" will also get passed to Clock as a property.

Both `mapStateToProps()` and `mapDispatchToProps` are passed to `connect` which returns a function that is expecting a component, in this case, Clock. That function will return a new component that renders Clock and passes along the props created by `mapStateToProps()` and `mapDispatchToProps`.

What that means is that we will need to alter the application to render ClockContainer instead of Clock. It works out because ClockContainer does the work of accessing the store and then passes the appropriate props down to Clock, which ClockContainer subsequently renders.

```js
import { connect } from "react-redux";
import Clock from "../components/clock/Clock";
import { updateTime } from "../actions";

const mapStateToProps = state => {
    return { time : state.clock.time };
};
const mapDispatchToProps = dispatch => {
    return { updateTime : () => dispatch( updateTime() ) }
};

const ClockContainer = connect( mapStateToProps, mapDispatchToProps )( Clock );
export default ClockContainer;
```

### "src/components/Clock.js"
Now that ClockContainer is wrapping Clock, we need to alter Clock to utilize the new props being passed down to it.

You should quickly notice several changes. First, the constructor has is gone because we aren't generating any local state for Clock. Instead, ClockContainer passes all relevant state as props from. Second, `tick()` is no longer needed because Clock doesn't directly update its state.

Within `componentDidMount()`, the prop `updateTime()` is called to initialize the state of the store. Remember, `updateTime()` runs `dispatch` and passes the "UPDATE_CLOCK" action to the clock Reducer. The clock Reducer then creates a new state with an updated time. Once the store is updated, the Components passed the new state and rendered. In this case, Clock is passed the new time in `this.props.time` which it subsequently renders.

This process demonstrates the Flux pattern:

1. `this.props.updateTime()` dispatches the Action "UPDATE_TIME."
2. Dispatch passes the Action to the store.
3. The clock Reducer responds to the Action by generating a new time and creating a new state
4. The new state then triggers a rerender of the view

```js
import React, { Component } from 'react';
import './Clock.css';

class Clock extends Component {
  componentDidMount() {
    this.props.updateTime();
    this.timerID = setInterval(
      () => {
        this.props.updateTime();
      }, 1000
    );
  }
  componentWillUnmount() {
    clearInterval( this.timerID );
  }
  render() {
    return (
      <div>
        <h2>It is { this.props.time }.</h2>
      </div>
    );
  }
}
export default Clock;
```


### "src/App.js"
App.js has only one change. Instead of importing and rendering the Clock component, App imports and renders the ClockContainer component.

```js
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';
import ClockContainer from './containers/ClockContainer';

class App extends Component {
  render() {
    return (
      <div className="App">
        <div className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <ClockContainer />
        </div>
      </div>
    );
  }
}
export default App;
```

### "src/index.js"
`index.js` is the starting point for our application and is the appropriate place to connect all the pieces of our application. Our main job in this file is to create the store and then pass it down to the application so that it is available.

First, import `Provider`, `createStore()` and `clockApp`.

Then create a variable called "store." Pass the clockApp Reducer to `createStore()` and assign the returned value to "store". `createStore()` will create the store from the Reducers and set up access.

Finally, you'll need to edit the render function and wrap the App component with the Provider component. Provider takes one attribute "store."  Pass it the the store variable we just created. Provider will pass the store down through App and to the components that have access to it, in this case, ClockContainer.

All of this code is boilerplate, but it's important to correctly wire up the app, or you won't have access to the store.

```js
import React from 'react';
import { render } from 'react-dom';
import { Provider } from 'react-redux';
import { createStore } from 'redux';
import clockApp from './reducers';
import './index.css';
import App from './App';
import registerServiceWorker from './registerServiceWorker';

let store = createStore( clockApp );

render(
    <Provider store={store}>
        <App />
    </Provider>,
    document.getElementById( 'root' )
);
registerServiceWorker();
```

Run your application and you should see the exact same thing as the first app.

## Conclusion
This application example has been relatively trivial, but all the principles discussed do scale well. Once you understand how Actions are created and passed to the store, you should have a pretty good grasp for how to structure your React/Redux applications.


## Further Reading
- [https://redux.js.org/](https://redux.js.org/)
- [Containers and Components](http://www.thegreatcodeadventure.com/the-react-plus-redux-container-pattern/)
- [Flux and Redux](https://www.fullstackreact.com/p/intro-to-flux-and-redux/)

