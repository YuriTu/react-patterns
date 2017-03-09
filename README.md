[译注]React最佳实践
=====

原文地址：[react-patterns](https://github.com/planningcenter/react-patterns)


----------


这篇文章主要讲的是在React组件开发中的一些规范，我也将其实践于项目中，结合我在项目中的实践进行了一些批注，意欲解释其为什么要这么做，希望能给新开发react的小伙伴有一些帮助。
----------


*React最佳实践*

## 目录

1. [概述](#概述)
1. 组件的组织结构
  1. [基础组件结构](#基础组件结构)
  1. [规范props格式](#规范props格式)
1. 推荐模式
  1. [Props数据的处理](#props数据的处理)
  1. [State数据的处理](#state数据的处理)
  1. [在render方法中使用三元表达式](#在render方法中使用三元表达式)
  1. [View层组件](#view层组件)
  1. [容器组件](#容器组件)
1. 反模式
  1. [条件判断语句](#条件判断语句)
  1. [不要在 render 方法中缓存数据](#不要在render方法中缓存数据)
  1. [存在性检验](#存在性检验)
  1. [根据props来设置state](#根据props来设置state)
1. 实例
  1. [合理命名事件处理函数](#合理命名事件处理函数)
  1. [对event合理命名](#对event合理命名)
  1. [使用propType](#使用proptype)
  1. [使用实体字符](#使用实体字符)
1. 坑
  1. [表格](#表格)
1. 工具库
  1. [classNames库](#classnames库)
1. 其他
  1. [JSX](#jsx)
  1. [ES2015](#es2015)
  1. [react-rails](#react-rails)
  1. [rails-assets](#rails-assets)
  1. [flux](#flux)

---

## 概述
本文讨论我们如何正确的在Rails框架中编写[React.js](https://facebook.github.io/react/)。我们一直在努力寻找最优雅的方式去编写React组件，在多次的失败尝试之后，我们得出了以下这些所推荐的方式；如果有的地方看起来不太适合，那么他可能真的不合适。希望你能让我们也知道哪些地方不合适。

所有的例子基于ES2015语法编写，使用 [babel](http://babeljs.io/)转译以及采用[react-rails gem](https://github.com/reactjs/react-rails)框架。

**[↑ 返回](#目录)**

---

## 基础组件结构

* class的定义
  * constructor
    * 事件绑定
  * 组件生命周期事件
  * get/set 数据处理函数
  * render 方法
* defaultProps
* proptypes

```javascript
class Person extends React.Component {
  constructor (props) {
    super(props);

    this.state = { smiling: false };

    this.handleClick = () => {
      this.setState({smiling: !this.state.smiling});
    };
  }

  componentWillMount () {
    // 事件监听 （flux/reduce 的Store、WebSocket、document等） 
  }

  componentDidMount () {
    // React.getDOMNode() DOM操作等相关逻辑
  }

  componentWillUnmount () {
    // 移除事件监听 
  }

  get smilingMessage () {
    return (this.state.smiling) ? "is smiling" : "";
  }

  render () {
    return (
      <div onClick={this.handleClick}>
        {this.props.name} {this.smilingMessage}
      </div>
    );
  }
}

Person.defaultProps = {
  name: 'Guest'
};

Person.propTypes = {
  name: React.PropTypes.string
};
```

> 译注：
在willMount 生命周期中virtualDom已经完成，所以可以进行事件绑定
在DidMount 中组件已经被渲染，此时可以用this.refs了，写canvas的同学要注意这点。


**[↑ 返回](#目录)**

## 规范Props格式 
两个及以上props换行书写。

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

**[↑ 返回](#目录)**

---

## Props数据的处理
使用 Get/Set访问器属性 来做数据处理。

```javascript
  // bad
  firstAndLastName () {
    return `${this.props.firstName} ${this.props.lastname}`;
  }

  // good
  get fullName () {
    return `${this.props.firstName} ${this.props.lastname}`;
  }
```
参阅：反模式- [不要在 render 方法中缓存数据](#不要在render方法中缓存数据)

**[↑ 返回](#目录)**

---

## State数据的处理
数据处理函数的命名应该在开头处添加一个动词，以增加可读性。

```javascript
// bad
happyAndKnowsIt () {
  return this.state.happy && this.state.knowsIt;
}
```

```javascript
// good
get isHappyAndKnowsIt () {
  return this.state.happy && this.state.knowsIt;
}
```
这里的对象方法*必须*返回一个`布尔值`。

> 译注：
这里想表达的是，state做为管理组件状态的数据，数据结构应该尽量简单，清晰的表示组件状态，复杂的业务逻辑应该放在父组件通过props传递的方式来获得，所以作者推荐state的处理必须返回布尔值。

参阅: 反模式-[条件判断语句](#条件判断语句)

**[↑ 返回](#目录)**

## 在render方法中使用三元表达式

在`render`中使用三元表达式来完成渲染判断。

```javascript
// bad
renderSmilingStatement () {
  return <strong>{(this.state.isSmiling) ? " is smiling." : ""}</strong>;
},

render () {
  return <div>{this.props.name}{this.renderSmilingStatement()}</div>;
}
```

```javascript
// good
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

**[↑ 返回](#目录)**

## View层组件
以view层为基本最小单元来组织组件。不要创建只能一次性使用的巨无霸组件以及域组件 。


```javascript
// bad
class PeopleWrappedInBSRow extends React.Component {
  render () {
    return (
      <div className="row">
        <People people={this.state.people} />
      </div>
    );
  }
}
```

```javascript
// good
class BSRow extends React.Component {
  render () {
    return <div className="row">{this.props.children}</div>;
  }
}

class SomeView extends React.Component {
  render () {
    return (
      <BSRow>
        <People people={this.state.people} />
      </BSRow>
    );
  }
}
```

**[↑ 返回](#目录)**

## 容器组件

> 使用一个父级容器来进行数据请求，然后用子级组件来进行对应的渲染 &mdash; Jason Bonta

#### Bad

```javascript
// CommentList.js

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

#### Good

```javascript
// CommentList.js

class CommentList extends React.Component {
  render() {
    return (
      <ul>
        {this.props.comments.map(({body, author}) => {
          return <li>{body}—{author}</li>;
        })}
      </ul>
    );
  }
}
```

```javascript
// CommentListContainer.js

class CommentListContainer extends React.Component {
  getInitialState () {
    return { comments: [] }
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
    return <CommentList comments={this.state.comments} />;
  }
}
```

[了解更多](https://medium.com/@learnreact/container-components-c0e67432e005)  
[观看相关视频](https://www.youtube.com/watch?v=KYzlpRvWZ6c&t=1351)

**[↑ 返回](#目录)**

---

## 不要在render方法中缓存数据

不要在 `render`方法中进行数据处理、state/props的保存。

```javascript
// bad
render () {
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
get fancyName () {
  return `Mrs. ${this.props.name}`;
}

render () {
  return <div>{this.fancyName}</div>;
}
```
*我认为这样的风格是为了性能考量。*


参阅：推荐模式- [props数据的处理](#props数据的处理)

**[↑ 返回](#目录)**

## 条件判断语句

不要在 `render`方法中写判断语句，多采用三元或短路。

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
这个情况最好的方法就是使用 [父组件容器](#容器组件)来管理state，新的state数据作为props传给子组件。

参阅: 推荐模式- [State数据的处理](#state数据的处理)

**[↑ 返回](#目录)**

## 存在性检验
不要在根组件检查存在性。
使用函数式编写无状态组件时，返回的数据类型应该恒定。


```javascript
// bad
const Person = props => {
//原文为this.props.firstName 为原作者笔误，故改正
  if (props.firstName)
    return <div>{props.firstName}</div>
  else
    return null
}
```
所有组件都*必须*被渲染。
所以可以增加一个自己觉得合适的默认值——`defaultProps`。

> 译注：
所有组件都进行渲染主要是为了方便进行控制和管理。
即不需要呈现的组件，渲染为一个空标签即可。

```javascript
// better
const Person = props =>
//原文为this.props.firstName 为作者笔误，故改正
  <div>{props.firstName}</div>

Person.defaultProps = {
  firstName: "Guest"
}
```
如果一个组件通过条件判断来决定渲染与否，那么可以将存在性检验写在其私有的组件语句中。

```javascript
// best
const TheOwnerComponent = props =>
  <div>
    {props.person && <Person {...props.person} />}
  </div>
```
上述的存在性检验主要用于对象或者数组。
组件所需要的props中，嵌套的数据类型使用 PropTypes来进行检测。

**[↑ 返回](#目录)**

## 根据props来设置state
根据props设置state，并且给予props明确的含义。

```javascript
// bad
getInitialState () {
  return {
    items: this.props.items
  };
}
```

```javascript
// good
getInitialState () {
  return {
    items: this.props.initialItems
  };
}
```

了解更多: ["Props in getInitialState Is an Anti-Pattern"](http://facebook.github.io/react/tips/props-in-getInitialState-as-anti-pattern.html)

**[↑ 返回](#目录)**

---

## 合理命名事件处理函数
为了让你给事件处理函数起个有意义的名字，先写事件触发的业务逻辑，之后再进行事件命名。

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


处理函数的命名应该:

- 以`handle`作为开始
- 以其对应的事件作为名字的结束 (例如, `Click`, `Change`)
- 使用一般现在时

如果你需要一些消除歧义的名称，你可以在`handle` 和事件名中间增加一些补充信息，
比如说你想要区分`onChange`事件，就可以是：`handleNameChange`和 `handleAgeChange`。不过若真如此，你可能需要想一下是不是需要创造一个新的组件。


**[↑ 返回](#目录)**


## 对event合理命名
对于所有者自身的事件使用自定义事件来命名。

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

**[↑ 返回](#目录)**

## 使用propType
proptype表示组件所期望的数据类型，以及用于产生有意义的警告。

```javascript
MyValidatedComponent.propTypes = {
  name: React.PropTypes.string
};
```
如果接受到的`name`而不是`string`，`MyValidatedComponent` 会打出一个警告。


```html
<Person name=1337 />
// Warning: Invalid prop `name` of type `number` supplied to `MyValidatedComponent`, expected `string`.
```
这个组件要求 所传入的`props`必须有 name 字段。
```javascript
MyValidatedComponent.propTypes = {
  name: React.PropTypes.string.isRequired
}
```
组件会对要求字段进行验证。
```html
<Person />
// Warning: Required prop `name` was not specified in `Person`
```

了解更多: [Prop Validation](http://facebook.github.io/react/docs/reusable-components.html#prop-validation)

**[↑ 返回](#目录)**

## 使用实体字符
使用React原生的 `String.fromCharCode()` 来处理特殊字符。

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

了解更多: [JSX Gotchas](http://facebook.github.io/react/docs/jsx-gotchas.html#html-entities)

**[↑ 返回](#目录)**

## 表格
在`table`标签中要记得使用`tbody`。

如果你忘记了`tbody`标签，虽然React不会像浏览器那样觉得你很ZZ然后帮你加一个，但是React会继续渲染JSX即给`table`里面直接插入`tr`，最后的结果可能不可控，使你一脸懵逼，所以记得使用 `tbody`。 

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


**[↑ 返回](#目录)**

## classNames库

使用 [classNames](https://www.npmjs.com/package/classnames)这个库来做类名判断与管理。

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

了解更多: [Class Name Manipulation](https://github.com/JedWatson/classnames/blob/master/README.md)

**[↑ 返回](#目录)**

## JSX
我在的项目组中曾经有一些 CoffeeScript的拥趸，不幸的事情是在写CoffeeScript数据模板的时候，因为JSX进行了抽象导致模板钩子的实现改变了。

所以我们不推荐使用CoffeeScript来编写render方法。

当必须使用CoffeeScript时，你可以看一下我们怎么使用CoffeeScript： [CoffeeScript and JSX](https://slack-files.com/T024L9M0Y-F02HP4JM3-80d714).


**[↑ 返回](#目录)**

## ES2015
[react-rails](https://github.com/reactjs/react-rails)现在已经集成了[babel](https://babeljs.io)，所有babel能做的事情，在Rails中也可以，你可以参阅文档来了解其相关配置。

**[↑ 返回](#目录)**

## react-rails
[react-rails](https://github.com/reactjs/react-rails)可以用于所有使用React开发的Rails Apps。它使得Rails与React之间可以完美结合。

**[↑ 返回](#目录)**

## rails-assets
[rails-assets](https://rails-assets.org)是一个资源管理框架，用于管理项目中的js/css等静态资源。我这里最流行的React库是[Bower](http://bower.io)它其可以非常方便的添加依赖与react资源

**警告：rails-assets 需要 Sprockets来进行 bower项目的访问。这是rails使用js传统的方法的胜利，这种方法不需要借助js模块化管理工具。**


**[↑ 返回](#目录)**

## flux
使用[Alt](http://alt.js.org)来进行flux开发。flux 文档提到建议配合Alt开发。

**[↑ 返回](#目录)**
