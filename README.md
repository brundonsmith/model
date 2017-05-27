# Model

This is a JavaScript class intended to serve as an alternative to React's normal `state` system. It utilizes ES6 Proxy to recursively intercept all assignment and deletion operations on all nested objects, and trigger a "publication" to registered subscribers with the new value.

In practical terms, this gives the developer a completely mutable, arbitrarily complex state object that will re-render a React component when any part of it is modified. This even applies to array mutation methods.

# Usage

```jsx
import Model from 'mutable-model';

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
