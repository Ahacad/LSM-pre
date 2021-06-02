<script setup lang="ts">
import { defineProps, ref } from "vue";

const props = defineProps({
  count: {
    default: 0,
  },
});

const counter = ref(props.count);

const getClass = (index: number) => {
  if (Number(counter.value) === 0) {
    return "text-black";
  }
  if (index + 1 === Number(counter.value)) {
    return "text-black";
  } else {
    return "text-gray-400";
  }
};

const items = [
  "Introduction to LSM-tree",
  "Measuring Latency",
  "Merge Shedulers",
  "Lessons and Conclusions",
  "References",
];
</script>

<template>
  <div>
    <ul>
      <li v-for="(item, key) in items" :key="key" :class="getClass(key)">
        {{ item }}
      </li>
    </ul>
  </div>
</template>

