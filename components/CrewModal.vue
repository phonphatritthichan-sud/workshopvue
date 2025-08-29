<template>
  <div
    class="fixed inset-0 bg-black bg-opacity-60 flex justify-center items-center z-50"
    @click.self="$emit('closeModal')"
  >
    <div
      class="bg-white w-[70%] max-h-[90vh] overflow-y-auto rounded-xl p-5 shadow-2xl flex flex-col gap-4 items-center text-center"
    >
      <header class="w-full flex justify-center items-center relative pb-2">
        <h2 class="text-xl font-bold text-black">{{ modal?.name }}</h2>
        <button
          @click="$emit('closeModal')"
          class="absolute right-0 top-1/2 transform -translate-y-1/2 text-lg"
        >
          ✖
        </button>
      </header>

      <p class="my-1 text-base text-gray-700">
        {{ formatDate(modal.date_utc) }}
      </p>

      <!-- Crews -->
      <h3
        class="inline-block px-3 py-1 rounded-full bg-brand text-white text-base my-2"
      >
        Crews
      </h3>
      <div class="flex flex-wrap gap-6 mt-2 justify-center text-black">
        <div
          v-for="crewId in modal.crew"
          :key="crewId"
          class="flex flex-col items-center gap-2 w-20"
        >
          <img
            v-if="crew[crewId]?.image"
            :src="crew[crewId].image"
            :alt="crew[crewId]?.name"
            class="w-16 h-16 object-cover rounded-full border-2 border-gray-700"
          />
          <span class="text-sm text-center">{{ crew[crewId]?.name }}</span>
        </div>
        <span v-if="!modal.crew?.length">—</span>
      </div>

      <h3
        class="inline-block px-3 py-1 rounded-full bg-brand text-white text-base my-2"
      >
        Rockets
      </h3>
      <p class="my-1 text-base text-gray-700">
        {{ rockets[modal.rocket]?.name || "—" }}
      </p>
      <div v-if="rockets[modal.rocket]?.flickr_images?.length" class="my-2">
        <img
          :src="rockets[modal.rocket].flickr_images[0]"
          alt="Rocket Image"
          class="max-w-full rounded-lg"
        />
      </div>

      <h3
        class="inline-block px-3 py-1 rounded-full bg-brand text-white text-base my-2"
      >
        Launchpad
      </h3>
      <p class="my-1 text-base text-gray-700">
        {{ pads[modal.launchpad]?.full_name || "—" }}
      </p>

      <h3
        class="inline-block px-3 py-1 rounded-full bg-brand text-white text-base my-2"
      >
        Details
      </h3>
      <p class="my-1 text-base text-gray-700">{{ modal.details || "—" }}</p>
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
