<template>
  <div class="popup">
    <div class="popup__content">
      <slot></slot>
    </div>
    <img
      src="@/assets/images/close.svg"
      alt="close image"
      class="popup__close"
      @click="$emit('close-click')"
    />
  </div>
</template>

<script>
export default {
  props: {
    record: Object,
  },
};
</script>

<style scoped>
.popup {
  background-color: #fff;
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  min-width: 300px;
  min-height: 200px;
  box-sizing: border-box;
  padding: 10px 30px;
  z-index: 2;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
}

.popup__content {
  font-family: Arial, Helvetica, sans-serif;
  font-size: 16px;
  text-align: left;
}

.popup__close {
  background-image: url('/images/close.png');
  position: absolute;
  top: -30px;
  right: -30px;
  width: 30px;
  height: 30px;
  cursor: pointer;
}
</style>
