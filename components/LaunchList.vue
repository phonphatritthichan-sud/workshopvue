<template>
  <article
    v-for="launch in filtered"
    :key="launch.id"
    class="flex items-center w-[70%] mx-auto mb-3 bg-white rounded-lg p-3 shadow-md cursor-pointer transition-transform hover:scale-[1.01]"
    @click="handleOpenModal(launch)"
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
            v-if="launch.crews?.length"
            class="bg-brand text-white font-bold badge-base"
          >
            {{ launch.crews.length }} crews
          </span>
          <span class="bg-gray-200 badge-base">
            {{ formatDate(launch.date_utc) }}
          </span>
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
const props = defineProps({
  filtered: Array,
  formatDate: Function,
  badge: Function,
});
const emit = defineEmits(["openModal"]);

const badgeBaseClass = "px-2 py-0.5 rounded";

const handleOpenModal = (launch) => {
  emit("openModal", launch);
};

const pickImage = (links) => {
  if (!links) return "";
  if (links.patch?.small) return links.patch.small;
  if (links.flickr?.original?.length) return links.flickr.original[0];
  return "";
};
</script>

<style scoped>
.badge-base {
  padding: 0.125rem 0.5rem;
  border-radius: 0.25rem;
}
</style>
