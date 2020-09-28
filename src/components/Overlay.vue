<template>
  <div class="overlay" @click="$emit('overlay-click')"></div>
</template>

<script>
export default {};
</script>

<style scoped>
.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: rgba(0, 0, 0, 0.3);
  z-index: 1;
}
</style>
