<script setup lang="ts">
import { defineProps, ref } from "vue";

const props = defineProps({
  count: {
    default: 0,
  },
  allblack: {
    default: false,
    type: Boolean,
  },
});

const counter = ref(props.count);

const getClass = (index: number) => {
  if (props.allblack) {
    return "text-black";
  }
  const res = ["text-gray-400"];
  if (index === Number(props.count)) {
    res.push("text-black");
  }
  return res;
};

const items = [
  "Introduction to LSM-tree",
  "Setups and Measuring Models",
  "Conclusions",
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

