<template>
    <div class="gradient-loader"></div>
</template>

<script>
export default {};
</script>

<style scoped>

.gradient-loader {
  background: repeating-linear-gradient(to right, #2b3cff 0%, #c5d3ff 50%, #3748ff 100%);
  width: 100vw;
  height: 3px;
  background-size: 200% auto;
  background-position: 0 100%;
  animation: gradient 1.5s infinite;
  animation-fill-mode: forwards;
  animation-timing-function: linear;
}

@keyframes gradient {
  0% {
    background-position: 0 0;
  }
  100% {
    background-position: -200% 0;
  }
}
</style>
