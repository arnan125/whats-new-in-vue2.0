## Templates

### 片断实例 <sup>废弃</sup>

每一个组件都应仅有一个根节点元素，片段实例不再被允许。如果有如下代码片段

``` html
<p>foo</p>
<p>bar</p>
```

建议直接给它们整个内容添加一个包裹元素，如下所示

``` html
<div>
  <p>foo</p>
  <p>bar</p>
</div>
```

## 生命周期

### `beforeCompile` <sup>废弃</sup>

使用`created`钩子代替.

### `compiled` <sup>废弃</sup>

使用`mounted`钩子代替

### `attached` <sup>废弃</sup>

不再使用`attached`钩子，而是在其他钩子中做in-dom检测（不好翻译，意思大概是检测dom元素是否渲染完成），将

``` js
attached: function () {
    doSomething()
}
```

替换为

``` js
mounted: function () {
  this.$nextTick(function () {
    doSomething()
  })
}
```

### `detached` <sup>废弃</sup>

不再使用`detached`钩子，而是在其他钩子中做in-dom检测，将

``` js
detached: function () {
  doSomething()
}
```

替换为

``` js
destroyed: function () {
  this.$nextTick(function () {
    doSomething()
  })
}
```

### `init` <sup>废弃</sup>

使用`beforeCreate`代替，本质上，`init`和`beforeCreate`是一回事。为保持生命周期命名的一致性，`init`钩子被重命名为`beforeCreate`

### `ready` <sup>废弃</sup>
使用新的`mounted`钩子代替，注意，`mounted`钩子不保证组件元素已经渲染到文档中，因此，也需要调用`Vue.nextTick`/`vm.$nextTick`。如下所示

``` js
mounted: function () {
  this.$nextTick(function () {
    // code that assumes this.$el is in-document
  })
}
```

## `v-for`

### `v-for` 应用于数组时的参数顺序

当需要使用数组索引时，过去的参数是`(index, value)`，现在则是`(value, index)`，它与js原生数组方法的参数形式保持一致，比如`forEach`、`map`等

### `v-for` 应用于对象时的参数顺序

当需要使用对象的键（`key`）时，过去的参数是`(key, value)`，现在则是`(value, key)`，它与一些常用的对象迭代器参数形式保持一致，比如[lodash](http://lodashjs.com/docs/#_forinobject-iteratee_identity-thisarg)

### `$index` and `$key` <sup>废弃</sup>

隐式参量`$index`和`$key`已被废弃，应在`v-for`表达式中显示定义，这样可以使代码易于阅读（尤其对于vue新手），并且使嵌套循环代码的行为更清晰。

### `track-by` <sup>废弃</sup>

属性`track-by`被`key`代替，`key`更像其他普通属性，没有`v-bind:`或者`:`前缀时被当做是一个字符串值字面量。大多数情况下，你想要动态绑定`key`的话，请使用`v-bind:`或者`:`前缀，如下所示

``` html
<div v-for="item in items" track-by="id">
```

替换为

``` html
<div v-for="item in items" v-bind:key="item.id">
```

### `v-for` 应用于数值范围

先前`v-for="number in 10"`将会遍历0到9，现在则是从1到10。

## Props

### Prop验证选项`coerce`<sup>废弃</sup>

如果想要转换一个`prop`，请设置一个基于此`prop`的计算属性，如下所示

``` js
props: {
  username: {
    type: String,
    coerce: function (value) {
      return value
      .toLowerCase()
      .replace(/\s+/, '-')
    }
  }
}
```

替换为

``` js
props: {
 username: String,
},
computed: {
  normalizedUsername: function () {
    return this.username
    .toLowerCase()
    .replace(/\s+/, '-')
  }
}
```

几点好处如下：
- 仍然可以获得`prop`的原始值
- 显式定义转换后的`prop`名称，使得它和传入的`prop`有所区别

### Prop`twoWay`选项<sup>废弃</sup>

Props总是单向传递下来的。为了对父组件的作用域产生影响，组件应该显式地触发一个事件而非隐式的双向绑定数据，更多信息请查看

- [Custom component events](http://vuejs.org/guide/components.html#Custom-Events)

- [Custom input components](http://vuejs.org/guide/components.html#Form-Input-Components-using-Custom-Events) (using component events)

- [Global state management](http://vuejs.org/guide/state-management.html)

### 指令`v-bind`的修饰符`.once` and `.sync`<sup>废弃</sup>

Props总是单向传递下来的。为了对父组件的作用域产生影响，组件应该显式地触发一个事件而非隐式的双向绑定数据，更多信息请查看

- [Custom component events](http://vuejs.org/guide/components.html#Custom-Events)

- [Custom input components](http://vuejs.org/guide/components.html#Form-Input-Components-using-Custom-Events) (using component events)

- [Global state management](http://vuejs.org/guide/state-management.html)

### Prop变动 <sup>废弃</sup>

修改prop被认为是反模式（anti-pattern）,例如，申明一个prop并给他赋值`this.myProp = 'someOtherValue'`。基于更新机制，父组件一旦重新渲染，它的子组件的本地状态将会被覆写

大多数需要修改prop的场景都可以通过以下方式来替代实现：

- 设置一个data属性，用prop的值去初始化它（如使用修饰符once的场景）

- 设置一个计算属性（如使用prop coerce选项的场景）

### vue根组件上的Props<sup>废弃</sup>

在vue根组件实例上（例如，使用`new Vue({ ... })`实例化的组件），申明prop时应该使用`propsData`，而不是`props`

## 内建指令（`Directive`）

### `v-bind`与布尔值 

当使用`v-bind`时，只有`null`、`undefined`和`false`是假值（Falsiness），也就意味着`0`和空字符串会被解析为真值（truthy）。例如，`v-bind:draggable="''"` 会被解析为 as `draggable="true"`
对于枚举属性，除了上述假值，字符串 `"false"`会被解析为`attr="false"`，也为假值

<p class="tip">注意：对于其它指令 (如，`v-if`和`v-show`)，JavaScript本来的真假值依然生效</p>

### 用`v-on`监听组件的原生事件 

对于组件来说，`v-on`只能监听用由`$emit`触发的组件自定义事件，为了监听组件根节点元素的原生DOM事件，可以使用`.native`修饰符，例如

``` html

<my-component v-on:click.native="doSomething"></my-component>

```

### `v-model` 与 `debounce` <sup>废弃</sup>

debounce（去抖）一般被用来限制Ajax请求或其它高耗操作的执行频率。Vue的 `debounce`属性配合`v-model`指令可以在一些简单的场景很容易地实现去抖。但是，`debounce`实际上限制的是状态的更新频率而非高耗操作的执行频率，两者虽然差别细微，但是随着应用规模的增长，这种实现方式局限性越发显著。

使用`debounce`属性时，因为不能实时获取输入框的状态，将不能(准确)检测到输入状态。将去抖（debounce）从Vue中解耦，使得可以仅仅限制高耗操作本身的执行，而不会有其他的局限性。

下例是一个搜索提示器的实现

``` html
<!--
By using the debounce function from lodash or another dedicated
utility library, we know the specific debounce implementation we
use will be best-in-class - and we can use it ANYWHERE. Not just
in our template.
-->
<script src="https://cdn.jsdelivr.net/lodash/4.13.1/lodash.js"></script>
<div id="debounce-search-demo">
    <input v-model="searchQuery" placeholder="Type something">
    <strong>{{ searchIndicator }}</strong>
</div>
<script>
new Vue({
    el: '#debounce-search-demo',
    data: {
        searchQuery: '',
        searchQueryIsDirty: false,
        isCalculating: false
    },
    computed: {
        searchIndicator: function() {
            if (this.isCalculating) {
                return '⟳ Fetching new results'
            } else if (this.searchQueryIsDirty) {
                return '... Typing'
            } else {
                return '✓ Done'
            }
        }
    },
    watch: {
        searchQuery: function() {
            this.searchQueryIsDirty = true
            this.expensiveOperation()
        }
    },
    methods: {
        // This is where the debounce actually belongs.
        expensiveOperation: _.debounce(function() {
            this.isCalculating = true
            setTimeout(function() {
                this.isCalculating = false
                this.searchQueryIsDirty = false
            }.bind(this), 1000)
        }, 500)
    }
})
</script>

```
这种处理方式的另一个优点是可以自主选择节流/去抖包装函数。比如在提供搜索候选建议的时候，实际上更应该使用一个节流(throttle)函数而非去抖(debounce)函数，因为去抖总会在用户输入结束后一段时间才给出搜索建议，这是不太理想的。如果你使用了类似lodash的工具库，从`debounce`重构为`throttle`也是很方便的。

### `v-model`与`lazy`或`number`属性 <sup>废弃</sup>

现在`lazy`和`number`属性应作为修饰符配合`v-model`指令使用，这样更清晰一些

``` html

<input v-model="name" lazy>
<input v-model="age" type="number" number>

```

替换为

``` html

<input v-model.lazy="name">
<input v-model.number="age" type="number">

```

### `v-model`与行内`value`属性 <sup>废弃</sup>

`v-model`不再将value属性的值视为其初始值。为了更好的可预测性，它始终把vue实例作为数据源。

也就是说

``` html

<input v-model="text" value="foo">

```

数据基于vue实例

``` js

data: {

 text: 'bar'

}

```

输入框的值将会渲染为"bar"而非"foo"，给定初始值的`<textarea>`同理。

``` html

<textarea v-model="text">
 hello world
</textarea>

```

所以需要确保`text`的初始值是"hello world"


### `v-model`与`v-for`原始类型的遍历值 <sup>废弃</sup>

下诉的表达式不再有效：

``` html

<input v-for="str in strings" v-model="str">

```

因为`<input>`编译后等价的代码如下所示

``` js

strings.map(function (str) {

 return createElement('input', ...)

})

```
显而易见，`v-model`的双向绑定不再有意义，因为设置局部变量str`的值不会对strings有任何影响
应该使用一个元素为对象的数组，这样`v-model`就可以更新数组内每个元素的值了（因为传入的是对象的引用）。

``` html

<input v-for="obj in objects" v-model="obj.str">

```

### `v-bind:style`在对象语法中使用`!important` <sup>废弃</sup>

下面的表达式不再有效

``` html

<p v-bind:style="{ color: myColor + ' !important' }">hello</p>

```
如果你确实需要使用`!important`，可以这样：

``` html

<p v-bind:style="'color: ' + myColor + ' !important'">hello</p>

```

### `v-el` and `v-ref` <sup>废弃</sup>

为了更简洁，`v-el`和`v-ref`指令被合并到属性`ref`中，可以通过`$refs`在组件实例上获取。`v-el:my-element`变为`ref="myElement"`，`v-ref:my-component`变为`ref="myComponent"`。当在普通dom元素上使用时，ref将获取到dom元素，用于组件上时，将获取到组件实例。

由于`v-ref`不再是一个指令，只是一个特殊的属性，因此可以被动态定义，这在与`v-for`组合使用是尤其有用

``` html

<p v-for="item in items" v-bind:ref="'item' + item.id"></p>

```
先前，`v-el`/`v-ref`与`v-for`组合使用时，由于无法给每个条目不同的名称，会得到一个dom元素或者组件的数组。现在，仍然可以通过设置相同的ref来使用这个特性。

``` html

<p v-for="item in items" ref="items"></p>

```
和1.x版本不一样，`$refs`不再是响应式的，因为它们注册/更新依赖于渲染过程本身，要使它是响应式的，需要在每次改变时重复渲染。（不太理解，附上原文。Unlike in 1.x, these $refs are not reactive, because they’re registered/updated during the render process itself. Making them reactive would require duplicate renders for every change.）

另一方面，`$refs`本来是设计用来在代码里获取组件/dom元素的，因此不建议在模板里面过于依赖它，会引入不属于实例本身的状态，违反了数据驱动的view模型（不太理解，附上原文。On the other hand, `$refs` are designed primarily for programmatic access in JavaScript - it is not recommended to rely on them in templates, 
because that would mean referring to state that does not belong to the instance itself. This would violate Vue's data-driven view model.）

### `v-else`与`v-show` <sup>废弃</sup>

`v-else`不再配合`v-show`使用（作用于同一个dom元素）。请使用`v-if`表达式替代。示例如下，

``` html

<p v-if="foo">Foo</p>

<p v-else v-show="bar">Not foo, but bar</p>

```

替换为

``` html

<p v-if="foo">Foo</p>

<p v-if="!foo && bar">Not foo, but bar</p>

```

## 自定义指令

指令的应用场景被大幅削减，他们目前主要被用来底层（low-level）的dom操作。大多数情况下，开发者更应该倾向于使用组件来实现代码重用，而不是通过自定义指令（比如，弃用[`v-drapload`](https://github.com/jy03078959/vue-drapload)，使用[`<list-view></list-view>`](https://github.com/CatchLabs/vue-list-view)）。

一些需要特别之处的不同之处如下，
- 指令不再有实例，也就意味着不能在指令内部通过`this`访问到该指令的实例。指令几乎将所有需要的数据作为钩子函数（hook functions，）的参数（el，binding，vnode，oldVnode）接收，如果实在需要维护跨钩子的状态，可以使用`el`（唯一可写的的参数，可以操作它的`dataSet`）
- 高级选项，例如`acceptStatement`，`deep`，`priority`等都已废弃。
- 一些原有的钩子的行为发生了变化，另外还添加了一些新的钩子函数。

幸运的是，新的指令系统简单的多，你可以更容易地掌握它们。[自定义指令教程](http://vuejs.org/guide/custom-directive.html)

### 指令`.literal`修饰符 <sup>废弃</sup>

指令`.literal`修饰符被移除了，因为直接传入一个字符串字面量可以更容易地达到同样的效果

比如将

``` js

<p v-my-directive.literal="foo bar baz"></p>

```

替换为

``` html

<p v-my-directive="'foo bar baz'"></p>

```

## Transitions

### `transition`属性 <sup>废弃</sup>

Vue过渡效果（transition）体系完全改变了，现在使用`<transition>`和`<transition-group>`组件来作为包裹容器，而不是使用`transition`属性，了解更多的细节请阅读[Transitions教程](https://vuejs.org/guide/transitions.html)（内容异常丰富）。

### 使用`Vue.transition`定义可重用的Transitions <sup>废弃</sup>

在新的transition体系内，可以[使用组件来定义可重用的transition](http://rc.vuejs.org/guide/transitions.html#Reusable-Transitions)。

### Transition渐进过渡属性`stagger` <sup>废弃</sup>

如果需要实现列表的渐进过渡效果，可以给元素设置`data-index`或者类似的属性，然后在transiton钩子函数中获取它来控制过渡开始的时间（delay）[一个示例](https://vuejs.org/guide/transitions.html#Staggering-List-Transitions)。


## Events

### `events`配置 <sup>废弃</sup>

组件`events`配置项被废弃。事件处理器（Event handlers）应在`created`钩子内注册。查看后文**`$dispatch`和`$broadcast`**获取详细信息

### `Vue.directive('on').keyCodes` <sup>废弃</sup>

一个新的更简洁的`keyCodes`配置方式是`Vue.config.keyCodes`

``` js

// enable v-on:keyup.f1

Vue.config.keyCodes.f1 = 112 

```

### `$dispatch`和`$broadcast` <sup>废弃</sup>

`$dispatch`和`$broadcast`被废弃，更利于跨组件通信和以及引入可维护性更好的状态管理方案，比如[Vuex](https://github.com/vuejs/vuex)。

先前事件系统的问题在于事件流依赖于组件树的结构，当组件树的规模变得太大以后，事件流很难追溯，非常脆弱。这一切都是因为当初没有规划好，我们不想继续使你为此感到痛苦。另外`$dispatch`和`$broadcast`也不能进行同级组件间的通信。

这些事件方法最普遍的使用场景是父组件和它的直接子组件们通信。大多数情况下，你可以用`v-on`监听来自子组件的`$emit`行为（[listen to an `$emit` from a child with `v-on`](http://vuejs.org/guide/components.html#Form-Input-Components-using-Custom-Events)）。这样保持了事件的直观，便利。

但是，当进行隔代组件之间的通信时，`$emit`方法并不有效。我们会使用一个集中式的事件管理器（centralized event hub），它可以允许在组件树中的任何位置进行通信，甚至是同级组件间的通信。实现的思路之一：Vue的实例提供了一个事件触发器，因此你可以创建一个空vue实例来作为这个事件管理器。

比如，一个土豆（todo）app的结构如下 

```

Todos
|-- NewTodoInput
|-- Todo
|-- DeleteTodoButton

```

我们可以用一个事件管理器管理组件间的通信：

``` js

// This is the event hub we'll use in every
// component to communicate between them.

var eventHub = new Vue()

```

在组件中，我们可以分别使用`$emit`，`$on`，`$off`触发事件、监听事件、解绑事件

``` js

// NewTodoInput
// ...

methods: {
  addTodo: function () {
    eventHub.$emit('add-todo', { text: this.newTodoText })
    this.newTodoText = ''
  }
}

```

``` js

// DeleteTodoButton
// ...

methods: {
  deleteTodo: function (id) {
    eventHub.$emit('delete-todo', id)
  }
}

```

``` js

// Todos
// ...

created: function () {
  eventHub.$on('add-todo', this.addTodo)
  eventHub.$on('delete-todo', this.deleteTodo)
},

// It's good to clean up event listeners before
// a component is destroyed.

beforeDestroy: function () {
  eventHub.$off('add-todo', this.addTodo)
  eventHub.$off('delete-todo', this.deleteTodo)
},

methods: {
  addTodo: function (newTodo) {
    this.todos.push(newTodo)
  },
  deleteTodo: function (todoId) {
    this.todos = this.todos.filter(function (todo) {
      return todo.id !== todoId
    })
  }
}

```

这种模式可以在比较简单的场景下作为`$dispatch`和`$broadcast`的替代，在更复杂的场景下，建议使用专用的状态管理工具比如[Vuex](https://github.com/vuejs/vuex)。 

## Filters
### 文本插入以外的过滤器<sup>废弃</sup>

现在过滤器只能用于文本插入（`{{ }}`）标签内。先前将过滤器用于`v-model`、`v-on`等指令内造成的麻烦比带来的便利要多；在`v-for`指令中使用过滤器也不如在js中使用计算属性来得复用性好。

通常，如果某些功能可以通过纯js代码实现，就尽量避免引入特殊的语法比如过滤器去做相应处理。以下介绍如何来替换这些内建的过滤器。

#### 替代`debounce`过滤器

不再使用

``` html

<input v-on:keyup="doStuff | debounce 500">

```

``` js

methods: {
  doStuff: function () {
  // ...
  }
}

```

而是使用[lodash's `debounce`](https://lodash.com/docs/4.15.0#debounce)（或者可以节流[`throttle`](https://lodash.com/docs/4.15.0#throttle)的工具）直接限制高耗操作的调用.可以像下面这样实现上面的dobounce功能：

``` html

<input v-on:keyup="doStuff">

```

``` js

methods: {
  doStuff: _.debounce(function () {
  // ...
  }, 500)
}

```

了解这种实现方式的更多好处可以查看前文**`v-model` 与 `debounce`**

#### 替代`limitBy`过滤器

不再使用

``` html

<p v-for="item in items | limitBy 10">{{ item }}</p>

```

而是在一个计算属性里使用js原生[`.slice`方法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice#Examples):

``` html

<p v-for="item in filteredItems">{{ item }}</p>

```

``` js

computed: {
  filteredItems: function () {
    return this.items.slice(0, 10)
  }
}

```

#### 替代`filterBy`过滤器

不再使用

``` html

<p v-for="user in users | filterBy searchQuery in 'name'">{{ user.name }}</p>

```

而是在一个计算属性里使用js原生[`.filter`方法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter#Examples)：

``` html

<p v-for="user in filteredUsers">{{ user.name }}</p>

```

``` js

computed: {
  filteredUsers: function () {
    return this.users.filter(function (user) {
       return user.name.indexOf(this.searchQuery)
    })
  }
}

```

js的原生`.filter`方法可以在计算属性里处理更复杂的过滤操作，能发挥出js的全部的能力。例如过滤出所有激活且他们的邮箱或名字能大小写不敏感地匹配搜索关键字的用户。

``` js

this.users.filter(function (user) {
  var searchRegex = new RegExp(this.searchQuery, 'i')
  return user.isActive && (
    searchRegex.test(user.name) ||
    searchRegex.test(user.email)
  )
})

```

#### 替代`orderBy`过滤器

不再使用

``` html

<p v-for="user in users | orderBy 'name'">{{ user.name }}</p>

```

而是在一个计算属性里使用[lodash's `orderBy`](https://lodash.com/docs/4.15.0#orderBy)（或者类似的排序的工具）：

``` html

<p v-for="user in orderedUsers">{{ user.name }}</p>

```

``` js

computed: {
  orderedUsers: function () {
    return _.orderBy(this.users, 'name')
  }
}

```
甚至可以依据多条件排序：

``` js

_.orderBy(this.users, ['name', 'last_login'], ['asc', 'desc'])

```

### 过滤器参数语法

过滤器参数语法现在更类似于js的函数调用，先前用空格隔开参数的方式不再使用

``` html

<p>{{ date | formatDate 'YY-MM-DD' timeZone }}</p>

```

使用括号包裹所有参数，参数之间用逗号分隔：

``` html

<p>{{ date | formatDate('YY-MM-DD', timeZone) }}</p>

```

### 内建的文本过滤器<sup>废弃</sup>

虽然文本插入标签被还是允许使用过滤器的，但是这个内建的文本过滤器全被移除。建议使用专门的工具库实现日期格式装换（[`date-fns`](https://date-fns.org/)）或者货币处理（[`accounting`](http://openexchangerates.github.io/accounting.js/)）等

我们提供了每个原有内建文本过滤器的替换方式，可以将他们使用在自定义帮助函数中、方法、计算属性中。

#### 替代`json`过滤器

实际上，根本不需要将debug信息转成json字符串，vue自动会做这些事，无论是什么类型的数据。如果还是需要这种功能，可以在方法或者计算属性内使用`JSON.stringify`。

#### 替代`capitalize`过滤器

``` js

text[0].toUpperCase() + text.slice(1)

```

#### 替代`uppercase`过滤器

``` js

text.toUpperCase()

```

#### 替代`lowercase`过滤器

``` js

text.toLowerCase()

```

#### 替代`pluralize`过滤器

[pluralize](https://www.npmjs.com/package/pluralize)npm包可以很好的实现“复数化”这个功能。如果只是想要处理特定语汇的复数形式或者仅仅是处理输出为“零”的情况，可以自定义一个pluralize方法，例如：

``` js

function pluralizeKnife (count) {
  if (count === 0) {
    return 'no knives'
  } else if (count === 1) {
    return '1 knife'
  } else {
    return count + 'knives'
  }
}

```

#### Replacing the `currency` Filter

一个非常简单的实现如下
  
``` js

'$' + price.toFixed(2)

```

在很多情况下，`toFixed`方法会出现一些怪异行为，比如`0.035.toFixed(2)`向上舍入为0.4，但是`0.045.toFixed(2)`向下舍入为0.4。为了解决这个问题，可以使用[`accounting`](http://openexchangerates.github.io/accounting.js/)获得更可靠的货币格式化结果。

### 双向过滤器 <sup>废弃</sup>

一些用户喜欢配合`v-model`使用双向过滤器来实现一些有趣的输入，只需要很少的几行代码。但这只是一个看上去简单的方式，双向过滤器会造成大量不易察觉的复杂性，甚至因为状态更新的延迟带来糟糕的用户体验。建议使用组件包装`input`标签，这是一种更显式、更功能丰富的自定义输入组件解决方案。

下面是一个双向过滤器的迁移示例：

<iframe width="100%" height="300" src="https://jsfiddle.net/chrisvfritz/6744xnjk/embedded/js,html,result" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

在多数情况下，它都能正常工作，但是由于状态的延迟更新会造成一些怪异的行为。比如，切换到上面的demo到Result页面，在任意一个输入框中输入`9.999`，当输入框失焦以后，它的值会更新为`$10.00`，但是计算结果却是`9.999`，也就是说输入框实际存储值为`9.999`,实际数据和用户感知并不同步。

以下使用Vue2.0来实现一个更健壮的解决方案。首先用`<currency-input>`组件包裹这个双向过滤器：

<iframe width="100%" height="300" src="https://jsfiddle.net/chrisvfritz/943zfbsh/embedded/js,html,result" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

这样可以允许添加一些过滤器单独不能封装的功能，比如选择在输入框获得焦点以后选中全部文本。下一步是提取过滤器的业务逻辑。通过引入一个外部依赖[`currencyValidator`](https://gist.github.com/chrisvfritz/5f0a639590d6e648933416f90ba7ae4e)来实现所有操作：

<iframe width="100%" height="300" src="https://jsfiddle.net/chrisvfritz/9c32kev2/embedded/js,html,result" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

模块化的增强不仅使过滤器更易于向Vue2.0迁移，也让货币值解析和格式化可以：

- 独立于vue进行单元测试
- 用于应用的其他部分，比如api请求的参数校验。

将验证器抽取出来，利于更顺畅地构建一个健壮的解决方案。异常的状态都被排除，实际上用户不会不会输入任何错误内容。

最后一步，升级到Vue2.0

<iframe width="100%" height="300" src="https://jsfiddle.net/chrisvfritz/1oqjojjx/embedded/js,html,result" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

You may notice that:

- Every aspect of our input is more explicit, using lifecycle hooks and DOM events in place of the hidden behavior of two-way filters.

- We can now use `v-model` directly on our custom inputs, which is not only more consistent with normal inputs, but also means our component is Vuex-friendly.

- Since we're no longer using filter options that require a value to be returned, our currency work could actually be done asynchronously. That means if we had a lot of apps that had to work with currencies, we could easily refactor this logic into a shared microservice.



















## Slots

### Duplicate Slots <sup>废弃</sup>

It is no longer supported to have `<slot>`s with the same name in the same template. When a slot is rendered it is "used up" and cannot be rendered elsewhere in the same render tree. If you must render the same content in multiple places, pass that content as a prop.

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run your end-to-end test suite or app after upgrading and look for <strong>console warnings</strong> about duplicate slots <code>v-model</code>.</p>

</div>

### `slot` Attribute Styling <sup>废弃</sup>

Content inserted via named `<slot>` no longer preserves the `slot` attribute. Use a wrapper element to style them, or for advanced use cases, modify the inserted content programmatically using [render functions](render-function.html).

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find CSS selectors targeting named slots (e.g. <code>[slot="my-slot-name"]</code>).</p>

</div>

## Special Attributes

### `keep-alive` Attribute <sup>废弃</sup>

`keep-alive` is no longer a special attribute, but rather a wrapper component, similar to `<transition>`. For example:

``` html

<keep-alive>

 <component v-bind:is="view"></component>

</keep-alive>

```

This makes it possible to use `<keep-alive>` on multiple conditional children:

``` html

<keep-alive>

 <todo-list v-if="todos.length > 0"></todo-list>

 <no-todos-gif v-else></no-todos-gif>

</keep-alive>

```

<p class="tip">When `<keep-alive>` has multiple children, they should eventually evaluate to a single child. Any child other than the first one will simply be ignored.</p>

When used together with `<transition>`, make sure to nest it inside:

``` html

<transition>

 <keep-alive>

 <component v-bind:is="view"></component>

 </keep-alive>

</transition>

```

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find <code>keep-alive</code> attributes.</p>

</div>

## Interpolation

### Interpolation within Attributes <sup>废弃</sup>

Interpolation within attributes is no longer valid. For example:

``` html

<button class="btn btn-{{ size }}"></button>

```

Should either be updated to use an inline expression:

``` html

<button v-bind:class="'btn btn-' + size"></button>

```

Or a data/computed property:

``` html

<button v-bind:class="buttonClasses"></button>

```

``` js

computed: {

 buttonClasses: function () {

 return 'btn btn-' + size

 }

}

```

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of interpolation used within attributes.</p>

</div>

### HTML Interpolation <sup>废弃</sup>

HTML interpolations (`{{{ foo }}}`) have been deprecated in favor of the [`v-html` directive](/api/#v-html).

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find HTML interpolations.</p>

</div>

### One-Time Bindings <sup>废弃</sup>

One time bindings (`{{* foo }}`) have been deprecated in favor of the new [`v-once` directive](/api/#v-once).

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find one-time bindings.</p>

</div>

## Reactivity

### `vm.$watch`

Watchers created via `vm.$watch` are now fired before the associated component rerenders. This gives you the chance to further update state before the component rerender, thus avoiding unnecessary updates. For example, you can watch a component prop and update the component's own data when the prop changes.

If you were previously relying on `vm.$watch` to do something with the DOM after a component updates, you can instead do so in the `updated` lifecycle hook.

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run your end-to-end test suite, if you have one. The <strong>failed tests</strong> should alert to you to the fact that a watcher was relying on the old behavior.</p>

</div>

### `vm.$set`

The former `vm.$set` behavior has been deprecated and it is now just an alias for [`Vue.set`](/api/#Vue-set).

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of the deprecated usage.</p>

</div>

### `vm.$delete`

The former `vm.$delete` behavior has been deprecated and it is now just an alias for [`Vue.delete`](/api/#Vue-delete)

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of the deprecated usage.</p>

</div>

### `Array.prototype.$set` <sup>废弃</sup>

use Vue.set instead

(console error, migration helper)

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of <code>.$set</code> on an array. If you miss any, you should see <strong>console errors</strong> from the missing method.</p>

</div>

### `Array.prototype.$remove` <sup>废弃</sup>

Use `Array.prototype.splice` instead. For example:

``` js

methods: {

 removeTodo: function (todo) {

 var index = this.todos.indexOf(todo)

 this.todos.splice(index, 1)

 }

}

```

Or better yet, just pass removal methods an index:

``` js

methods: {

 removeTodo: function (index) {

 this.todos.splice(index, 1)

 }

}

```

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of <code>.$remove</code> on an array. If you miss any, you should see <strong>console errors</strong> from the missing method.</p>

</div>

### `Vue.set` and `Vue.delete` on Vue instances <sup>废弃</sup>

Vue.set and Vue.delete can no longer work on Vue instances. It is now mandatory to properly declare all top-level reactive properties in the data option. If you'd like to delete properties on a Vue instance or its `$data`, just set it to null.

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of <code>Vue.set</code> or <code>Vue.delete</code> on a Vue instance. If you miss any, they'll trigger <strong>console warnings</strong>.</p>

</div>

### Replacing `vm.$data` <sup>废弃</sup>

It is now prohibited to replace a component instance's root $data. This prevents some edge cases in the reactivity system and makes the component state more predictable (especially with type-checking systems).

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of overwriting <code>vm.$data</code>. If you miss any, <strong>console warnings</strong> will be emitted.</p>

</div>

### `vm.$get` <sup>废弃</sup>

Just retrieve reactive data directly.

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of <code>vm.$get</code>. If you miss any, you'll see <strong>console errors</strong>.</p>

</div>

## DOM-Focused Instance Methods

### `vm.$appendTo` <sup>废弃</sup>

Use the native DOM API:

``` js

myElement.appendChild(vm.$el)

```

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of <code>vm.$appendTo</code>. If you miss any, you'll see <strong>console errors</strong>.</p>

</div>

### `vm.$before` <sup>废弃</sup>

Use the native DOM API:

``` js

myElement.parentNode.insertBefore(vm.$el, myElement)

```

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of <code>vm.$before</code>. If you miss any, you'll see <strong>console errors</strong>.</p>

</div>

### `vm.$after` <sup>废弃</sup>

Use the native DOM API:

``` js

myElement.parentNode.insertBefore(vm.$el, myElement.nextSibling)

```

Or if `myElement` is the last child:

``` js

myElement.parentNode.appendChild(vm.$el)

```

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of <code>vm.$after</code>. If you miss any, you'll see <strong>console errors</strong>.</p>

</div>

### `vm.$remove` <sup>废弃</sup>

Use the native DOM API:

``` js

vm.$el.remove()

```

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of <code>vm.$remove</code>. If you miss any, you'll see <strong>console errors</strong>.</p>

</div>

## Meta Instance Methods

### `vm.$eval` <sup>废弃</sup>

No real use. If you do happen to rely on this feature somehow and aren't sure how to work around it, post on [the forum](http://forum.vuejs.org/) for ideas.

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of <code>vm.$eval</code>. If you miss any, you'll see <strong>console errors</strong>.</p>

</div>

### `vm.$interpolate` <sup>废弃</sup>

No real use. If you do happen to rely on this feature somehow and aren't sure how to work around it, post on [the forum](http://forum.vuejs.org/) for ideas.

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of <code>vm.$interpolate</code>. If you miss any, you'll see <strong>console errors</strong>.</p>

</div>

### `vm.$log` <sup>废弃</sup>

Use the [Vue Devtools](https://github.com/vuejs/vue-devtools) for the optimal debugging experience.

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of <code>vm.$log</code>. If you miss any, you'll see <strong>console errors</strong>.</p>

</div>

## Instance DOM Options

### `replace: false` <sup>废弃</sup>

Components now always replace the element they're bound to. To simulate the behavior of `replace: false`, you can wrap your root component with an element similar to the one you're replacing. For example:

``` js

new Vue({

 el: '#app',

 template: '<div id="app"> ... </div>'

})

```

Or with a render function:

``` js

new Vue({

 el: '#app',

 render: function (h) {

 h('div', {

 attrs: {

 id: 'app',

 }

 }, /* ... */)

 }

})

```

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of <code>replace: false</code>.</p>

</div>

## Global Config

### `Vue.config.debug` <sup>废弃</sup>

No longer necessary, since warnings come with stack traces by default now.

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of <code>Vue.config.debug</code>.</p>

</div>

### `Vue.config.async` <sup>废弃</sup>

Async is now required for rendering performance.

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of <code>Vue.config.async</code>.</p>

</div>

### `Vue.config.delimiters` <sup>废弃</sup>

This has been reworked as a [component-level option](/api/#delimiters). This allows you to use alternative delimiters within your app without breaking 3rd-party components.

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of <code>Vue.config.delimiters</code>.</p>

</div>

### `Vue.config.unsafeDelimiters` <sup>废弃</sup>

HTML interpolation has been [deprecated in favor of `v-html`](#HTML-Interpolation-deprecated).

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of <code>Vue.config.unsafeDelimiters</code>. After this, the helper will also find instances of HTML interpolation so that you can replace them with `v-html`.</p>

</div>

## Global API

### `Vue.extend` with `el` <sup>废弃</sup>

The el option can no longer be used in `Vue.extend`. It's only valid as an instance creation option.

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run your end-to-end test suite or app after upgrading and look for <strong>console warnings</strong> about the <code>el</code> option with <code>Vue.extend</code>.</p>

</div>

### `Vue.elementDirective` <sup>废弃</sup>

Use components instead.

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of <code>Vue.elementDirective</code>.</p>

</div>

### `Vue.partial` <sup>废弃</sup>

Partials have been deprecated in favor of more explicit data flow between components, using props. Unless you're using a partial in a performance-critical area, the recommendation is to simply use a [normal component](components.html) instead. If you were dynamically binding the `name` of a partial, you can use a [dynamic component](http://vuejs.org/guide/components.html#Dynamic-Components).

If you happen to be using partials in a performance-critical part of your app, then you should upgrade to [functional components](render-function.html#Functional-Components). They must be in a plain JS/JSX file (rather than in a `.vue` file) and are stateless and instanceless, just like partials. This makes rendering extremely fast.

A benefit of functional components over partials is that they can be much more dynamic, because they grant you access to the full power of JavaScript. There is a cost to this power however. If you've never used a component framework with render functions before, they may take a bit longer to learn.

<div class="upgrade-path">

 <h4>Upgrade Path</h4>

 <p>Run the <a href="https://github.com/vuejs/vue-migration-helper">migration helper</a> on your codebase to find examples of <code>Vue.partial</code>.</p>

</div>


