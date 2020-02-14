<template>
  <div>
    <h1 class="title">Todos</h1>
    <h1 class="email">{{userEmail}}</h1>
    <section class="todoapp">
      <div v-if="loading">
        <h1 class="loading">Loading...</h1>
      </div>
      <div v-else>
        <header class="header">
          <input 
            autofocus autocomplete="off"
            :placeholder="this.inputPlaceholder"
            v-model="newTodo"
            @keyup.enter="addTodo"
            class="new-todo">
        </header>
        <section class="main" v-show="todos.length" v-cloak>
          <input type="checkbox" class="toggle-all" v-model="allDone">
          <ul class="todo-list">
            <li v-for="todo in filteredTodos" 
                class="todos" :key="todo.id"
                :class="{ completed: todo.completed, editing: todo == editedTodo }">
              <div class="view">
                <input type="checkbox" class="toggle" v-model="todo.completed" @change="completeTodo(todo)">
                <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
                <button class="destroy" @click="removeTodo(todo)"></button>
              </div>
              <input type="text" class="edit"
                v-model="todo.title"
                v-todo-focus="todo == editedTodo"
                @blur="doneEdit(todo)"
                @keyup.enter="doneEdit(todo)"
                @keyup.esc="cancelEdit(todo)">
            </li>
          </ul>
        </section>
        <footer class="footer" v-show="todos.length" v-cloak>
          <span class="todo-count">
            <strong>{{ remaining }}</strong> {{ remaining | pluralize }} left
          </span>
          <ul class="filters">
            <li><a href="#/all" @click="setVisibility('all')" :class="{ selected: visibility == 'all' }">All</a></li>
            <li><a href="#/active" @click="setVisibility('active')" :class="{ selected: visibility == 'active' }">Active</a></li>
            <li><a href="#/completed" @click="setVisibility('completed')" :class="{ selected: visibility == 'completed' }">Completed</a></li>
          </ul>
          <button class="clear-completed" @click="removeCompleted" v-show="todos.length > remaining">
            Clear completed
          </button>
        </footer>
      </div>
    </section>
    <div v-if="error" class="error" @click="handleErrorClick">
      ERROR: {{this.error}}
    </div>
  </div>
</template>

<script>

let filters = {
  all: todos => todos,
  active: todos => todos.filter(todo => !todo.completed),
  completed: todos => todos.filter(todo => todo.completed)
}
export default {
  name: 'Todos',
  props: {
    activeUser: Object
  },

  // app initial state
  data: () => ({
    todos: [],
    newTodo: '',
    editedTodo: null,
    visibility: 'all',
    loading: true,
    error: null,
  }),
  mounted() {
    // inject some startup data
    this.todos = [{id:1, title: 'Study Java', completed: false}, {id:2, title: 'Create Microservice in Go', completed: false}, {id:3, title: 'Drink Coffee', completed: false}]
    // hide the loading message
    this.loading = false
  },

  // computed properties
  computed: {
    filteredTodos: function () {
      return filters[this.visibility](this.todos)
    },
    remaining: function () {
      return filters.active(this.todos).length
    },
    allDone: {
      get: function () {
        return this.remaining === 0
      },
      set: function (value ) {
        this.todos.forEach(function (todo) {
          todo.completed = value
        })
      }
    },
    userEmail: function () {
      return this.activeUser ? this.activeUser.email : ''
    },
    inputPlaceholder: function () {
      return this.activeUser ? this.activeUser.given_name + ', what needs to be done?': 'What needs to be done?'     
    }
  },
  filters: {
    pluralize: n => n === 1 ? 'item' : 'items'
  },
  // methods that implement data logic.
  // note there's no DOM manipulation here at all.
  methods: {
    addTodo: function () {
      const value = this.newTodo && this.newTodo.trim()
      if (!value) return

      this.todos.push({
        title: value,
        completed: false
      })

      this.newTodo = ''
    },
    
    setVisibility: function(vis) {
      this.visibility = vis
    },

    completeTodo (todo) {
      console.log(todo);
    },
    removeTodo: function (todo) {
      this.todos.splice(this.todos.indexOf(todo), 1)
    },

    editTodo: function (todo) {
      this.beforeEditCache = todo.title
      this.editedTodo = todo
    },

    doneEdit: function (todo) {
      if (!this.editedTodo) return

      this.editedTodo = null
      todo.title = todo.title.trim()

      if (!todo.title) {
        this.removeTodo(todo)
      }
    },

    cancelEdit: function (todo) {
      this.editedTodo = null
      todo.title = this.beforeEditCache
    },

    removeCompleted: function () {
      this.todos = filters.active(this.todos)
    },
    
    handleErrorClick: function () {
      this.error = null
    }
  },

  // a custom directive to wait for the DOM to be updated
  // before focusing on the input field.
  directives: {
    'todo-focus': function (el, binding) {
      if (binding.value) {
        el.focus()
      }
    }
  }
}
</script>

<style scoped>
  [v-cloak] { display: none; }
</style>