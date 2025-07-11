<script setup>
import { ref } from 'vue';

const counter = ref(0);

const props = defineProps({
  user: String,
});

const emits = defineEmits(['counter_update']);

const increment = () => {
  counter.value = counter.value + 1;
  emits('counter_update', props.user, counter.value);
};

const decrement = () => {
  counter.value = counter.value - 1;
  emits('counter_update', props.user, counter.value);
};
</script>

<template>
  <div>Hello {{ user }}</div>
  <button @click="decrement">-</button>
  {{ counter }}
  <button @click="increment">+</button>
</template>