<template>
  <div class="tabs">
    <button
      v-for="scope in ['all', 'upcoming', 'past']"
      :key="scope"
      :class="['tab', scopeFilter === scope && 'active']"
      @click="$emit('update:scopeFilter', scope)"
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
