<template>
  <div>
    <div class="page-top">
      <h2>任务计划列表</h2>
    </div>

    <div class="main">
      <h3 class="big-title">添加任务:</h3>
      <input type="text" class="task-input" placeholder="提示: +回车即可添加任务" v-model="todo" @keyup.enter="addTodo">
      <ul class="task-count">
        <li>{{noCheckedLength}}个任务未完成</li>
        <li class="action">
          <a v-for="item in hashItems" :class="{active: item.isActive}" @click="handleHash(item)">{{item.title}}</a>
        </li>
      </ul>

      <h3 class="big-title">任务列表:</h3>
      <div class="tasks">
        <span class="no-task-tip" v-if="list.length==0">还没有添加任何任务</span>
        <ul class="todo-list">
          <li class="todo" v-for="(item, index) in filterList"
              :class="{completed: item.isChecked, editing: isEditorItem(item)}">
            <div class="view">
              <input class="toggle" type="checkbox" v-model="item.isChecked">
              <label @dblclick="editTodo(item)">{{item.title}}</label>
              <button @click="deleteTodo(index)" class="destroy"></button>
            </div>
            <input
              v-focus="isEditorItem(item)"
              @blur="editCompleted(item)"
              type="text"
              class="edit"
              v-model="item.title"
              @keyup.enter="editCompleted"
              @keyup.esc="cancelEdit(item)">
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
  let store = {
    save (key, value) {
      localStorage.setItem(key, JSON.stringify(value))
    },
    fetch (key) {
      return JSON.parse(localStorage.getItem(key)) || []
    }
  }
  let list = store.fetch('all')
  let filter = {
    all: function (list) {
      return list
    },
    finished: function (list) {
      return list.filter(function (item) {
        return item.isChecked
      })
    },
    unfinished: function () {
      return list.filter(function (item) {
        return !item.isChecked
      })
    }
  }
  export default {
    name: 'todoList',
    data () {
      return {
        todo: '',
        list: list,
        editorTodoItem: null,
        beforeTodoContent: '',
        hash: 'all',
        hashItems: [
          {
            title: '所有任务',
            isActive: true,
            hash: 'all'
          },
          {
            title: '未完成的任务',
            isActive: false,
            hash: 'unfinished'
          },
          {
            title: '完成的任务',
            isActive: false,
            hash: 'finished'
          }
        ]
      }
    },
    methods: {
      addTodo () {
        let item = {title: this.todo, isChecked: false}
        this.list.push(item)
        this.todo = ''
      },
      deleteTodo (index) {
        this.list.splice(index, 1)
      },
      editTodo (item) {
        this.beforeTodoContent = item.title
        this.editorTodoItem = item
      },
      isEditorItem (item) {
        return item === this.editorTodoItem
      },
      editCompleted () {
        this.editorTodoItem = null
      },
      cancelEdit (item) {
        item.title = this.beforeTodoContent
        this.editorTodoItem = null
      },
      handleHash (item) {
        for (let i = 0; i < this.hashItems.length; i++) {
          this.hashItems[i].isActive = false
        }
        item.isActive = true
        this.hash = item.hash
      }
    },
    computed: {
      noCheckedLength: function () {
        let count = 0
        for (let i = 0; i < this.list.length; i++) {
          if (!this.list[i].isChecked) {
            count++
          }
        }
        return count
      },
      filterList: function () {
        return filter[this.hash] ? filter[this.hash](this.list) : this.list
      }
    },
    directives: {
      'focus': {
        update (el, binding) {
          if (binding.value) {
            el.focus()
          }
        }
      }
    },
    watch: {
      list: {
        handler: function () {
          store.save('all', this.list)
        },
        deep: true
      }
    }
  }
</script>

<style>
  .page-top {
    width: 100%;
    height: 40px;
    background-color: #db4c3f;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .page-top h2 {
    font-size: 18px;
    color: #fff;
  }

  body {
    margin: 0;
    background-color: #fafafa;
    font: 14px 'Helvetica Neue', Helvetica, Arial, sans-serif;
  }

  h2 {
    margin: 0;
    font-size: 12px;
  }

  a {
    color: #000;
    text-decoration: none;
  }

  input {
    outline: 0;
  }

  button {
    margin: 0;
    padding: 0;
    border: 0;
    background: none;
    font-size: 100%;
    vertical-align: baseline;
    font-family: inherit;
    font-weight: inherit;
    color: inherit;
    outline: 0;
  }

  ul {
    padding: 0;
    margin: 0;
    list-style: none;
  }

  .main {
    width: 50%;
    margin: 0px auto;
    box-sizing: border-box;
  }

  .task-input {
    width: 99%;
    height: 30px;
    outline: 0;
    border: 1px solid #ccc;
  }

  .task-count {
    display: flex;
    margin: 10px 0;
  }

  .task-count li {
    padding-left: 10px;
    flex: 1;
    color: #dd4b39;
  }

  .task-count li:nth-child(1) {
    padding: 5px 0 0 10px;
  }

  .action {
    text-align: center;
    display: flex;
  }

  .action a {
    margin: 0px 10px;
    flex: 1;
    padding: 5px 0;
    color: #777;

  }

  .action a:nth-child(3) {
    margin-right: 0;
  }

  .active {
    border: 1px solid rgba(175, 47, 47, 0.2);
  }

  .tasks {
    background-color: #fff;
  }

  .no-task-tip {
    padding: 10px 0 10px 10px;
    display: block;
    border-bottom: 1px solid #ededed;
    color: #777;
  }

  .big-title {
    color: #222;
  }

  .todo-list {
    margin: 0;
    padding: 0;
    list-style: none;
  }

  .todo-list li {
    position: relative;
    font-size: 16px;
    border-bottom: 1px solid #ededed;
  }

  .todo-list li:hover {
    background-color: #fafafa;
  }

  .todo-list li.editing {
    border-bottom: none;
    padding: 0;
  }

  .todo-list li.editing .edit {
    display: block;
    width: 506px;
    padding: 13px 17px 12px 17px;
    margin: 0 0 0 43px;
  }

  .todo-list li.editing .view {
    display: none;
  }

  .todo-list li .toggle {
    text-align: center;
    width: 40px;
    /* auto, since non-WebKit browsers doesn't support input styling */
    height: auto;
    position: absolute;
    top: 5px;
    bottom: 0;
    margin: auto 0;
    border: none; /* Mobile Safari */
    -webkit-appearance: none;
    appearance: none;
  }

  .toggle {
    text-align: center;
    width: 40px;
    /* auto, since non-WebKit browsers doesn't support input styling */
    height: auto;
    position: absolute;
    top: 5px;
    bottom: 0;
    margin: auto 0;
    border: none; /* Mobile Safari */
    -webkit-appearance: none;
    appearance: none;
  }

  .toggle:after {
    content: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="40" height="40" viewBox="-10 -18 100 135"><circle cx="50" cy="50" r="40" fill="none" stroke="#ededed" stroke-width="3"/></svg>');
  }

  .toggle:checked:after {
    content: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="40" height="40" viewBox="-10 -18 100 135"><circle cx="50" cy="50" r="40" fill="none" stroke="#bddad5" stroke-width="3"/><path fill="#5dc2af" d="M72 25L42 71 27 56l-4 4 20 20 34-52z"/></svg>');
  }

  .todo-list li label {
    white-space: pre-line;
    word-break: break-all;
    padding: 15px 60px 15px 15px;
    margin-left: 45px;
    display: block;
    line-height: 1.2;
    transition: color 0.4s;
  }

  .todo-list li.completed label {
    color: #d9d9d9;
    text-decoration: line-through;
  }

  /*.tip-toggle {
      padding-left: 0;
      position: relative;
  }

  .tip-toggle .toggle {
      top: -2px;
  }

  .tip-toggle span {
      margin-left: 45px;
  }*/

  .todo-list li .destroy {
    display: none;
    position: absolute;
    top: 0;
    right: 10px;
    bottom: 0;
    width: 40px;
    height: 40px;
    font-size: 30px;
    color: #cc9a9a;
    margin: auto 0 11px;
    transition: color 0.2s ease-out;
  }

  .todo-list li .destroy:hover {
    color: #af5b5e;
  }

  .todo-list li .destroy:after {
    content: '×';
  }

  .todo-list li:hover .destroy {
    display: block;
  }

  .todo-list li .edit {
    display: none;
  }

  .todo-list li.editing:last-child {
    margin-bottom: -1px;
  }

</style>
