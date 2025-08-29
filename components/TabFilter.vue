<template>
  <div class="flex mb-4 rounded-md shadow-sm w-full max-w-md mx-auto">
    <button
      v-for="(scope, index) in ['all', 'upcoming', 'past']"
      :key="scope"
      :class="[
        'flex-1 px-6 py-2 border font-bold transition cursor-pointer text-center',
        scopeFilter === scope
          ? 'bg-brand text-white'
          : 'bg-white text-brand hover:bg-blue-100',
        index === 0 ? 'rounded-l-md' : '',
        index === 2 ? 'rounded-r-md' : '',
        index !== 0 ? '-ml-px' : '',
      ]"
      @click="$emit('update:scopeFilter', scope)"
    >
      {{ tabLabel[scope] }}
    </button>
  </div>
</template>

<script setup>
defineProps({ scopeFilter: String });
defineEmits(["update:scopeFilter"]);
const tabLabel = { all: "All", upcoming: "Upcoming", past: "Launched" };
</script>
