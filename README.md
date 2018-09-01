# Simple State

Simple state management.

## Install

```shell
# NPM
$ npm install -S simple-state
```

```shell
# yarn
$ yarn add simple-state
```

## Setting up your state

#### Simple Example

To use `simple-state`, just instantiate the `SimpleState` class:

```javascript
import { SimpleState } from 'simple-state';
  
const userState = new SimpleState();
console.log(userState.get()); // {}
```

Each instantiated state is a separate instance of the `simple-state` class, and contains all the methods needed
to use the state.

#### Default State

You can add a default state by passing in an object as the first argument:

```javascript
import { SimpleState } from 'simple-state';

const userState = new SimpleState({
  firstName: 'John',
  lastName: 'Doe',
  age: 5,
});

console.log(userState.get('firstName')); // 'John'
```

#### Extend State

If you need to add complicated logic or ajax calls when you change state, we can extend the `SimpleState` class
with any sort of methods that we want:

```javascript
import { SimpleState } from 'simple-state';

class UserState extends SimpleState {
  isAdult() {
    return this.state.age >= 18;
  }
  hasLastName() {
    return !!this.state.lastName;
  }
}

const userState = new UserState({ age: 5 });
console.log(userState.isAdult()); // false
``` 

## Using your state

Using the example above, we can now use our `userState` in our app.

#### .get([key])

The easiest way to get your entire state is to use the `.get()` method without any parameters. To get just one
property on state, just pass in the prop key:

```javascript
import userState from './user-state';

userState.get(); // entire state object
userState.get('age'); // just the state.age property
```

#### .set(state)

To set data on your state, you need to use the `.set()` method. This can be done within your extending class, or your
instantiated instance:

**inside your extending class:**

```javascript
import { SimpleState } from 'simple-state';

class UserState extends SimpleState {
  updateName(name) {
    const fullName = name.split(' ');
    this.set({
      firstName: fullName[0],
      lastName: fullName[1],
    });
  }
}

const userState = new UserState();
userState.updateName('John Doe');
```

**from the instantiated instance:**

```javascript
import userState from './user-state';

userState.set({
  firstName: 'John',
  lastName: 'Doe',
});
```

#### .subscribe(next [, props])

Subscribe allows us to get incremental changes over time. Much like how RxJS works, we subscribe to the state and pass
in a callback to be called on every change:

```javascript
// from the instantiated instance:
import userState from './user-state';

userState.subscribe((state) => console.log(state));
```

If your state is big, there's a good chance that not EVERY observer will want EVERY change. To make sure we're being
smart with our updates, we can pass in an array of key names to tell our state what props to watch for and let us
know if any changes have occurred:

```javascript
import userState from './user-state';

userState.subscribe((state) => console.log(state), ['firstName']);
// only changes to 'firstName' will fire an event here
```

## Practical React Example

**user-state.js**

```javascript
import SimpleState from 'simple-state';

class UserState extends SimpleState {
  isAdult() {
    return this.state.age >= 18;
  }
}

export default new UserState();
```

**app.js**

```javascript
import React, { Component } from 'react';
import userState from './user-state';

export default class DisplayAge extends Component {
  constructor() {
    super();
    userState.subscribe(({ age }) => {
      this.setState({ age, isAdult: userState.isAdult() });
    }, ['age']); // this component only cares about the 'age' property
  }
  state = {
    age: userState.get('age'),
    isAdult: userState.isAdult(),
  }
  
  updateName = () => {
    userState.set({ age: 18 });
  }

  render() {
    return (
      <div>
        <div>Age: {this.state.age}</div>
        <div>Is adult: {this.state.isAdult ? 'true' : 'false'}</div>
      </div>
    );
  }
}
```
