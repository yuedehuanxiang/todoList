<script setup>
import { computed, ref, watch } from 'vue'

const STORAGE_KEY = 'vue3-todolist-items'

const filters = [
  { label: '全部', value: 'all' },
  { label: '进行中', value: 'active' },
  { label: '已完成', value: 'completed' }
]

const newTodo = ref('')
const currentFilter = ref('all')
const todos = ref(loadTodos())

const totalCount = computed(() => todos.value.length)
const completedCount = computed(() => todos.value.filter((todo) => todo.done).length)
const activeCount = computed(() => totalCount.value - completedCount.value)

const filteredTodos = computed(() => {
  if (currentFilter.value === 'active') {
    return todos.value.filter((todo) => !todo.done)
  }

  if (currentFilter.value === 'completed') {
    return todos.value.filter((todo) => todo.done)
  }

  return todos.value
})

const progressPercent = computed(() => {
  if (totalCount.value === 0) {
    return 0
  }

  return Math.round((completedCount.value / totalCount.value) * 100)
})

const emptyText = computed(() => {
  if (currentFilter.value === 'active') {
    return '当前没有进行中的任务'
  }

  if (currentFilter.value === 'completed') {
    return '当前没有已完成的任务'
  }

  return '先添加一个待办事项吧'
})

watch(
  todos,
  (value) => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(value))
  },
  { deep: true }
)

function loadTodos() {
  const savedTodos = localStorage.getItem(STORAGE_KEY)

  if (!savedTodos) {
    return [
      { id: 1, title: '学习 Vue3 组合式 API', done: true },
      { id: 2, title: '完成一个 TodoList 小项目', done: false },
      { id: 3, title: '运行工程并检查页面效果', done: false }
    ]
  }

  try {
    return JSON.parse(savedTodos)
  } catch {
    return []
  }
}

function addTodo() {
  const title = newTodo.value.trim()

  if (!title) {
    return
  }

  todos.value.unshift({
    id: Date.now(),
    title,
    done: false
  })
  newTodo.value = ''
}

function removeTodo(id) {
  todos.value = todos.value.filter((todo) => todo.id !== id)
}

function clearCompleted() {
  todos.value = todos.value.filter((todo) => !todo.done)
  currentFilter.value = 'all'
}
</script>

<template>
  <main class="page-shell">
    <section class="todo-panel">
      <div class="panel-header">
        <div>
          <p class="eyebrow">Vue3 TodoList</p>
          <h1>今日任务</h1>
        </div>
        <div class="progress-pill">{{ progressPercent }}%</div>
      </div>

      <div class="progress-track" aria-label="任务完成进度">
        <span :style="{ width: `${progressPercent}%` }"></span>
      </div>

      <form class="todo-form" @submit.prevent="addTodo">
        <input
          v-model="newTodo"
          type="text"
          maxlength="40"
          placeholder="输入新的待办事项"
          aria-label="输入新的待办事项"
        />
        <button type="submit">添加</button>
      </form>

      <div class="summary-grid">
        <div>
          <span>{{ totalCount }}</span>
          <p>全部</p>
        </div>
        <div>
          <span>{{ activeCount }}</span>
          <p>进行中</p>
        </div>
        <div>
          <span>{{ completedCount }}</span>
          <p>已完成</p>
        </div>
      </div>

      <div class="toolbar">
        <div class="filters" aria-label="待办筛选">
          <button
            v-for="filter in filters"
            :key="filter.value"
            type="button"
            :class="{ active: currentFilter === filter.value }"
            @click="currentFilter = filter.value"
          >
            {{ filter.label }}
          </button>
        </div>
        <button
          class="clear-btn"
          type="button"
          :disabled="completedCount === 0"
          @click="clearCompleted"
        >
          清空已完成
        </button>
      </div>

      <TransitionGroup v-if="filteredTodos.length" name="todo-list" tag="ul" class="todo-list">
        <li v-for="todo in filteredTodos" :key="todo.id" :class="{ done: todo.done }">
          <label>
            <input v-model="todo.done" type="checkbox" />
            <span>{{ todo.title }}</span>
          </label>
          <button type="button" aria-label="删除待办" @click="removeTodo(todo.id)">删除</button>
        </li>
      </TransitionGroup>

      <div v-else class="empty-state">
        <span>✓</span>
        <p>{{ emptyText }}</p>
      </div>
    </section>
  </main>
</template>
