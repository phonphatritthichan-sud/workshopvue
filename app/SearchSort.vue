<!-- SearchSort.vue -->
<template>
  <div class="controls">
    <input
      :value="searchQuery"
      @input="$emit('update:searchQuery', $event.target.value)"
      type="search"
      placeholder="ค้นหาด้วยชื่อ…"
    />

    <select
      :value="sortKey"
      @change="$emit('update:sortKey', $event.target.value)"
    >
      <option value="date">เรียงตามเวลา</option>
      <option value="name">เรียงตามชื่อ</option>
    </select>

    <select
      :value="sortOrder"
      @change="$emit('update:sortOrder', $event.target.value)"
    >
      <option value="desc">มาก → น้อย</option>
      <option value="asc">น้อย → มาก</option>
    </select>
  </div>
</template>

<script setup>
defineProps({
  searchQuery: String,
  sortKey: String,
  sortOrder: String,
});

defineEmits(["update:searchQuery", "update:sortKey", "update:sortOrder"]);
</script>
