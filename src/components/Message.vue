<template>
    <div
      class="message"
      :class="{ user: userMessage, bot: !userMessage }"
    >
      <slot/>
    </div>
</template>

<script>
export default {
  name: 'Message',
  props: {
    userMessage: {
      type: Boolean,
      default: false,
    },
  },
};
</script>

<style scoped lang="scss">
.message {
    width: 85%;
    padding: 1rem;
    box-shadow: 3px 3px 10px rgba(0, 0, 0, 0.2);
    p {
        width: 100%;
    }
}
.bot {
    background: rgb(255, 255, 255);
    border-radius: 1rem 1rem 1rem 0rem;
}
.user {
    background: rgb(172, 249, 255);
    border-radius: 1rem 1rem 0rem 1rem;
    align-self: flex-end;
}
</style>
