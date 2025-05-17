---
title: Congratulations!
prev: /tutorial/09-error-handling
solvable: false
---

# 恭喜！

你已经完成了Preact教程！

可以随意继续玩一下这个演示代码。

### 下一步

- [了解更多关于类组件的信息](/guide/v10/components)
- [了解更多关于钩子的信息](/guide/v10/hooks)
- [创建你自己的项目](https://vite.new/preact)

> **我们想要你的反馈！**
>
> 你觉得你学会了Preact吗？你有遇到困难吗？
>
> 欢迎在[这个讨论](https://github.com/preactjs/preact-www/discussions/815)中提供反馈。

```jsx:repl-initial
import { render } from 'preact';
import { useState, useEffect } from 'preact/hooks'

const getTodos = async () => {
  try {
    return JSON.parse(localStorage.todos)
  } catch (e) {
    return [
      { id: 1, text: '学习Preact', done: true },
      { id: 2, text: '制作一个很棒的应用', done: false },
    ]
  }
}

function ToDos() {
  const [todos, setTodos] = useState([])

  useEffect(() => {
    getTodos().then(todos => {
      setTodos(todos)
    })
  }, [])

  // 每次todos更改时...
  useEffect(() => {
    // ...将列表保存到localStorage:
    localStorage.todos = JSON.stringify(todos)
    // (尝试重新加载页面以查看保存的待办事项！)
  }, [todos])

  function toggle(id) {
    setTodos(todos => {
      return todos.map(todo => {
        // 用done状态已切换的版本替换匹配的待办事项
        if (todo.id === id) {
          todo = { ...todo, done: !todo.done }
        }
        return todo
      })
    })
  }

  function addTodo(e) {
    e.preventDefault()
    const form = e.target
    const text = form.todo.value
    // 给`todos`状态设置器传递一个回调以就地更新其值:
    setTodos(todos => {
      const id = todos.length + 1
      const newTodo = { id, text, done: false }
      return todos.concat(newTodo)
    })
    form.reset()
  }

  return (
    <div>
      <ul style={{ listStyle: 'none', padding: 0 }}>
        {todos.map(todo => (
          <li key={todo.id}>
            <label style={{ display: 'block' }}>
              <input type="checkbox" checked={todo.done} onClick={() => toggle(todo.id)} />
              {' ' + todo.text}
            </label>
          </li>
        ))}
      </ul>
      <form onSubmit={addTodo}>
        <input name="todo" placeholder="添加待办事项 [回车]" />
      </form>
    </div>
  )
}

render(<ToDos />, document.getElementById("app"));
```
