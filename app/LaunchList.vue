<template>
  <article
    v-for="launch in filtered"
    :key="launch.id"
    class="card"
    @click="$emit('openModal', launch)"
  >
    <div class="thumb">
      <img v-if="pickImage(launch.links)" :src="pickImage(launch.links)" />
      <span v-else>No photo</span>
    </div>

    <div class="content">
      <div class="header">
        <h2 class="name">{{ launch.name }}</h2>
        <div class="meta">
          <span v-if="launch.crew?.length" class="crew-badge">
            {{ launch.crew.length }} crews
          </span>
          <span class="badge">{{ formatDate(launch.date_utc) }}</span>
          <span
            class="state_upcoming"
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
