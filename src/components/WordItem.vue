<script setup lang="ts">
import {defineProps} from 'vue'

const props = defineProps({
  value: { type: String, required: true },
  remove: {type: Function, required: true}
})
</script>

<template>
 <div>
   {{ value }}
   <button @click="remove">x</button>
 </div>
</template>

<style scoped>

</style>
