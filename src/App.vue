<script setup>
import MovieList from './components/MovieList.vue'
</script>

<template>
  <header>
    <h1>Studio Ghibli Movies</h1>
  </header>
  <main>
    <MovieList />
  </main>
</template>

<style lang="scss" scoped>
header {
  line-height: 1.5;
  padding: 2rem;
  text-align: center;

  h1 {
    color: #333;

    &:hover {
      color: #666;
    }
  }
}

main {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 2rem;
}
</style>
