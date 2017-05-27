# Model

This is a JavaScript class intended to serve as an alternative to React's normal `state` system. It utilizes ES6 Proxy to recursively intercept all assignment and deletion operations on all nested objects, and trigger a "publication" to registered subscribers with the new value.

In practical terms, this gives the developer a completely mutable, arbitrarily complex state object that will re-render a React component when any part of it is modified. This even applies to array mutation methods.

# Usage

```jsx
var Model = require('mutable-model');

class ShoppingCart extends React.Component {

  componentWillMount() {
    this.model = new Model(this, {
      user: {
        firstName: 'Bob',
        lastName: 'Belcher'
      },
      items: [ ]
    });
  }

  render() {
    ...
  }

  handleUserFirstNameChange(newVal) {
    /**
     * Normal React method:
     *
     * var newUser = this.state.user;
     * newUser.firstName = newVal;
     * this.setState({
     *   user: newUser
     * });
     *
     */

    this.model.user.firstName = newVal;
  }

  handleAddItemClick(item) {
    /**
     * Normal React method:
     *
     * this.setState({
     *   items: this.state.items.concat([ item ]);
     * });
     *
     */

    this.model.items.push(item);
  }

}
```

# About Performance
It should be noted that unless extra measures are taken, Model tends to sacrifice a little bit of performance in exchange for better syntax. By that I mean, it can trigger a few extra renders. Consider the following:
```jsx
this.setState({
  varOne: 12,
  varTwo: 'foo'
});
```
In the normal React style, this would only trigger one render. Whereas:
```jsx
this.model.varOne = 12;
this.model.varTwo = 'foo';
```
This would trigger two renders, because there are two separate assignment operations. Also, in cases like array mutation, there will often be two renders triggered; one for the addition (or removal) of the item, and one for the modification of `.length` (a separate, internal property assignment).

That said, this can be avoided on a case-by-case basis by mimicking the React style. This:
```jsx
this.model.myArr = this.model.myArr.concat([ 'foo' ]);
```
will only trigger one render, because there is only one assignment being performed. This also works for objects, in addition to arrays.

In summary, take advantage of the mutation syntax when it will simplify your code, and feel free to use the single-assignment style if you need to optimize. Model gives you the flexibility to do both.

# Flexibility
Model was designed and intended for managing complex React component state, but a subscriber to a Model doesn't have to be a React component, it can be any function. You could, for example, say:
```jsx
var myModel = new Model(
  function(model) {
    console.log(JSON.stringify(model));
  },
  {
    someNumber: 12
  }
);

myModel.someNumber = 100; // would print "{someNumber:100}" to the console
```
Passing a component as a subscriber directly is just for convenience; under the hood, all it does is call the component's `setState({})` method with an empty object; this doesn't change the React state but still triggers an update.
