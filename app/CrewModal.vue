<template>
  <div class="modal-backdrop" @click.self="$emit('closeModal')">
    <div class="modal">
      <header class="modal-header">
        <h2>{{ modal?.name }}</h2>
        <button class="close-btn" @click="$emit('closeModal')">✖</button>
      </header>

      <div class="modal-content">
        <p class="section">{{ formatDate(modal.date_utc) }}</p>

        <h3 class="section-title">Crews</h3>
        <div class="crew-list">
          <div v-for="crewId in modal.crew" :key="crewId" class="crew-item">
            <img
              v-if="crew[crewId]?.image"
              :src="crew[crewId].image"
              :alt="crew[crewId]?.name"
            />
            <span>{{ crew[crewId]?.name }}</span>
          </div>
          <span v-if="!modal.crew?.length">—</span>
        </div>

        <h3 class="section-title">Rockets</h3>
        <p class="section">{{ rockets[modal.rocket]?.name || "—" }}</p>

        <div
          v-if="rockets[modal.rocket]?.flickr_images?.length"
          class="rocket-image"
        >
          <img
            :src="rockets[modal.rocket].flickr_images[0]"
            alt="Rocket Image"
          />
        </div>

        <h3 class="section-title">Launchpad</h3>
        <p class="section">{{ pads[modal.launchpad]?.full_name || "—" }}</p>

        <h3 class="section-title">Details</h3>
        <p class="section">{{ modal.details || "—" }}</p>
      </div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  modal: Object,
  crew: Object,
  rockets: Object,
  pads: Object,
  formatDate: Function,
});
defineEmits(["closeModal"]);
</script>
