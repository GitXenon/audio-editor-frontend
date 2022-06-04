<template>
  <button
    class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded w-40"
  >
    <slot />
  </button>
</template>

<script lang="ts">
export default {
  name: "PlayButton",
};
</script>
