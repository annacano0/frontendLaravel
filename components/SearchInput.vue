<script setup lang="ts">
import {ref} from 'vue'
const emit = defineEmits<{
  (e: "update:modelValue", payload: string): void;
}>();
const prop = defineProps({
  modelValue:{type:String, required:true},
})
const userInput=ref('')
const localValue=ref(prop.modelValue)
const debouncedLocalValue=refDebounced(localValue,500)

watch(debouncedLocalValue, ()=>{
  emit('update:modelValue', localValue.value)
})
</script>
<template>
  <div class="relative">
    <IconSearch
      class="w-3 w-3 absolute top-[50%] translate-y-[-50%] left-2 opacity-30"
    />

    <input
      type="text"
      placeholder="Search"
      class="pl-10 p-2 rounded"
      v-model="userInput"
    />
  </div>
</template>
