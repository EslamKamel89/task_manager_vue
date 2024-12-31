<script lang="ts">
export default {
  setup() {
    const message = 'Hello world'
    return {
      message,
    }
  },
}
</script>

<template>
  <h2>{{ message }}</h2>
</template>

<style></style>
