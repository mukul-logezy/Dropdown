<script setup>
import { ref } from "vue";
import Dropdown from "./components/Dropdown.vue";

const dropdownItems = [
  { title: "single 1", value: "1" },
  { title: "select 2", value: "2" },
  { title: "select 3", value: "3" },
];

const dropdownItems2 = [
  { title: "multiple 1", value: "1" },
  { title: "multiple 2", value: "2" },
  { title: "multiple 3", value: "3" },
];

let singleSelect = ref();

let multipleSelect = ref();
</script>

<template>
  <div class="h-screen flex flex-row justify-center gap-8 items-center">
    <!-- Single select -->
    <div class="flex flex-col">
      <Dropdown v-model="singleSelect" :items="dropdownItems" />
      <p>Selected Item: {{ singleSelect }}</p>
    </div>
    <!-- Select with multiple option -->
    <div class="flex flex-col">
      <Dropdown
        v-model="multipleSelect"
        :items="dropdownItems2"
        multiple="true"
      />
      <p>Selected Item: {{ multipleSelect }}</p>
    </div>
  </div>
</template>

<style scoped></style>
