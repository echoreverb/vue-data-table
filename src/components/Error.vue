<template>
  <div class="error">
    <slot></slot>
  </div>
</template>

<script>
export default {};
</script>

<style scoped>
.error {
  display: block;
  min-width: 200px;
  min-height: 200px;
  padding: 30px;
  font-family: Arial, Helvetica, sans-serif;
  font-size: 16px;
}
</style>
