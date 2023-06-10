# 快速课程

让我们直接进入它！让我们通过构建一个简单的 Todo 应用程序并编写测试来学习 Vue Test Utils（VTU）。本指南将介绍如何：

- 挂载组件
- 查找元素
- 填充表单
- 触发事件

## 开始吧

我们将从一个简单的 `TodoApp` 组件开始，其中包含单个待办事项：

```vue
<template>
  <div></div>
</template>

<script>
export default {
  name: 'TodoApp',

  data() {
    return {
      todos: [
        {
          id: 1,
          text: 'Learn Vue.js 3',
          completed: false
        }
      ]
    }
  }
}
</script>
```

## 第一个测试 - 呈现待办事项

我们要写的第一个测试要验证一个待办被渲染了。让我们先看看这个测试，然后逐个谈论：

```js
import { mount } from '@vue/test-utils'
import TodoApp from './TodoApp.vue'

test('renders a todo', () => {
  const wrapper = mount(TodoApp)

  const todo = wrapper.get('[data-test="todo"]')

  expect(todo.text()).toBe('Learn Vue.js 3')
})
```

首先我们引入 `mount` - 这是在 VTU 里用于渲染组件的主要方式。你通过 `test` 函数加上这个测试的简短描述声明一个测试。 `test` 和 `expect` 函数都是在大部分测试运行器的全局变量 (该示例使用 [Jest](https://jestjs.io/en/)). 如果 `test` 和 `expect` 看起来不太明白，在 Jest 的文档有 [更多简单的示例](https://jestjs.io/docs/en/getting-started) 说明如何使用它们以及它们是如何工作的。

接下来，我们调用 `mount` 并且传递一个组件作为第一个参数 - 这些几乎是你写每个测试都要做的。 通常，我们将这个的结果命名为变量 `wrapper`，于是 `mount` 提供了一个简单的包裹 app 的 "包装器"，它上面有很多用于测试的便捷方法。

最后，我们使用另一个许多测试运行器上常见的全局函数 - Jest 上就有 - `expect`。 意思是我们断定，或者说 _期望_ , 真实的输出与我们想要的结果相匹配。在这个用例中，我们要查找在 DOM 中，一个选择器是 `data-test="todo"` 的元素，它看起来会像这样的 `<div data-test="todo">...</div>`。 我们然后调用 `text` 方法来获取内容， 我们期望它的值是 `'Learn Vue.js 3'`。

> 并不一定要实用 `data-test` 选择器，不过它可以使得你的测试不那么脆弱。类名和 ID 选择器会发生改变或者随着应用的迭代发生移动 - 通过使用 `data-test`，别的开发者会清楚哪个元素是用于测试的，就不应该被改变。

## 让测试通过

如果我们现在运行测试，会失败并且有这样的报错信息： `Unable to get [data-test="todo"]`。这是因为我们现在没有渲染任何待办元素，所以 调用 `get()` 来返回一个包装器（记住，VTU 包装所有的组件，以及 DOM 元素在一个有一些便捷方法的“包装器”里）会失败。让我们更新 `TodoApp.vue` 里的 `<template>` 来渲染 `todos` 数组：

```vue
<template>
  <div>
    <div v-for="todo in todos" :key="todo.id" data-test="todo">
      {{ todo.text }}
    </div>
  </div>
</template>
```

这样一改，测试就通过了。恭喜你，你写完了你的第一个组件测试。

## 添加一个新待办

我们要添加的下一个功能是让用户能够创建新的待办事项。要实现这个，我们需要一个表单，它有一个输入框给用户输入一些文字。当用户提交表单，我们希望新的待办事项可以呈现。让我们看一看测试：

```js
import { mount } from '@vue/test-utils'
import TodoApp from './TodoApp.vue'

test('creates a todo', () => {
  const wrapper = mount(TodoApp)
  expect(wrapper.findAll('[data-test="todo"]')).toHaveLength(1)

  wrapper.get('[data-test="new-todo"]').setValue('New todo')
  wrapper.get('[data-test="form"]').trigger('submit')

  expect(wrapper.findAll('[data-test="todo"]')).toHaveLength(2)
})
```

与往常一样，我们一开始使用 `mount` 来渲染元素。我们还断言只渲染了一个待办事项 -  to render the element. We are also asserting that only 1 todo is rendered - this makes it clear that we are adding an additional todo, as the final line of the test suggests.

我们使用 `setValue` 来更新 `<input>` - 这个可以让我们设置输入框 的值。

更新 `<input>` 之后，我们使用 `trigger` 方法模拟用户提交表单。最终，我们断言待办事项的数量从 1 增长到了 2。

如果我们运行这个测试，很明显他会失败。让我们更新 `TodoApp.vue` 让它有 `<form>` 和 `<input>` 元素，使测试通过

```vue
<template>
  <div>
    <div v-for="todo in todos" :key="todo.id" data-test="todo">
      {{ todo.text }}
    </div>

    <form data-test="form" @submit.prevent="createTodo">
      <input data-test="new-todo" v-model="newTodo" />
    </form>
  </div>
</template>

<script>
export default {
  name: 'TodoApp',

  data() {
    return {
      newTodo: '',
      todos: [
        {
          id: 1,
          text: 'Learn Vue.js 3',
          completed: false
        }
      ]
    }
  },

  methods: {
    createTodo() {
      this.todos.push({
        id: 2,
        text: this.newTodo,
        completed: false
      })
    }
  }
}
</script>
```

我们正使用 `v-model` 来绑定 `<input>` 以及 `@submit` 来监听表单提交。当表单提交了，`createTodo` 被调用并且插入一个新的待办项到 `todos` 数组。

看起来不错，运行显示一个报错：

```
expect(received).toHaveLength(expected)

    Expected length: 2
    Received length: 1
    Received array:  [{"element": <div data-test="todo">Learn Vue.js 3</div>}]
```

待办项的数量没有增加。问题在于 Jest 以同步方式执行测试，
The number of todos has not increased. The problem is that Jest executes tests in a synchronous manner, ending the test as soon as the final function is called. Vue, however, updates the DOM asynchronously. We need to mark the test `async`, and call `await` on any methods that might cause the DOM to change. `trigger` is one such methods, and so is `setValue` - we can simply prepend `await` and the test should work as expected:

```js
import { mount } from '@vue/test-utils'
import TodoApp from './TodoApp.vue'

test('creates a todo', async () => {
  const wrapper = mount(TodoApp)

  await wrapper.get('[data-test="new-todo"]').setValue('New todo')
  await wrapper.get('[data-test="form"]').trigger('submit')

  expect(wrapper.findAll('[data-test="todo"]')).toHaveLength(2)
})
```

现在测试最终通过了！

## 完善待办事项

Now that we can create todos, let's give the user the ability to mark a todo item as completed/uncompleted with a checkbox. As previously, let's start with the failing test:
现在我们可以创建待办了，让我们给用户一个能标记待办事项完成/未完成的复选框。像之前那样，我们从失败的测试开始：

```js
import { mount } from '@vue/test-utils'
import TodoApp from './TodoApp.vue'

test('completes a todo', async () => {
  const wrapper = mount(TodoApp)

  await wrapper.get('[data-test="todo-checkbox"]').setValue(true)

  expect(wrapper.get('[data-test="todo"]').classes()).toContain('completed')
})
```

This test is similar to the previous two; we find an element and interact with it in same way (we use `setValue` again, since we are interacting with a `<input>`).
这个测试和之前两个测试相同；我们以相同的方式查找元素并且与之交互（我们再次使用`setValue`，因为我们正在与 `<input>` 进行交互）。

接下来，我们做一个断言。我们将会给已完成的待办项应用一个 `completed` 类名 - 我们然后可以使用这个去添加一些样式来可视化的表示待办的状态。

我们可以通过更新 `<template>` 让它包含 `<input type="checkbox">` 并且有一个绑定在待办元素上的类，使这个测试通过：

```vue
<template>
  <div>
    <div
      v-for="todo in todos"
      :key="todo.id"
      data-test="todo"
      :class="[todo.completed ? 'completed' : '']"
    >
      {{ todo.text }}
      <input
        type="checkbox"
        v-model="todo.completed"
        data-test="todo-checkbox"
      />
    </div>

    <form data-test="form" @submit.prevent="createTodo">
      <input data-test="new-todo" v-model="newTodo" />
    </form>
  </div>
</template>
```

恭喜你！ 你编写了你的第一个组件测试。

## 编排，执行，断言

你可能注意到在每个测试的代码之间有一些换行。让我们再仔细看一遍第二个测试

```js
import { mount } from '@vue/test-utils'
import TodoApp from './TodoApp.vue'

test('creates a todo', async () => {
  const wrapper = mount(TodoApp)

  await wrapper.get('[data-test="new-todo"]').setValue('New todo')
  await wrapper.get('[data-test="form"]').trigger('submit')

  expect(wrapper.findAll('[data-test="todo"]')).toHaveLength(2)
})
```

测试被换行分割成三个显著阶段。这三个阶段代表着测试的三个时期：**编排**，**执行**与**断言**。

在 _编排_ 阶段，我们为测试设置方案。一个更复杂的示例可能需要创建一个 Vuex 存储，或者构建一个数据库。

在 _执行_ 阶段，我们在方案上执行，模拟用户如何与组件或者应用践行交互。

在 _断言_ 阶段，我们断言当前组件所处的状态的预期。

几乎所有的测试都遵循着这三个步骤。你不必用教程中这样的换行来分割它们，但是在你写你的测试的时候最好记住这三个阶段。

## 总结

- 使用 `mount()` 来渲染一个组件。
- 使用 `get()` 和 `findAll()` 来查找 DOM。
- `trigger()` 和 `setValue()` 可以用户模拟用户的输入。
- 更新 DOM 是一个异步操作，所以确保使用了 `async` 和 `await`。
- 测试通常由 3 部分组成， **编排**，**执行**与**断言**。
