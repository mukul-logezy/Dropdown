<template>
  <div class="relative">
    <!-- Activator Button -->
    <button
      @click="toggleDropdown"
      class="bg-white min-w-64 border border-gray-300 py-2 px-4 rounded"
      :class="isOpen ? 'border-blue-400' : 'border-gray-300'"
    >
      <div v-if="props.modelValue == undefined || !selectedItems?.length">
        Select
      </div>
      <div class="flex flex-row justify-between items-center" v-else>
        <div class="grid grid-cols-3 gap-2" v-if="props.multiple !== undefined">
          <div
            class="bg-gray-200 px-1 py-1 rounded"
            v-for="(item, index) in selectedItems"
            :key="index"
          >
            <div class="text-sm">{{ item?.title }}</div>
          </div>
        </div>
        <div v-else>
          {{ selectedItems?.[0]?.title }}
        </div>
        <div @click="clear">
          <svg
            class="h-5 w-5 text-gray-300"
            width="24"
            height="24"
            viewBox="0 0 24 24"
            stroke-width="2"
            stroke="currentColor"
            fill="none"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <path stroke="none" d="M0 0h24v24H0z" />
            <line x1="18" y1="6" x2="6" y2="18" />
            <line x1="6" y1="6" x2="18" y2="18" />
          </svg>
        </div>
      </div>
    </button>

    <!-- Dropdown Menu -->
    <Transition @before-enter="handleDropdownBeforeEnter">
      <div
        v-if="isOpen"
        ref="dropdownMenu"
        id="dropdown-menu"
        class="absolute mt-2 w-64 bg-white border rounded shadow-lg"
      >
        <!-- Dropdown Items -->
        <div class="px-2" v-for="item in props.items" :key="item.value">
          <label class="block cursor-pointer my-2 rounded">
            <span v-if="props.multiple !== undefined">
              <div
                :class="checkIfExists(item) ? 'bg-gray-300' : ''"
                class="rounded px-4"
                @click="updateModelValue(item)"
              >
                {{ item.title }}
              </div>
            </span>
            <div
              :class="checkIfExists(item) ? 'bg-gray-300' : ''"
              class="rounded px-4 w-100"
              v-else
              @click="selectItem(item)"
            >
              {{ item.title }}
            </div>
          </label>
        </div>
      </div>
    </Transition>
  </div>
</template>

<script setup>
import {
  ref,
  watch,
  defineProps,
  defineEmits,
  onMounted,
  onUnmounted,
} from "vue";

const isOpen = ref(false);
const selectedItems = ref([]);
const dropdownMenuRef = ref(null);

const props = defineProps(["multiple", "items", "modelValue"]);

const emit = defineEmits();

const toggleDropdown = () => {
  isOpen.value = !isOpen.value;
};

const selectItem = (item) => {
  selectedItems.value = [item];
  closeDropdown();
  updateModelValue();
};

const clear = () => {
  selectedItems.value = [];
  closeDropdown();
  emit("update:modelValue", []);
};
const updateModelValue = (item) => {
  if (props.multiple !== undefined) {
    if (checkIfExists(item)) {
      let index = selectedItems.value.findIndex(
        (el) => el.value === item.value
      );

      selectedItems.value.splice(index, 1);
    } else selectedItems.value.push(item);
  }

  let emitValue = selectedItems.value.map((el) => el.value);

  emit(
    "update:modelValue",
    props.multiple !== undefined ? emitValue : emitValue[0]
  );
};

// check if the v-model values in parent exists in the items
const checkIfExists = (item) => {
  let exist = false;
  if (selectedItems.value?.length) {
    selectedItems.value.forEach((el) => {
      if (el.value === item.value) exist = true;
    });
  }
  return exist;
};

const closeDropdown = () => {
  isOpen.value = false;
};

const handleDropdownBeforeEnter = () => {
  // Close dropdown if clicked outside
  document.addEventListener("click", handleClickOutside);
};

/**
 * Click outside the dropdown is not working for now.
 */
const handleClickOutside = (event) => {
  // console.log("Clicked outside", dropdownMenuRef.value, event);
  if (dropdownMenuRef.value && !dropdownMenuRef.value.contains(event.target)) {
    closeDropdown();
  }
};

// Watch for changes in the modelValue prop and update the selectedItems accordingly
watch(
  () => props.modelValue,
  () => {
    selectedItems.value = [];
    if (props.multiple !== undefined && props.modelValue !== undefined) {
      props.items.forEach((el) => {
        props.modelValue.forEach((val) => {
          if (el.value == val) selectedItems.value.push(el);
        });
      });
    } else if (props.multiple === undefined && props.modelValue !== undefined) {
      props.items.forEach((el) => {
        if (el.value == props.modelValue) selectedItems.value = [el];
      });
    } else selectedItems.value = [];
  }
);

// Updating selectedItems on Based of modelValue
onMounted(() => {
  if (props.multiple !== undefined && props.modelValue !== undefined) {
    props.items.forEach((el) => {
      props.modelValue.forEach((val) => {
        if (el.value == val) selectedItems.value.push(el);
      });
    });
  } else if (props.multiple === undefined && props.modelValue !== undefined) {
    props.items.forEach((el) => {
      if (el.value == props.modelValue) selectedItems.value = [el];
    });
  } else selectedItems.value = [];

  dropdownMenuRef.value = document.getElementById("dropdown-menu");
});

// unmounting the EventListener
onUnmounted(() => {
  document.removeEventListener("click", handleClickOutside);
});
</script>

<style scoped>
.v-enter-active,
.v-leave-active {
  transition: opacity 0.5s ease;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}
</style>
