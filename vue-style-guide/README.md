## 优先级 A 的规则：必要的 (规避错误)



### 组件名为多个单词

**组件名应该始终是多个单词的，根组件 `App` 除外。**

#### 好例子

``` js
Vue.component('todo-item', {
  // ...
})
```

``` js
export default {
  name: 'TodoItem',
  // ...
}
```


### 组件数据

**组件的 `data` 必须是返回一个对象的函数。**


### Prop 定义

**Prop 定义应该尽量详细。**

在健壮的代码中, prop 定义应该始终尽可能详细，至少应该指定类型。


### 为 `v-for` 设置键值

**在组件上总是用 `key` 配合 `v-for`。**


### 为组件样式设置作用域

**对于应用来说，顶级 `App` 组件和布局组件中的样式可以是全局的，但是其它所有组件都应该是有作用域的。**

这条规则只和[单文件组件](../guide/single-file-components.html)有关。你_不一定_要使用 [`scoped` 特性](https://vue-loader.vuejs.org/zh-cn/features/scoped-css.html)。设置作用域也可以通过 [CSS Modules](https://vue-loader.vuejs.org/zh-cn/features/css-modules.html)，那是一个基于 class 的类似 [BEM](http://getbem.com/) 的策略，当然你也可以使用其它的库或约定。


**不管怎样，对于组件库，我们应该更倾向于选用基于 class 的策略而不是 `scoped` 特性。**

这让覆写内部样式更容易：使用了常人可理解的 class 名称且没有太高的选择器优先级，而且不太会导致冲突。


### 私有属性名

**在插件、混入等扩展中始终为自定义的私有属性使用 `$_` 前缀。并附带一个命名空间以回避和其它作者的冲突 (比如 `$_yourPluginName_`)。**


#### 好例子

``` js
var myGreatMixin = {
  // ...
  methods: {
    $_myGreatMixin_update: function () {
      // ...
    }
  }
}
```


## 优先级 B 的规则：强烈推荐 (增强可读性)



### 组件文件

**只要有能够拼接文件的构建系统，就把每个组件单独分成文件。**

#### 好例子
```
components/
|- TodoList.vue
|- TodoItem.vue
```


### 单文件组件文件的大小写

**[单文件组件](../guide/single-file-components.html)的文件名应该要么始终是单词大写开头 (PascalCase)，要么始终是横线连接 (kebab-case)。**

单词大写开头对于代码编辑器的自动补全最为友好，因为这使得我们在 JS(X) 和模板中引用组件的方式尽可能的一致。然而，混用文件命名方式有的时候会导致大小写不敏感的文件系统的问题，这也是横线连接命名同样完全可取的原因。

#### 好例子

```
components/
|- MyComponent.vue
```

```
components/
|- my-component.vue
```


### 基础组件名

**应用特定样式和约定的基础组件 (也就是展示类的、无逻辑的或无状态的组件) 应该全部以一个特定的前缀开头，比如 `Base`、`App` 或 `V`。**


### 单例组件名

**只应该拥有单个活跃实例的组件应该以 `The` 前缀命名，以示其唯一性。**

这不意味着组件只可用于一个单页面，而是_每个页面_只使用一次。这些组件永远不接受任何 prop，因为它们是为你的应用定制的，而不是它们在你的应用中的上下文。如果你发现有必要添加 prop，那就表明这实际上是一个可复用的组件，_只是目前_在每个页面里只使用一次。


### 紧密耦合的组件名

**和父组件紧密耦合的子组件应该以父组件名作为前缀命名。**

如果一个组件只在某个父组件的场景下有意义，这层关系应该体现在其名字上。因为编辑器通常会按字母顺序组织文件，所以这样做可以把相关联的文件排在一起。

#### 好例子
```
components/
|- TodoList.vue
|- TodoListItem.vue
|- TodoListItemButton.vue
```


### 组件名中的单词顺序

**组件名应该以高级别的 (通常是一般化描述的) 单词开头，以描述性的修饰词结尾。**

你可能想换成多级目录的方式，把所有的搜索组件放到“search”目录，把所有的设置组件放到“settings”目录。我们只推荐在非常大型 (如有 100+ 个组件) 的应用下才考虑这么做，因为：

- 在多级目录间找来找去，要比在单个 `components` 目录下滚动查找要花费更多的精力。
- 存在组件重名 (比如存在多个 `ButtonDelete` 组件) 的时候在编辑器里更难快速定位。
- 让重构变得更难，因为为一个移动了的组件更新相关引用时，查找/替换通常并不高效。

#### 好例子
```
components/
|- SearchButtonClear.vue
|- SearchButtonRun.vue
|- SearchInputQuery.vue
|- SearchInputExcludeGlob.vue
|- SettingsCheckboxTerms.vue
|- SettingsCheckboxLaunchOnStartup.vue
```


### 自闭合组件

**在[单文件组件](../guide/single-file-components.html)、字符串模板和 [JSX](../guide/render-function.html#JSX) 中没有内容的组件应该是自闭合的——但在 DOM 模板里永远不要这样做。**


### 模板中的组件名大小写 <sup data-p="b">强烈推荐</sup>

**在[单文件组件](../guide/single-file-components.html)和字符串模板中组件名应该总是 PascalCase 的。但是在 DOM 模板中总是 kebab-case 的。**

PascalCase 相比 kebab-case 有一些优势：

- 编辑器可以在模板里自动补全组件名，因为 PascalCase 同样适用于 JavaScript。
- `<MyComponent>` 视觉上比 `<my-component>` 更能够和单个单词的 HTML 元素区别开来，因为前者的不同之处有两个大写字母，后者只有一个横线。
- 如果你在模板中使用任何非 Vue 的自定义元素，比如一个 Web Component，PascalCase 确保了你的 Vue 组件在视觉上仍然是易识别的。

不幸的是，由于 HTML 是大小写不敏感的，在 DOM 模板中必须仍使用 kebab-case。


### JS/JSX 中的组件名大小写

**JS/[JSX](../guide/render-function.html#JSX) 中的组件名应该始终是 PascalCase 的，尽管在较为简单的应用中只使用 `Vue.component` 进行全局组件注册时，可以使用 kebab-case 字符串。**


### 完整单词的组件名

**组件名应该倾向于完整单词而不是缩写。**


### Prop 名大小写

**在声明 prop 的时候，其命名应该始终使用 camelCase，而在模板和 [JSX](../guide/render-function.html#JSX) 中应该始终使用 kebab-case。**


### 多个特性的元素

**多个特性的元素应该分多行撰写，每个特性一行。**


### 模板中的复杂表达式

**组件模板应该仅包括简单表达式 , 更加复杂的表达式 应该重构为计算属性或方法。**


### 复杂计算属性 <sup data-p="b">强烈推荐</sup>

**复杂计算属性应该尽可能被拆分为许多简单的属性。**


### 引用属性值

**非空 HTML 属性值应始终包含在引号中(单引或双引，选择不在JS中使用的).**


### 指令缩写

**始终使用或者始终不适用指令缩写.**

## 优先级 C 的规则: 推荐 (Minimizing Arbitrary Choices and Cognitive Overhead)



### 组件/实例 选项顺序

**组件/实例 选项顺序 应该始终保持一致的排序.**

This is the default order we recommend for component options. They're split into categories, so you'll know where to add new properties from plugins.

1. **Side Effects** (triggers effects outside the component)
  - `el`

2. **Global Awareness** (requires knowledge beyond the component)
  - `name`
  - `parent`

3. **Component Type** (changes the type of the component)
  - `functional`

4. **Template Modifiers** (changes the way templates are compiled)
  - `delimiters`
  - `comments`

5. **Template Dependencies** (assets used in the template)
  - `components`
  - `directives`
  - `filters`

6. **Composition** (merges properties into the options)
  - `extends`
  - `mixins`

7. **Interface** (the interface to the component)
  - `inheritAttrs`
  - `model`
  - `props`/`propsData`

8. **Local State** (local reactive properties)
  - `data`
  - `computed`

9. **Events** (callbacks triggered by reactive events)
  - `watch`
  - Lifecycle Events (in the order they are called)

10. **Non-Reactive Properties** (instance properties independent of the reactivity system)
  - `methods`

11. **Rendering** (the declarative description of the component output)
  - `template`/`render`
  - `renderError`



### 元素属性顺序

**元素属性(包括组件) 应该保持一致的排序.**

This is the default order we recommend for component options. They're split into categories, so you'll know where to add custom attributes and directives.

1. **Definition** (provides the component options)
  - `is`

2. **List Rendering** (creates multiple variations of the same element)
  - `v-for`

2. **Conditionals** (whether the element is rendered/shown)
  - `v-if`
  - `v-else-if`
  - `v-else`
  - `v-show`
  - `v-cloak`

3. **Render Modifiers** (changes the way the element renders)
  - `v-pre`
  - `v-once`

4. **Global Awareness** (requires knowledge beyond the component)
  - `id`

5. **Unique Attributes** (attributes that require unique values)
  - `ref`
  - `key`
  - `slot`

6. **Two-Way Binding** (combining binding and events)
  - `v-model`

7. **Other Attributes** (all unspecified bound & unbound attributes)

8. **Events** (component event listeners)
  - `v-on`

9. **Content** (overrides the content of the element)
  - `v-html`
  - `v-text`



### 组件/实例选项中的空行  <sup data-p="c">recommended</sup>

**你可能想要在多行属性间增加空行, 尤其是如果选项在不滚动时不再适配你的屏幕。**



### 单文件组件顶层元素排序

**[单文件组件](../guide/single-file-components.html) 应该始终保持`template`, `script`, and `style` 标签排序一致 ，  `<style>` 在最后, 因为其余两种中总是至少存在一个。**


## 优先级 D 的规则: Use with Caution (Potentially Dangerous Patterns)



### `v-if`/`v-if-else`/`v-else` without `key`

**通常最好将`key` 和 `v-if` + `v-else` 同时使用, 如果它们是同种元素类型 (e.g. both `<div>` elements).**


### 带有 `scoped` 的元素选择器

**元素选择器应该避免依赖 `scoped`.**

在`scoped` 样式中比起元素选择器更倾向于类选择器, 因为大量的元素选择器很慢.


### 父子组件通信

**父子组件通信更倾向于 Props and events , 代替 `this.$parent` 或 mutating props.**


### 全局状态管理

**[Vuex](https://github.com/vuejs/vuex) should be preferred for 全局状态管理, instead of `this.$root` or a global event bus.**
