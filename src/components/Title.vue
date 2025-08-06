<script setup>
defineProps({
  msg: {
    type: String,
    required: true,
  },
})
</script>

<template>
  <h1>{{ msg }}</h1>
</template>

<style scoped>
h1 {
  font-weight: 500;
  font-size: 2.3rem;
  position: relative;
  padding-top: 10px;
}
</style>
