# React Cookbook

## Index

-   [Declaration of component](#declaration-of-component)
-   [compute value](#compute-value)
-   [event handler naming](#event-handler-naming)
-   [Divide to multiple component better then multiple render in component](#component-better-then-nest-render)
-   [Move state to upper component better then provide public method](#move-state-to-upper-component-better-then-public-method)
-   [Container component](#container-componet)
-   [Render function must be pure](#pure-render)
-   [Always declare propTypes](#always-declare-proptypes)
-   [Use props to initial state](#use-props-to-initial-state)
-   [File naming](#file-naming)
-   [Classnames](#classnames)
-   [Miscellaneous](#miscellaneous)

<a name="declaration-of-component"/>

## Declaration of Component

Use ES6 class to declare component in following[Transform Babel class properties is a must](http://babeljs.io/docs/plugins/transform-class-properties/)

-   class
    -   propTypes
    -   defaultPropTypes
    -   constructor
        -   state declaration
    -   lifecycle events(shouldComponetUpdate,componentWillMount,componentDidMount...)
    -   computeFunc(getters,isXX)
    -   event handlers
    -   render

```javascript
class Person extends React.Component {
  static propTypes = {
    firstName: PropTypes.string.isRequired,
    lastName: PropTypes.string.isRequired,
    gender: PropTypes.string,
  }
  static defaultProps = {
    gender: 'M'
  }
  constructor (props) {
    super(props);
    this.state = { smiling: false }
  }

  componentWillMount () {}

  componentDidMount () {}

  get fullName () {
    return firstName + lastName;
  }
  get isMan(){
    return this.state.gender == 'M';
  }
  handleClick = () => {
    this.setState({smiling: !this.state.smiling})
  }

  render () {
    return (
      <div onClick={this.handleClick}>
        {this.fullName} {this.state.smiling ? 'is smiling.' : ''} to you.And who is a {this.isMan?' man':'woman'}
      </div>
    )
  }
}
```

**[⬆ Back to index](#index)**

<a name="compute-value"/>
## Compute value

use getter for computed properties in rendering

For getter that return boolean,use is- as prefixer

```javascript
  // bad
  render () {
    return (
      <div>
        {
          this.state.age > 18
            && (this.props.school === 'A'
              || this.props.school === 'B')
            ? <VipComponent />
            : <NormalComponent />
        }
      </div>
    )
  }

  // good
  get isVIP() {
    return
      this.state.age > 18
        && (this.props.school === 'A'
          || this.props.school === 'B')
  }
  render() {
    return (
      <div>
        {this.isVIP ? <VipComponent /> : <NormalComponent />}
      </div>
    )
  }
```

**[⬆ Back to index](#index)**

<a name="event-handler-naming"/>
## Event handler naming

Handler naming style:

-   begines with `handle`
-   ends with event type (like `Click`, `Change`)
-   use present tense

```javascript
// bad
closeAll = () => {},

render () {
  return <div onClick={this.closeAll} />
}
```

```javascript
// good
handleClick = () => {},

render () {
  return <div onClick={this.handleClick} />
}
```

if you same type of event handler but different naming,(like `handleNameChange` and `handleEmailChange`),that may be a signal to break down component

**[⬆ Back to index](#index)**

## Divide to multiple component better then multiple render in component

<a name="component-better-then-nest-render" />

When componet jsx in one render looks too cumbersome,it is better then divide to another component instead of one more render function.It can be es6 class syntax or functional stateless component(depends on the state).

```javascript
// bad
renderItem ({name}) {
  return (
    <li>
    	{name}
    	{/* ... */}
    </li>
  )
}

render () {
  return (
    <div className="menu">
      <ul>
        {this.props.items.map(item => this.renderItem(item))}
      </ul>
    </div>
  )
}
```

```javascript
// good
function Items ({name}) {
  return (
    <li>
    	{name}
    	{/* ... */}
    </li>
  )
}

render () {
  return (
    <div className="menu">
      <ul>
        {this.props.items.map(item => <Items {...item} />)}
      </ul>
    </div>
  )
}
```

P.S:Exception:multiple render layer in same component is acceptable when they share same state and need to setState in same component(but it worth to consider to put them in redux store).

**[⬆ Back to index](#index)**

<a name="move-state-to-upper-component-better-then-public-method" />

## Move state to upper component(or redux) better then provide public method

Component should not provide public method for modifying it's own state because it will destroy one way dataflow principle of react.

Therefore,the component state can move to upper component(or better,the redux store).It help all react component to share this state easily.

```javascript
//bad
class DropDownMenu extends Component {
  constructor (props) {
    super(props)
    this.state = {
      showMenu: false
    }
  }

  show () {
    this.setState({display: true})
  }

  hide () {
    this.setState({display: false})
  }

  render () {
    return this.state.display && (
      <div className="dropdown-menu">
        {/* ... */}
      </div>
    )
  }
}

class MyComponent extends Component {
  // ...
  showMenu () {
    this.refs.menu.show()
  }
  hideMenu () {
    this.refs.menu.hide()
  }
  render () {
    return <DropDownMenu ref="menu" />
  }
}

//good
class DropDownMenu extends Component {
  static propsType = {
    display: PropTypes.boolean.isRequired
  }

  render () {
    return this.props.display && (
      <div className="dropdown-menu">
        {/* ... */}
      </div>
    )
  }
}

class MyComponent extends Component {
  constructor (props) {
    super(props)
    this.state = {
      showMenu: false
    }
  }

  // ...

  showMenu () {
    this.setState({showMenu: true})
  }

  hideMenu () {
    this.setState({showMenu: false})
  }

  render () {
    return <DropDownMenu display={this.state.showMenu} />
  }
}
```

Read More: [lifting-state-up](https://facebook.github.io/react/docs/lifting-state-up.html)

**[⬆ Back to index](#index)**

<a name="container-componet" />

## Container component

Container only maintain state ,it don't have presentation logic(no div,class,style...etc).All container do is pass state and handler of state to presentation component by props.

The advantage of seperate container component and presentational component is when we change the presentation logic without consider the data change.It increase the modularization and reusability.

```javascript
// bad
class MessageList extends Component {
  constructor (props) {
    super(props)
  	this.state = {
        onlyUnread: false,
        messages: []
  	}
  }

  componentDidMount () {
    $.ajax({
      url: "/api/messages",
    }).then(({messages}) => this.setState({messages}))
  }

  handleClick = () => this.setState({onlyUnread: !this.state.onlyUnread})

  render () {
    return (
      <div class="message">
        <ul>
          {
            this.state.messages
              .filter(msg => this.state.onlyUnread ? !msg.asRead : true)
              .map(({content, author}) => {
                return <li>{content}—{author}</li>
              })
          }
        </ul>
        <button onClick={this.handleClick}>toggle unread</button>
      </div>
    )
  }
}
```

```javascript
// good
class MessageContainer extends Component {
  constructor (props) {
    super(props)
  	this.state = {
        onlyUnread: false,
        messages: []
  	}
  }

  componentDidMount () {
    $.ajax({
      url: "/api/messages",
    }).then(({messages}) => this.setState({messages}))
  }

  handleClick = () => this.setState({onlyUnread: !this.state.onlyUnread})

  render () {
    return <MessageList
      messages={this.state.messages.filter(msg => this.state.onlyUnread ? !msg.asRead : true)}
      toggleUnread={this.handleClick}
    />
  }
}

function MessageList ({messages, toggleUnread}) {
  return (
    <div class="message">
      <ul>
        {
          messages
            .map(({content, author}) => {
              return <li>{content}—{author}</li>
            })
        }
      </ul>
      <button onClick={toggleUnread}>toggle unread</button>
    </div>
  )
}
MessageList.propTypes = {
  messages: propTypes.array.isRequired,
  toggleUnread: propTypes.func.isRequired
}
```

Read More:

-   [Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.sz7z538t6)
-   [React AJAX Best Practices](http://andrewhfarmer.com/react-ajax-best-practices/)

**[⬆ Back to index](#index)**

<a name="pure-render"/>

## Render function must be pure

Render function should be pure function(also the stateless component),only rely on this.state,this.props.It also don't produce side effect to environment.

```javascript
// bad
render () {
  return <div>{window.navigator.userAgent}</div>
}

// good
render () {
  return <div>{this.props.userAgent}</div>
}
```

Read more:

-   [What is pure function](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-pure-function-d1c076bec976)

-   [Return as soon as you know the answer](https://medium.com/@SimonRadionov/return-as-soon-as-you-know-the-answer-dec6369b9b67#.q67w8z60g)

**[⬆ Back to index](#index)**

<a name="always-declare-proptypes" />

## Always declare propTypes

### Every component should declare proptype,not required props should provide default value(by defaultProps).

Adv:
Not if checking in render function for existence

```javascript
// bad
render () {
  if (this.props.person) {
    return <div>{this.props.person.firstName}</div>
  } else {
    return <div>Guest</div>
  }
}
```

```javascript
// good
class MyComponent extends Component {
  render() {
    return <div>{this.props.person.firstName}</div>
  }
}

MyComponent.defaultProps = {
  person: {
    firstName: 'Guest'
  }
}
```

### If the component has children,it should be mention in proptypes

Adv:
Component has children in normal usage.

### If the component has handler,it should be mention in proptypes

Adv:
Component can distribute action to change state.

Read More:

[Prop Validation](http://facebook.github.io/react/docs/reusable-components.html#prop-validation)

**[⬆ Back to index](#index)**

<a name="use-props-to-initial-state" />

## Use props to initial state

Props can be used to initial state only when props name say so.

```javascript
// bad
constructor (props) {
  this.state = {
    items: props.items
  }
}
```

```javascript
// good
constructor (props) {
  this.state = {
    items: props.initialItems
  }
}
```

["Props in getInitialState Is an Anti-Pattern"](http://facebook.github.io/react/tips/props-in-getInitialState-as-anti-pattern.html)

**[⬆ Back to index](#index)**

<a name="classnames" />

## classnames

use [classNames](https://www.npmjs.com/package/classnames) to generate classname.

```javascript
// bad
render () {
  return <div className={'menu ' + this.props.display ? 'active' : ''} />
}
```

```javascript
// good
render () {
  let classes = {
    menu: true,
    active: this.props.display
  }

  return <div className={classnames(classes)} />
}
```

Read: [Class Name Manipulation](https://github.com/JedWatson/classnames/blob/master/README.md)

**[⬆ Back to index](#index)**

<a name="miscellaneous" />

## Miscellaneous

### React class naming convention

Upper camel case

### CRUD Related Component

ElementCreate
ElementEdit
ElementList
ElementListItem
ElementDelete(the delete button)

### Variables

elements (array)
elementsByKey,e.g elementsById(collection by id field)
