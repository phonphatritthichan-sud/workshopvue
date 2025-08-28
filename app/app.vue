<template>
  <div class="min-h-screen font-sans text-[#000000] bg-gray-100">
    <header class="flex flex-col p-4 bg-[#e9e9e9] sticky top-0 space-y-2">
      <!-- บรรทัด 1: Title -->
      <div class="flex justify-center">
        <h1 class="text-xl font-bold">🚀 SpaceX Launches</h1>
      </div>

      <!-- บรรทัด 2: Tabs -->
      <div class="flex justify-center">
        <TabFilter
          :scopeFilter="scopeFilter"
          @update:scopeFilter="scopeFilter = $event"
        />
      </div>

      <!-- บรรทัด 3: Search + Sort -->
      <div class="flex justify-center space-x-2">
        <SearchSort
          :searchQuery="searchQuery"
          :sortKey="sortKey"
          :sortOrder="sortOrder"
          @update:searchQuery="searchQuery = $event"
          @update:sortKey="sortKey = $event"
          @update:sortOrder="sortOrder = $event"
        />
      </div>
    </header>

    <main>
      <div v-if="loading" class="py-4 text-center">กำลังโหลดข้อมูล…</div>

      <LaunchList
        :filtered="filtered"
        :pickImage="pickImage"
        :formatDate="formatDate"
        :badge="badge"
        @openModal="openModal"
      />
    </main>

    <CrewModal
      v-if="isModalVisible"
      :modal="modal"
      :crew="crew"
      :rockets="rockets"
      :pads="pads"
      :formatDate="formatDate"
      @closeModal="closeModal"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
import moment from "moment";
import { useHead } from "#imports";

import TabFilter from "./TabFilter.vue";
import SearchSort from "./SearchSort.vue";
import LaunchList from "./LaunchList.vue";
import CrewModal from "./CrewModal.vue";

import "../src/index.css";

const API = "https://api.spacexdata.com/v4";

const launches = ref([]);
const rockets = ref({});
const pads = ref({});
const crew = ref({});
const loading = ref(true);

const searchQuery = ref("");
const scopeFilter = ref("all");
const sortKey = ref("date");
const sortOrder = ref("desc");

const isModalVisible = ref(false);
const modal = ref(null);

const tabLabel = {
  all: "All",
  upcoming: "Upcoming",
  past: "Launched",
};

const formatDate = (dateStr) =>
  moment(dateStr).format("dddd, MMMM Do YYYY, h:mm:ss a");

const badge = (status, upcoming) => {
  if (upcoming)
    return `<span class='px-2 py-0.5 rounded bg-yellow-100 text-yellow-600'>upcoming</span>`;
  if (status === true)
    return `<span class='px-2 py-0.5 rounded bg-green-100 text-green-600'>launches</span>`;
  if (status === false)
    return `<span class='px-2 py-0.5 rounded bg-red-100 text-red-600'>ล้มเหลว</span>`;
  return `<span class='px-2 py-0.5 rounded bg-gray-200 text-gray-600'>ไม่ทราบ</span>`;
};

const pickImage = (links) => {
  if (!links) return "";
  if (links.patch?.small) return links.patch.small;
  if (links.flickr?.original?.length) return links.flickr.original[0];
  return "";
};

const filtered = computed(() => {
  let list = launches.value.filter((l) => {
    if (scopeFilter.value === "upcoming" && !l.upcoming) return false;
    if (scopeFilter.value === "past" && l.upcoming) return false;
    if (
      searchQuery.value &&
      !l.name.toLowerCase().includes(searchQuery.value.toLowerCase())
    )
      return false;
    return true;
  });
  list.sort((a, b) => {
    if (sortKey.value === "name") {
      return sortOrder.value === "asc"
        ? a.name.localeCompare(b.name)
        : b.name.localeCompare(a.name);
    } else {
      const da = new Date(a.date_utc).getTime();
      const db = new Date(b.date_utc).getTime();
      return sortOrder.value === "asc" ? da - db : db - da;
    }
  });
  return list;
});

const fetchByIds = async (endpoint, ids) => {
  if (!ids.size) return {};
  const body = {
    query: { _id: { $in: Array.from(ids) } },
    options: { pagination: false },
  };
  const res = await fetch(`${API}/${endpoint}/query`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(body),
  });
  const json = await res.json();
  const map = {};
  (json.docs || []).forEach((d) => (map[d.id] = d));
  return map;
};

const bootstrap = async () => {
  const res = await fetch(`${API}/launches`);
  const data = await res.json();
  launches.value = data;

  const rocketIds = new Set();
  const padIds = new Set();
  const crewIds = new Set();
  data.forEach((l) => {
    if (l.rocket) rocketIds.add(l.rocket);
    if (l.launchpad) padIds.add(l.launchpad);
    if (Array.isArray(l.crew)) l.crew.forEach((id) => crewIds.add(id));
  });

  rockets.value = await fetchByIds("rockets", rocketIds);
  pads.value = await fetchByIds("launchpads", padIds);
  crew.value = await fetchByIds("crew", crewIds);

  loading.value = false;
};

onMounted(bootstrap);

const openModal = (l) => {
  modal.value = l;
  isModalVisible.value = true;
};

const closeModal = () => {
  isModalVisible.value = false;
  modal.value = null;
};

useHead({ title: "SpaceX Launches" });
</script>
