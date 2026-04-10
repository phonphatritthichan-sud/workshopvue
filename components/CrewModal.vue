<template>
  <div
    class="fixed inset-0 bg-black bg-opacity-60 flex justify-center items-center z-50"
    @click.self="handleCloseModal"
  >
    <div
      class="bg-white w-[70%] max-h-[90vh] overflow-y-auto rounded-xl p-5 shadow-2xl flex flex-col gap-4 items-center text-center"
    >
      <header class="w-full flex justify-center items-center relative pb-2">
        <h2 class="text-xl font-bold text-black">{{ modal?.name }}</h2>
        <button
          @click="handleCloseModal"
          class="absolute right-0 top-1/2 transform -translate-y-1/2 text-lg"
        >
          ✖
        </button>
      </header>

      <p class="my-1 text-base text-gray-700">
        {{ formatDate(modal.date_utc) }}
      </p>

      <h3
        class="inline-block px-3 py-1 rounded-full bg-brand text-white text-base my-2"
      >
        Crews
      </h3>
      <div class="flex flex-wrap gap-6 mt-2 justify-center text-black">
        <div
          v-for="crewMember in modal.crews"
          :key="crewMember.id"
          class="flex flex-col items-center gap-2 w-20"
        >
          <img
            v-if="crewMember.image"
            :src="crewMember.image"
            :alt="crewMember.name"
            class="w-16 h-16 object-cover rounded-full border-2 border-gray-700"
          />
          <span class="text-sm text-center">{{ crewMember.name }}</span>
        </div>
        <span v-if="!modal.crews?.length">—</span>
      </div>

      <h3
        class="inline-block px-3 py-1 rounded-full bg-brand text-white text-base my-2"
      >
        Rockets
      </h3>
      <p class="my-1 text-base text-gray-700">
        {{ rocketName }}
      </p>
      <div v-if="shouldShowRocketImage" class="my-2">
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
import { computed } from "vue";

const props = defineProps({
  modal: Object,
  rockets: Object,
  pads: Object,
  formatDate: Function,
});
const emit = defineEmits(["closeModal"]);

const handleCloseModal = () => {
  emit("closeModal");
};

const rocketName = computed(
  () => props.rockets[props.modal.rocket]?.name || "—"
);
const shouldShowRocketImage = computed(() =>
  Boolean(props.rockets[props.modal.rocket]?.flickr_images?.length)
);
</script>
