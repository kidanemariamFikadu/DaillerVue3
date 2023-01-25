<template>
  <div class="bottom-right">
    <button class="close-button" @click="close">X</button>
    <div class="content">
      <Phone />
    </div>
  </div>
</template>

<script>
import Phone from "./phone.vue";
export default {
  components: {
    Phone,
  },
  methods: {
    close() {
      this.$emit("close");
    },
  },
};
</script>


<style>
.bottom-right {
  position: fixed;
  bottom: 20px;
  right: 20px;
  z-index: 1000;
}

.bottom-right .close-button {
  position: absolute;
  top: -10px;
  right: -10px;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background-color: #ddd;
  color: #fff;
  font-size: 24px;
  line-height: 28px;
  text-align: center;
  cursor: pointer;
}

.bottom-right .content {
  padding: 20px;
  background-color: #fff;
  border-radius: 10px;
}
</style>
