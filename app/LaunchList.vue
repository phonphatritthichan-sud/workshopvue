<template>
  <article
    v-for="launch in filtered"
    :key="launch.id"
    class="flex items-center w-[70%] mx-auto mb-3 bg-white rounded-lg p-3 shadow-md cursor-pointer transition-transform hover:scale-[1.01]"
    @click="$emit('openModal', launch)"
  >
    <div
      class="flex items-center justify-center w-10 h-10 mr-4 bg-gray-100 rounded-md overflow-hidden"
    >
      <img
        v-if="pickImage(launch.links)"
        :src="pickImage(launch.links)"
        class="object-cover w-full h-full"
      />
      <span v-else>No photo</span>
    </div>
    <div class="flex-1 flex flex-col">
      <div class="flex justify-between items-center">
        <h2 class="text-sm font-bold text-gray-800">{{ launch.name }}</h2>
        <div class="flex gap-2 text-xs text-black">
          <span
            v-if="launch.crew?.length"
            class="bg-brand text-white font-bold px-2 py-0.5 rounded-full"
            >{{ launch.crew.length }} crews</span
          >
          <span class="bg-gray-200 px-2 py-0.5 rounded">{{
            formatDate(launch.date_utc)
          }}</span>
          <span
            class="font-bold text-brand"
            v-html="badge(launch.success, launch.upcoming)"
          ></span>
        </div>
      </div>
    </div>
  </article>
</template>

<script setup>
defineProps({
  filtered: Array,
  pickImage: Function,
  formatDate: Function,
  badge: Function,
});
defineEmits(["openModal"]);
</script>
