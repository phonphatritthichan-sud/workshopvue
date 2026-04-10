<template>
  <div class="flex gap-2 mt-3 justify-center text-black">
    <input
      type="search"
      placeholder="ค้นหาด้วยชื่อ…"
      :value="searchQuery"
      @input="handleSearchInput"
      class="px-2 py-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-brand"
    />
    <select
      :value="sortKey"
      @change="handleSortKeyChange"
      class="px-2 py-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-brand"
    >
      <option value="date">เรียงตามเวลา</option>
      <option value="name">เรียงตามชื่อ</option>
    </select>
    <select
      :value="sortOrder"
      @change="handleSortOrderChange"
      class="px-2 py-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-brand"
    >
      <option value="desc">มาก → น้อย</option>
      <option value="asc">น้อย → มาก</option>
    </select>
  </div>
</template>

<script setup>
const props = defineProps({
  searchQuery: String,
  sortKey: String,
  sortOrder: String,
});
const emit = defineEmits([
  "update:searchQuery",
  "update:sortKey",
  "update:sortOrder",
]);

const handleSearchInput = (event) => {
  emit("update:searchQuery", event.target.value);
};
const handleSortKeyChange = (event) => {
  emit("update:sortKey", event.target.value);
};
const handleSortOrderChange = (event) => {
  emit("update:sortOrder", event.target.value);
};
</script>
