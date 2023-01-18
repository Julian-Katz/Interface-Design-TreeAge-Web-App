<template>
  <div class="input">
    <button
      v-for="button in buttons"
      :key="button"
      v-html="button.text"
      type="button"
      @click="handleClick(button)"
    />
  </div>
</template>

<script>
export default {
  name: 'Message',
  props: {
    buttons: {
      type: Boolean,
      default: false,
    },
  },
  methods: {
    handleClick(button) {
      this.$emit('buttonClicked', button);
    },
  },
};
</script>

<style scoped lang="scss">

</style>
