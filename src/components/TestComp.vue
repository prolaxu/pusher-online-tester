<template>
  <div>
    <q-input v-model="vmodel" />
  </div>
</template>
<script setup>
import {computed} from "vue";

const props =defineProps({
  modelValue: {},
})
const emit = defineEmits(['update:modelValue','hi'])
const vmodel = computed({
  get: () => props.modelValue,
  set: (value) => {
    if (value ==='hi'){
      emit('hi', "fdasdfjalkdfja")
    }
    emit('update:modelValue', value)
  }
})
</script>
