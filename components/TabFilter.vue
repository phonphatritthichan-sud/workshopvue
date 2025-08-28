<template>
  <div class="flex gap-4 justify-center">
    <button class="bg-purple-800 text-primary px-4 py-2 rounded">
      Test Button
    </button>
    <button
      v-for="scope in ['all', 'upcoming', 'past']"
      :key="scope"
      @click="$emit('update:scopeFilter', scope)"
      class="px-4 py-2 rounded-md font-bold border border-[#4d36ca] bg-[#4d36ca] text-white cursor-pointer hover:opacity-80 transition-colors duration-200"
    >
      {{ tabLabel[scope] }}
    </button>
  </div>
</template>

<script setup>
defineProps({
  scopeFilter: String,
});

defineEmits(["update:scopeFilter"]);

const tabLabel = {
  all: "All",
  upcoming: "Upcoming",
  past: "Launched",
};
</script>
