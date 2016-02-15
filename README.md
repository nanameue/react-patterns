React
=====
Adjusted guided for frontend development using React at Nanameue.jp.
Some examples were written as ES2015, some with legacy js (will update soon).

## Table of Contents

1. Organization
  1. [Component Organization](#component-organization)
  1. [Formatting Props](#formatting-props)
1. Patterns
  1. [Computed Props](#computed-props)
  1. [Compound State](#compound-state)
  1. [Use Sub-render](#use-sub-render)
  1. [View Components](#view-components)
  1. [Container Components](#container-components)
  1. [Compound Conditions](#compound-conditions)
1. Practices
  1. [Naming Handle Methods](#naming-handler-methods)
  1. [Naming Events](#naming-events)
  1. [Using PropTypes](#using-proptypes)
  1. [Using Entities](#using-entities)
1. Gotchas
  1. [Tables](#tables)
1. Libraries
  1. [classnames](#classnames)
1. Other
  1. [JSX](#jsx)
  1. [react-rails](#react-rails)
  1. [flux](#flux)

---


## Component Organization

* class definition
  * proptypes
  * getDefaultProps
  * getInitialState
  * constructor
    * event handlers
  * 'component' lifecycle events
  * sub-render
  * render


```javascript
var Person = React.createClass({
  
  propTypes = {
    name: React.PropTypes.string
  };
  
  getDefaultProps: function() {
    return {
	  name: 'Guest'
	};
  },  
 
  componentWillMount: function() {

  },

  componentDidMount: function() {
    // React.getDOMNode()
  },
  
  componentWillReceiveProps: function(nextProps) {
  },
  
  shouldComponentUpdate: function(nextProps, nextState) {
  },

  componentWillUnmount: function() {

  },

  renderMessage: function() {
    return (
      <div>This is a message</div>
    );
  },

  render: function() {
    return (
      <div onClick={this.handleClick}>
        {this.props.name}
        {this.renderMessage()}
      </div>
    );
  }
  
});

```

**[⬆ back to top](#table-of-contents)**

## Formatting Props

Wrap props on newlines for exactly 2 or more.

```html
// bad
<Person
 firstName="Michael" />

// good
<Person firstName="Michael" />
```

```html
// bad
<Person firstName="Michael" lastName="Chan" occupation="Designer" favoriteFood="Drunken Noodles" />

// good
<Person
 firstName="Michael"
 lastName="Chan"
 occupation="Designer"
 favoriteFood="Drunken Noodles" />
```

**[⬆ back to top](#table-of-contents)**

---

## Computed Props

Use getters to name computed properties.

```javascript
  // bad
  firstAndLastName: function() {
    return `${this.props.firstName} ${this.props.lastname}`;
  }

  // good
  getFullName: function() {
    return `${this.props.firstName} ${this.props.lastname}`;
  }
```

See: [Cached State in render](#cached-state-in-render) anti-pattern

**[⬆ back to top](#table-of-contents)**

---

## Compound State

Prefix compound state getters with a verb for readability and make sure that it always returns boolean.

```javascript
// bad
happyAndKnowsIt: function() {
  return this.state.happy && this.state.knowsIt;
}
```

```javascript
// good
get isHappyAndKnowsIt: function() {
  return this.state.happy && this.state.knowsIt;
}
```

See: [Compound Conditions](#compound-conditions) anti-pattern

**[⬆ back to top](#table-of-contents)**

## Use Sub-render

Use subrender to make render function easy to read. Ternary looks ugly, please do not use it.
```javascript
// bad
render () {
  return (
    <div>
      {this.props.name}
      {(this.state.smiling)
        ? <span>is smiling</span>
        : null
      }
    </div>
  );
}
```

```javascript
// good
renderSmilingStatement: function() {
  var text = ' is smiling.';
  if (!this.state.isSmiling) {
    text = '';
  }
  return <strong>{text}</strong>;
},

render: function() {
  return <div>{this.props.name}{this.renderSmilingStatement()}</div>;
}
```

**[⬆ back to top](#table-of-contents)**

## View Components

Compose components into views. Don't create one-off components that merge layout
and domain components. Unless the wrapper component contains extended logics.

```javascript
// bad. the wrapper only wrap <div> which doesn't cause any values and didn't 
// do anything with the inner component.
class PeopleWrappedInBSRow extends React.Component {
  render: function() {
    return (
      <div className="row">
        <People people={this.state.people} />
      </div>
    );
  }
}
```

```javascript
// good. BSRow does a wrapping job and can be reused.
class BSRow extends React.Component {
  render: function() {
    return <div className="row">{this.props.children}</div>;
  }
}

class SomeView extends React.Component {
  render: function() {
    return (
      <BSRow>
        <People people={this.state.people} />
      </BSRow>
    );
  }
}
```


**[⬆ back to top](#table-of-contents)**

## Container Components
Create a dumb component for reusability. Wrap it with another component to make it clever.
```javascript
// bad. Component contains extra logic and can not be reused.
class PersonRow extends React.Component {
  prefix: function(name) {
    return 'Dr.' + name;
  }, 
  render: function() {
    return (
      <div className="person-row">
        <span>Name:</span> {this.prefix(this.props.person.name)}
      </div>
    );
  }
}
```


```javascript
// good. Dumb component is easy to reuse.
class Person extends React.Component {
  return (
    <div className="person-row">
      <span>Name:</span> {this.prefix(this.props.person.name)}
    </div>    
  );
}

// To reuse it, just create another wrapper that add extened logics.
class PrefixedPersonRow extends React.Component {
  prefix: function(person) {
    return {
      name: 'Dr.' + person.name,
      ....
    }
  },
  render: function() {
    return (
      <PersonRow person={this.prefix(this.props.person)} />
    );
  }
}
```

> A container does data fetching and then renders its corresponding
> sub-component. That's it. &mdash; Jason Bonta

Also, please use Flux pattern (reflux, redux, alt, ...) for data fetching. 

```javascript
// CommentList.js

// Bad. Component should contain only display-related logics.
class CommentList extends React.Component {
  getInitialState () {
    return { comments: [] };
  }

  componentDidMount () {
    $.ajax({
      url: "/my-comments.json",
      dataType: 'json',
      success: function(comments) {
        this.setState({comments: comments});
      }.bind(this)
    });
  }

  render () {
    return (
      <ul>
        {this.state.comments.map(({body, author}) => {
          return <li>{body}—{author}</li>;
        })}
      </ul>
    );
  }
}
```

**[⬆ back to top](#table-of-contents)**

---

## Cached State in `render`

Do not keep state in `render`

```javascript
// bad
render: function() {
  let name = `Mrs. ${this.props.name}`;

  return <div>{name}</div>;
}

// good
render () {
  return <div>{`Mrs. ${this.props.name}`}</div>;
}
```

```javascript
// best
getFancyName: function() {
  return `Mrs. ${this.props.name}`;
}

render: function() {
  return <div>{this.fancyName}</div>;
}
```

See: [Computed Props](#computed-props) pattern

**[⬆ back to top](#table-of-contents)**

## Compound Conditions

Don't put compound conditions in `render`.

```javascript
// bad
render () {
  return <div>{if (this.state.happy && this.state.knowsIt) { return "Clapping hands" }</div>;
}
```

```javascript
// better
get isTotesHappy() {
  return this.state.happy && this.state.knowsIt;
},

render() {
  return <div>{(this.isTotesHappy) && "Clapping hands"}</div>;
}
```

The best solution for this would use a [container
component](#container-components) to manage state and
pass new state down as props.

See: [Compound State](#compound-state) pattern

**[⬆ back to top](#table-of-contents)**

## Naming Handler Methods

Name the handler methods after their triggering event.

```javascript
// bad
punchABadger () { /*...*/ },

render () {
  return <div onClick={this.punchABadger} />;
}
```

```javascript
// good
handleClick () { /*...*/ },

render () {
  return <div onClick={this.handleClick} />;
}
```

Handler names should:

- begin with `handle`
- end with the name of the event they handle (eg, `Click`, `Change`)
- be present-tense

If you need to disambiguate handlers, add additional information between
`handle` and the event name. For example, you can distinguish between `onChange`
handlers: `handleNameChange` and `handleAgeChange`. When you do this, ask
yourself if you should be creating a new component.

**[⬆ back to top](#table-of-contents)**

## Naming Events

Use custom event names for ownee events.

```javascript
class Owner extends React.Component {
  handleDelete () {
    // handle Ownee's onDelete event
  }

  render () {
    return <Ownee onDelete={this.handleDelete} />;
  }
}

class Ownee extends React.Component {
  render () {
    return <div onChange={this.props.onDelete} />;
  }
}

Ownee.propTypes = {
  onDelete: React.PropTypes.func.isRequired
};
```

**[⬆ back to top](#table-of-contents)**

## Using PropTypes

Use PropTypes to communicate expectations and log meaningful warnings.

```javascript
MyValidatedComponent.propTypes = {
  name: React.PropTypes.string
};
```
`MyValidatedComponent` will log a warning if it receives `name` of a type other than `string`.


```html
<Person name=1337 />
// Warning: Invalid prop `name` of type `number` supplied to `MyValidatedComponent`, expected `string`.
```

Components may also require `props`.

```javascript
MyValidatedComponent.propTypes = {
  name: React.PropTypes.string.isRequired
}
```

This component will now validate the presence of name.

```html
<Person />
// Warning: Required prop `name` was not specified in `Person`
```

Read: [Prop Validation](http://facebook.github.io/react/docs/reusable-components.html#prop-validation)

**[⬆ back to top](#table-of-contents)**

## Using Entities

Use React's `String.fromCharCode()` for special characters.

```javascript
// bad
<div>PiCO · Mascot</div>

// nope
<div>PiCO &middot; Mascot</div>

// good
<div>{'PiCO ' + String.fromCharCode(183) + ' Mascot'}</div>

// better
<div>{`PiCO ${String.fromCharCode(183)} Mascot`}</div>
```

Read: [JSX Gotchas](http://facebook.github.io/react/docs/jsx-gotchas.html#html-entities)

**[⬆ back to top](#table-of-contents)**

## Tables

The browser thinks you're dumb. But React doesn't. Always use `tbody` in
`table` components.

```javascript
// bad
render () {
  return (
    <table>
      <tr>...</tr>
    </table>
  );
}

// good
render () {
  return (
    <table>
      <tbody>
        <tr>...</tr>
      </tbody>
    </table>
  );
}
```

The browser is going to insert `tbody` if you forget. React will continue to
insert new `tr`s into the `table` and confuse the heck out of you. Always use
`tbody`.

**[⬆ back to top](#table-of-contents)**

## classnames

Use [classNames](https://www.npmjs.com/package/classnames) to manage conditional classes.

```javascript
// bad
get classes () {
  let classes = ['MyComponent'];

  if (this.state.active) {
    classes.push('MyComponent--active');
  }

  return classes.join(' ');
}

render () {
  return <div className={this.classes} />;
}
```

```javascript
// good
render () {
  let classes = {
    'MyComponent': true,
    'MyComponent--active': this.state.active
  };

  return <div className={classnames(classes)} />;
}
```

Read: [Class Name Manipulation](https://github.com/JedWatson/classnames/blob/master/README.md)

**[⬆ back to top](#table-of-contents)**

## JSX

Use JSX in your code. https://facebook.github.io/react/docs/jsx-in-depth.html . It helps making the code easier to read for developer and designer.

**[⬆ back to top](#table-of-contents)**

## react-rails

[react-rails](https://github.com/reactjs/react-rails) should be used in all
Rails apps that use React. It provides the perfect amount of glue between Rails
conventions and React.

**[⬆ back to top](#table-of-contents)**

## flux

We recommend using [Reflux](https://github.com/reflux/refluxjs) for a small to medium size project, and [Redux](https://github.com/reactjs/redux) for large&complex project.

**[⬆ back to top](#table-of-contents)**
