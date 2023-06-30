<template>
  <div class="container mx-auto max-w-5xl">
    <h1>Todos</h1>

    <form class="flex gap-2" @submit.prevent="() => addTodo()">
      <input
        class="border border-gray-700 rounded-md px-2 py-1"
        type="text"
        v-model="newTodo"
        placeholder="New Todo"
      />
      <button class="border border-gray-700 rounded-md px-2 py-1" type="submit">Add</button>
    </form>

    <div>
      <span v-if="loading">Loading...</span>
      <ul v-else class="space-y-4">
        <li
          v-for="todo in todos"
          :key="todo.id"
          class="relative px-2 rounded-md py-4 shadow-md boder border-gray-700 bg-stone-50"
        >
          <button class="absolute top-2 right-2" @click="() => deleteTodo(todo.id)">X</button>
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
import { nextTick, onMounted, onServerPrefetch, onUpdated, ref, watchEffect } from 'vue'

async function getTodos() {
  return await fetch('https://localhost:7064/api/todos')
    .then((response) => response.json())
    .then((json) => json)
}

async function toggleCompleted(id: number) {
  await fetch(`https://localhost:7064/api/todos/${id}/toggle`, {
    method: 'POST'
  }).then((response) => response.ok)

  todos.value = await getTodos()
}

async function deleteTodo(id: number) {
  todos.value = todos.value.filter((todo) => todo.id !== id)
  await fetch(`https://localhost:7064/api/todos/${id}`, {
    method: 'DELETE'
  })
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

const todos = ref<
  {
    id: number
    name: string
    isComplete: boolean
  }[]
>([])

const newTodo = ref('')

const loading = ref(false)

onServerPrefetch(async () => {
  const res = await getTodos()
  todos.value = res
})

onMounted(() => {
  console.log('mounted')
  if (todos.value.length) return
  loading.value = true
  getTodos().then((res) => {
    todos.value = res
    loading.value = false
  })
})
</script>
