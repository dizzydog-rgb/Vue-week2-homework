<script setup>
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <header>
    <div class="wrapper">
      <nav>
        <RouterLink to="/">Home</RouterLink> | 
        <RouterLink to="/todo">Todo</RouterLink>
      </nav>
    </div>
  </header>
  <RouterView />
</template>
