<template>
  <div class="app">
    <header>
      <h1>🚀 SpaceX Launches</h1>
      <div class="tabs">
        <button
          v-for="s in ['all', 'upcoming', 'past']"
          :key="s"
          :class="['tab', scope === s && 'active']"
          @click="scope = s"
        >
          {{ tabLabel[s] }}
        </button>
      </div>
      <div class="controls">
        <input v-model="q" type="search" placeholder="ค้นหาด้วยชื่อ…" />
        <select v-model="sortBy">
          <option value="date">เรียงตามเวลา</option>
          <option value="name">เรียงตามชื่อ</option>
        </select>
        <select v-model="order">
          <option value="desc">มาก → น้อย</option>
          <option value="asc">น้อย → มาก</option>
        </select>
      </div>
    </header>

    <main>
      <div v-if="loading">กำลังโหลดข้อมูล…</div>

      <article
        v-for="l in filtered"
        :key="l.id"
        class="card"
        @click="openModal(l)"
      >
        <div class="thumb">
          <img v-if="pickImage(l.links)" :src="pickImage(l.links)" />
          <span v-else>ไม่มีรูปภาพ</span>
        </div>
        <div class="content">
          <div class="header">
            <h2 class="name">{{ l.name }}</h2>
            <div class="meta">
              <span class="badge">{{ formatDate(l.date_utc) }}</span>
              <span
                class="state_upcoming"
                v-html="badge(l.success, l.upcoming)"
              ></span>
            </div>
          </div>
        </div>
      </article>
    </main>

    <!-- Modal popup ที่แสดงขึ้นมา-->
    <div v-if="show" class="modal-backdrop" @click.self="closeModal">
      <div class="modal">
        <header>
          <h2>{{ modal?.name }}</h2>
          <button @click="closeModal">ปิด</button>
        </header>
        <div>
          <p><b>เวลา:</b> {{ formatDate(modal.date_utc) }}</p>
          <p><b>รายละเอียด:</b> {{ modal.details || "—" }}</p>
          <p><b>Rocket:</b> {{ rockets[modal.rocket]?.name || "—" }}</p>
          <p><b>Launchpad:</b> {{ pads[modal.launchpad]?.name || "—" }}</p>
          <p>
            <b>Crew:</b>
            <span v-for="cid in modal.crew" :key="cid" class="chip">
              {{ crew[cid]?.name }}
            </span>
            <span v-if="!modal.crew?.length">—</span>
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";

const API = "https://api.spacexdata.com/v4";
const launches = ref([]);
const rockets = ref({});
const pads = ref({});
const crew = ref({});
const loading = ref(true);

const q = ref("");
const scope = ref("all");
const sortBy = ref("date");
const order = ref("desc");

const show = ref(false);
const modal = ref(null);

const tabLabel = {
  all: "All",
  upcoming: "upcoming",
  past: "Launched",
};

const formatDate = (iso) =>
  new Date(iso).toLocaleString(undefined, {
    dateStyle: "medium",
    timeStyle: "short",
  });

const badge = (status, upcoming) => {
  if (upcoming) return `<span class='badge warn'>upcoming</span>`;
  if (status === true) return `<span class='badge ok'>launches</span>`;
  if (status === false) return `<span class='badge bad'>ล้มเหลว</span>`;
  return `<span class='badge'>ไม่ทราบ</span>`;
};

const pickImage = (links) => {
  if (!links) return "";
  if (links.patch?.small) return links.patch.small;
  if (links.flickr?.original?.length) return links.flickr.original[0];
  return "";
};

const filtered = computed(() => {
  let list = launches.value.filter((l) => {
    if (scope.value === "upcoming" && !l.upcoming) return false;
    if (scope.value === "past" && l.upcoming) return false;
    if (q.value && !l.name.toLowerCase().includes(q.value.toLowerCase()))
      return false;
    return true;
  });
  list.sort((a, b) => {
    if (sortBy.value === "name") {
      return order.value === "asc"
        ? a.name.localeCompare(b.name)
        : b.name.localeCompare(a.name);
    } else {
      const da = new Date(a.date_utc).getTime();
      const db = new Date(b.date_utc).getTime();
      return order.value === "asc" ? da - db : db - da;
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
  show.value = true;
};

const closeModal = () => {
  show.value = false;
  modal.value = null;
};
</script>

<style scoped>
/* ย่อ CSS สำหรับ Vue (ใช้จากตัวอย่าง HTML เดิมได้เลย) */
.app {
  font-family: sans-serif;
  color: #e8ecf3;
  background: #0b1020;
  min-height: 100vh;
}
header {
  padding: 16px;
  background: #121833;
  position: sticky;
  top: 0;
}
.tabs {
  display: flex;
  gap: 8px;
  margin-bottom: 1rem;
  align-items: center;
  justify-content: center; /* จัดกึ่งกลางแนวนอน */
}

.tab {
  padding: 8px 16px;
  border: 1px solid #007bff;
  border-radius: 6px;
  background-color: white;
  color: #007bff;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.2s;
}

.tab:hover {
  background-color: #e6f0ff; /* ฟ้าอ่อนเมื่อ hover */
}

.tab.active {
  background-color: #007bff; /* น้ำเงิน */
  color: white; /* ตัวหนังสือขาว */
}

.controls {
  display: flex;
  gap: 8px;
  margin-top: 12px;
  justify-content: center;
  color: black;
}
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 12px;
  margin: 20px 0;
}
.card {
  display: flex;
  align-items: center;
  width: 100%;
  background: #fff;
  border-radius: 8px;
  padding: 12px;
  margin-bottom: 12px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
  cursor: pointer;
  transition: transform 0.2s ease;
}

.card:hover {
  transform: scale(1.01);
}

.thumb {
  flex: 0 0 60px;
  height: 60px;
  margin-right: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f3f3f3;
  border-radius: 6px;
  overflow: hidden;
}

.thumb img {
  max-width: 100%;
  max-height: 100%;
  object-fit: cover;
}

.content {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.header {
  display: flex;
  justify-content: space-between; /* ซ้าย-ขวา */
  align-items: center;
}

.name {
  font-size: 1.1rem;
  font-weight: bold;
  margin: 0;
  color: #333;
}

.meta {
  display: flex;
  gap: 8px;
  font-size: 0.85rem;
  color: #666;
}

.state_upcoming {
  font-weight: bold; /* ตัวหนา */
  color: #007bff; /* ตัวอักษรสีน้ำเงิน */
}

.badge {
  background: #eee;
  padding: 2px 8px;
  border-radius: 4px;
}
.badge.ok {
  color: #22c55e;
  border-color: #22c55e;
}
.badge.bad {
  color: #ef4444;
  border-color: #ef4444;
}
.badge.warn {
  color: #f59e0b;
  border-color: #f59e0b;
}
.modal-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
}
.modal {
  background: #1e293b;
  border-radius: 10px;
  padding: 20px;
  max-width: 600px;
  width: 100%;
}
.chip {
  display: inline-block;
  background: #0f172a;
  border: 1px solid #334155;
  border-radius: 12px;
  padding: 2px 8px;
  margin: 2px;
}
</style>
