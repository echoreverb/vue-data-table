<template>
  <div class="header">
    <div class="header__container">
      <p href="#" class="header__logo">vue-data-table</p>
      <nav class="header__nav"><slot></slot></nav>
    </div>
  </div>
</template>

<script>
export default {};
</script>

<style scoped>
.header {
  font-family: Arial, Helvetica, sans-serif;
  width: 100%;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
}

.header__nav {
  display: flex;
  height: 37px;
}

.header__container {
  max-width: 1120px;
  width: calc(100% - 130px);
  margin: 0 auto;
  box-sizing: border-box;
  display: flex;
  align-items: center;
  justify-content: space-between;
  min-height: 60px;
}

.header__logo{
  font-size: 16px;
  font-weight: bold;
  text-decoration: none;
  color: inherit;
  cursor: default;
}
</style>
