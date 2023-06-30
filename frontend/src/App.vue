<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <header class="flex flex-col items-center justify-center gap-4">
    <img alt="Vue logo" class="" src="@/assets/logo.svg" width="125" height="125" />

    <div class="">
      <h1 class="text-4xl font-bold leading-5 tracking-tight">Todos!</h1>

      <nav class="mt-6 flex gap-4 items-center justify-center">
        <RouterLink to="/" class="hover:underline">Home</RouterLink>
        <RouterLink to="/todos" class="hover:underline" :active-class="'text-green-400'"
          >Todos</RouterLink
        >
      </nav>
    </div>
  </header>

  <RouterView />
</template>
