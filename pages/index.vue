<template>
  <div class="min-h-screen font-sans text-[#000000] bg-gray-100">
    <header class="flex flex-col p-4 bg-[#e9e9e9] sticky top-0 space-y-2">
      <div class="flex justify-center">
        <h1 class="text-xl font-bold">🚀 SpaceX Launches</h1>
      </div>

      <!-- Navigation -->
      <nav class="flex justify-center">
        <div class="flex space-x-4 bg-white rounded-lg p-2 shadow-sm">
          <NuxtLink
            to="/"
            class="px-4 py-2 rounded-md text-sm font-medium transition-colors"
            :class="$route.path === '/' ? 'bg-blue-600 text-white' : 'text-gray-700 hover:bg-gray-100'"
          >
            🚀 Launches
          </NuxtLink>
          <NuxtLink
            to="/todo"
            class="px-4 py-2 rounded-md text-sm font-medium transition-colors"
            :class="$route.path === '/todo' ? 'bg-blue-600 text-white' : 'text-gray-700 hover:bg-gray-100'"
          >
            📋 Todo List
          </NuxtLink>
        </div>
      </nav>

      <div class="flex justify-center">
        <TabFilter
          :scopeFilter="scopeFilter"
          @update:scopeFilter="scopeFilter = $event"
        />
      </div>

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
        :formatDate="formatDate"
        :badge="badge"
        @openModal="openModal"
      />
    </main>

    <CrewModal
      v-if="isModalVisible"
      :modal="modal"
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

import TabFilter from "~/components/TabFilter.vue";
import SearchSort from "~/components/SearchSort.vue";
import LaunchList from "~/components/LaunchList.vue";
import CrewModal from "~/components/CrewModal.vue";

const API = "https://api.spacexdata.com/v4";

const launches = ref([]);
const rockets = ref({});
const pads = ref({});
const crewMap = ref({});
const loading = ref(true);

const searchQuery = ref("");
const scopeFilter = ref("all");
const sortKey = ref("date");
const sortOrder = ref("desc");

const isModalVisible = ref(false);
const modal = ref(null);

const formatDate = (dateStr) =>
  moment(dateStr).format("dddd, MMMM Do YYYY, h:mm:ss a");

const badgeBaseClass = "px-2 py-0.5 rounded";
const badge = (status, upcoming) => {
  if (upcoming)
    return `<span class='${badgeBaseClass} bg-yellow-100 text-yellow-600'>upcoming</span>`;
  if (status === true)
    return `<span class='${badgeBaseClass} bg-green-100 text-green-600'>launches</span>`;
  if (status === false)
    return `<span class='${badgeBaseClass} bg-red-100 text-red-600'>ล้มเหลว</span>`;
  return `<span class='${badgeBaseClass} bg-gray-200 text-gray-600'>ไม่ทราบ</span>`;
};

const filtered = computed(() => {
  let lists = launches.value.filter((launch) => {
    if (scopeFilter.value === "upcoming" && !launch.upcoming) return false;
    if (scopeFilter.value === "past" && launch.upcoming) return false;
    if (
      searchQuery.value &&
      !launch.name?.toLowerCase().includes(searchQuery.value.toLowerCase())
    )
      return false;
    return true;
  });
  lists.sort((a, b) => {
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
  return lists;
});

const fetchByIds = async (endpoint, ids) => {
  if (!ids.size) return {};
  try {
    const body = {
      query: { _id: { $in: Array.from(ids) } },
      options: { pagination: false },
    };
    const res = await fetch(`${API}/${endpoint}/query`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(body),
    });
    const json = await res?.json();
    const docs = json?.docs || [];
    return Object.fromEntries(docs.map((doc) => [doc.id, doc]));
  } catch {
    return {};
  }
};

const bootstrap = async () => {
  try {
    const res = await fetch(`${API}/launches`);
    const data = (await res?.json()) || [];

    const rocketIds = new Set();
    const padIds = new Set();
    const crewIds = new Set();

    data.forEach((launch) => {
      if (launch?.rocket) rocketIds.add(launch.rocket);
      if (launch?.launchpad) padIds.add(launch.launchpad);
      if (Array.isArray(launch?.crew))
        launch.crew.forEach((id) => crewIds.add(id));
    });

    rockets.value = await fetchByIds("rockets", rocketIds);
    pads.value = await fetchByIds("launchpads", padIds);
    crewMap.value = await fetchByIds("crew", crewIds);

    launches.value = data.map((launch) => ({
      ...launch,
      crews: (launch.crew || []).map((id) => crewMap.value[id]).filter(Boolean),
    }));

    loading.value = false;
  } catch {
    loading.value = false;
  }
};

onMounted(bootstrap);

const openModal = (launch) => {
  modal.value = launch;
  isModalVisible.value = true;
};

const closeModal = () => {
  isModalVisible.value = false;
  modal.value = null;
};
</script>
