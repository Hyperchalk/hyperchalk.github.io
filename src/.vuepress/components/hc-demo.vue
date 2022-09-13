<template>
  <div>
    <input type="text" v-bind="url">
    <slot></slot>
  </div>
</template>

<script>
  export default {
    data() {
      let hash = new URLSearchParams(window.location.hash.slice(1))
      return {
        url: hash.get("hcUrl") ?? ""
      }
    }
  }
</script>
