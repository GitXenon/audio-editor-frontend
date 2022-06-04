<template>
  <button class="bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded w-40">
    <slot/>
  </button>
</template>
<script>
export default {
  name: "RecordButton"
}
</script>

<style scoped>

</style>