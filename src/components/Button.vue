<template>
  <button
    :disabled="isDisabled"
    :class="['button', { button_disabled: isDisabled }]"
    @click="$emit('button-click')"
  >
    <slot></slot>
  </button>
</template>

<script>
export default {
  props: {
    isDisabled: Boolean,
  },
};
</script>

<style scoped>
.button {
  display: flex;
  align-items: center;
  min-width: 120px;
  justify-content: center;
  height: 100%;
  font-family: Arial, Helvetica, sans-serif;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  color: #fff;
  border: none;
  background-color: #536ffc;
  outline: none;
  /* border-radius: 50px; */
  font-size: 14px;
  cursor: pointer;
  padding: 18px;
  margin-left: 10px;
  box-sizing: border-box;
}

.button.button_disabled {
  opacity: 0.6;
  cursor: auto;
}

.button:hover {
  background-color: #5d77fe;
}

.button:active {
  background-color: #3d5dfc;
}

.button:hover.button_disabled {
  background-color: #536ffc;
}
</style>
