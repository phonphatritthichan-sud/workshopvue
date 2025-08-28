<template>
  <div class="flex gap-2 mt-3 justify-center text-black">
    <input
      type="search"
      placeholder="ค้นหาด้วยชื่อ…"
      :value="searchQuery"
      @input="$emit('update:searchQuery', $event.target.value)"
      class="px-2 py-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-brand"
    />
    <select
      :value="sortKey"
      @change="$emit('update:sortKey', $event.target.value)"
      class="px-2 py-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-brand"
    >
      <option value="date">เรียงตามเวลา</option>
      <option value="name">เรียงตามชื่อ</option>
    </select>
    <select
      :value="sortOrder"
      @change="$emit('update:sortOrder', $event.target.value)"
      class="px-2 py-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-brand"
    >
      <option value="desc">มาก → น้อย</option>
      <option value="asc">น้อย → มาก</option>
    </select>
  </div>
</template>

<script setup>
defineProps({ searchQuery: String, sortKey: String, sortOrder: String });
defineEmits(["update:searchQuery", "update:sortKey", "update:sortOrder"]);
</script>
