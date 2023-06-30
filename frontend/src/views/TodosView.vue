<template>
  <div class="container mx-auto max-w-5xl">
    <h1>Todos</h1>

    <form class="flex gap-2" @submit.prevent="() => addTodo()">
      <input
        class="border peer border-gray-700 rounded-md px-2 py-1"
        type="text"
        v-model="newTodo"
        required
        min="1"
        placeholder="New Todo"
      />
      <button
        class="border peer-invalid:border-red-500 disabled:opacity-30 border-gray-700 rounded-md px-2 py-1 flex items-center gap-2"
        type="submit"
        :disabled="!newTodo"
      >
        <PlusIcon class="w-4 h-4" />
        Add
      </button>
      <button
        type="button"
        @click="getTodos"
        class="border border-gray-700 rounded-md flex items-center gap-2 px-2 py-1"
      >
        <RefreshIcon class="w-4 h-4" />
        Refresh
      </button>
    </form>

    <div class="mt-4">
      <span v-if="loading">Loading...</span>
      <ul v-else class="space-y-4">
        <li
          v-for="todo in todos"
          :key="todo.id"
          class="relative px-2 rounded-md py-4 shadow-md boder border-gray-700 bg-stone-50"
        >
          <button
            class="absolute top-2 right-2 flex items-center gap-2"
            @click="() => deleteTodo(todo.id)"
          >
            X
          </button>
          <header class="flex items-center gap-2">
            <span> {{ todo.id }}. </span>
            <h2 class="text-xl font-bold">
              {{ todo.name }}
            </h2>
          </header>
          <div class="mt-4 px-4">
            <label>
              <input
                type="checkbox"
                :checked="todo.isComplete"
                @change="() => toggleCompleted(todo.id)"
              />
              Complete
            </label>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, onServerPrefetch, ref } from 'vue'
import { PlusIcon, ArrowPathIcon as RefreshIcon } from '@heroicons/vue/24/solid'

// methods
async function getTodos() {
  loading.value = true
  return await fetch('https://localhost:7064/api/todos')
    .then((response) => response.json())
    .then((json) => json)
    .then((todos) => {
      setTimeout(() => {
        loading.value = false
      }, 500)
      return todos
    })
}

async function toggleCompleted(id: number) {
  await fetch(`https://localhost:7064/api/todos/${id}/toggle`, {
    method: 'POST'
  }).then((response) => response.ok)
  todos.value = await getTodos()
}

async function deleteTodo(id: number) {
  // do optimistic update
  todos.value = todos.value.filter((todo) => todo.id !== id)
  await fetch(`https://localhost:7064/api/todos/${id}`, {
    method: 'DELETE'
  })
  // do pessimistic update
  todos.value = await getTodos()
}

async function addTodo() {
  const res = await fetch(`https://localhost:7064/api/todos`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      name: newTodo.value,
      isComplete: false
    })
  })
  const json = await res.json()
  todos.value = [...todos.value, json]
  newTodo.value = ''
}

// state
const todos = ref<
  {
    id: number
    name: string
    isComplete: boolean
  }[]
>([])
const newTodo = ref('')
const loading = ref(false)

// lifecycle
onServerPrefetch(async () => {
  const res = await getTodos()
  todos.value = res
})

onMounted(() => {
  if (todos.value.length) return
  getTodos().then((res) => {
    todos.value = res
    setTimeout(() => {
      loading.value = false
    }, 500)
  })
})
</script>
