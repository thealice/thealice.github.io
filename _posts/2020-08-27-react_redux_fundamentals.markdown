---
layout: post
title:      "React Redux Fundamentals"
date:       2020-08-27 18:39:29 -0400
permalink:  react_redux_fundamentals
---


For my final Flatiron portfolio project, I had to write a React app that made use of Redux middleware to respond to and modify state changes. I had just mastered React and passing down props and adding a local state to a class component made sense to me, so I wasn't sure why we had to complicate things with Redux.

At first I had trouble grasping the point of Redux besides adding what seemed like a billion new steps to complete a simple state change.

![Still from 'The Way Things Go'. The tire will begin to move when the newpaper is ignited, setting off a 30 minute long series of similiar events that may or may not end in a simple task being completed.](https://i.imgur.com/aB7GLXx.png)

On top of that, all the steps came with a bunch of new confusing terminology. My brain, not exactly in peak condition busy as it was constantly re-learning how life is supposed to work in 2020, just about broke waiting for everything to fall into place.

![Action creator, don't leave the house, store, stop touching your face, mapDispatchToProps…was that teargas???](https://media.giphy.com/media/3o7aCWDyW0PJCsxHna/source.gif)

Now that I'm (am I?) on the other side of that, I thought I'd break down a few of the fundamental concepts in case it helps someone else avoid what I'm calling the Redux Brain Break.

## State/Store

I'm going to start with state, since this is not an entirely new concept if you're familiar with React. The state is where we can stash useful information to use later on. It looks like a plain old JavaScript object (POJO) with key/value pairs. The values can be strings, arrays, objects, anything that is expected to change at some point during the component's life. One key difference between props and state is props are used for properties that are passed down from a parent component, so they can't be easily changed. State is typically for storing data that needs to change at some point throughout the component's life.

```
state = {
  locations: [{...}],
  formData: {...},
  currentUser: "Lisa Simpson"
}
```

States can be local to Components (meaning scoped to be used anywhere within a class component), but using Redux it can be stored in a globally accessible state. In fact, the whole purpose of Redux is to manage this state store. In order to access it, a component needs to connect to that store, it's not accessible by default. Still with me?

Let's say we're fetching data from an external API and storing that data in the Redux store because we may be adding or changing that data. For example, in my app I'm fetching location data, and people can add locations and send them back to the API to be saved in a database.

There are components that aren't going to need to use this location data and don't need to re-render every time it changes. That's why they're not automatically connected to the store, it helps with performance. But what if a component does need to access that data?

## mapStateToProps()

That is where mapStateToProps() comes in. Connect the component to the Redux store and use mapStateToProps() to indicate which parts of the state its going to use, which will bring that data from the store into your component's props. mapStateToProps() is a function that takes in the store'sstate as an argument and returns an object with the new props to give to the component. These new props are given as an argument when using the connect method to connect to the Redux store. Eg.
```
const mapStateToProps = state => {
    return {
        formData: state.locationForm
    }
}
export default connect(mapStateToProps)(NameOfComponent);
```
Mapping the formData to a component's props allows that component (in this case a form to add or edit a location) to pull from and update that formData. this.props.formData would now return an object that might look like:
```
{
    name: "Apple tree",
    address: "2821 Telegraph Ave, Berkeley, CA 94705",
    lat: 37.8588635,
    lng: -122.2591357
}
```
But what if we're loading a fresh form to add a new location and we want to reset the formData so it doesn't display anything until a user types new information into the form? Or someone needs to edit that data? We can't do something like create a function that returns this.setState({new form data goes here}) to update the state. No…that would be too easy for Redux.

![](https://media.giphy.com/media/JRhS6WoswF8FxE0g2R/source.gif)

## Breaking Down The Redux Flow
Remember the Rube Goldberg-like series of events I alluded to earlier? Well, here's where that really kicks in. To update the store'sstate a very specific series of events must take place. It doesn't really feel like a linear path, though, at least not in my brain (which happens to be very prone to tangents). This is what that path looks like to me:

1. Something happens on the client's side to trigger a change in state. This might be something like clicking on a button or typing in a form field. We'll call this an event.
2. The event handler (the function in your component that determines what is to be done when someone triggers an event) specifies what part of the state should be updated using the store's dispatch() method. The dispatch method takes in an argument with some information about how the state should be updated. Within the event handler that might (but probably doesn't, see mapDispatchToProps below) look like: `this.props.store.dispatch(action)`. The dispatch function doesn't really return anything*, it's there tell the store to fire off (aka dispatch) a state change as well as what type of change. But what's that argument passed into the dispatch method?
3. That's called an action. An action is a POJO with a 'type' key. Eg. `{ type: 'RESET_FORM' }`. Actions can also have keys with additional information, often called their payload, eg. `{ type: 'UPDATE_FORM', formData: {...} }`. Yes, that stationary POJO is called an action. Why? I don't know. I guess it has something to do with how it tells the reducer what action to take.
4. Oh cool! Another word with no obvious meaning! Despite the name, the reducer is where most of the action happens. Back when we created the store (this happens at the very top of our app, often in index.js), it is passed a reducer (or a set of reducers) as an argument. A reducer is a pure function that takes in the previous state and an action as arguments, and returns a copy of the state, now updated. It looks like: `reducerName(initialState, action) => updatedState`. The reducer is what actually returns our updated state!

So, eventHandler(event) => dispatch(action) => reducer(initialState, action) => Updated State!

![](https://media.giphy.com/media/inyqrgp9o3NUA/source.gif)

## More on Reducers
I said above that dispatch tells the store what action to take. I also said it triggers this action. Well, it triggers the action by passing the action object as an argument along with the current state to the reducer. If it is keeping track of more than one reducer the state and action are passed to all of them until a reducer with a switch case for the action's type is found and a new state is returned based on what the reducer tells it to return. In the case that the `action.type ==='RESET_FORM'`, for instance, it might return a copy of the state with blank form fields. I mention a "copy" of the state, because reducers are pure functions. This means there are no side effects and we can't change the original state, so we return a modified copy of it.

What you name your reducer determines the key for the updated part of the state we're working on, (which if you remember was determined by the action). If the reducer is called locationForm.js, then it will return a copy of the state which includes locationForm and its data as a key/value pair. If the initial state included another key, perhaps from another reducer, it will include that as well. For example:
```
state = {
   locationForm: {
                     name: "a name",
                     address: "123 Sesame St., Berkeley, CA",
                     etc.
                  }
    locations: [{...}, {...}, {...}]
}
```

## mapDispatchToProps()
When you connect your component to the store, I mentioned above you will pass in mapStateToProps (the function) as an argument. If you're updating the state in this component, you'll also want to pass in either a mapDispatchToProps() function, or an object. Then the component will have access to the dispatch method as dispatch() rather than this.props.store.dispatch() and you can dispatch your action from the event handler.
code at the bottom of a LocationForm component. mapDispatchToProps is passing in an action creator to dispatch, the return value of which is an action object

\*Dispatch CAN return a promise (when asynchronously fetching data, for instance) when paired with middleware like Redux Thunk.


