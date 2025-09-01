<template>
  <div class="flex mb-4 rounded-md shadow-sm w-full max-w-md mx-auto">
    <button
      v-for="(tab, index) in tabs"
      :key="tab.key"
      :class="classTabs(tab, index)"
      @click="handleTabClick(tab.key)"
    >
      {{ tab.name }}
    </button>
  </div>
</template>

<script setup>
const props = defineProps({ scopeFilter: String });
const emit = defineEmits(["update:scopeFilter"]);

const tabs = [
  { key: "all", name: "All" },
  { key: "upcoming", name: "Upcoming" },
  { key: "past", name: "Launched" },
];

const handleTabClick = (key) => {
  emit("update:scopeFilter", key);
};

const classTabs = (tab, index) => {
  const base =
    "flex-1 px-6 py-2 border font-bold transition cursor-pointer text-center";
  const active =
    props.scopeFilter === tab.key
      ? "bg-brand text-white"
      : "bg-white text-brand hover:bg-blue-100";
  const rounded =
    index === 0
      ? "rounded-l-md"
      : index === tabs.length - 1
      ? "rounded-r-md"
      : "";
  const borderFix = index !== 0 ? "-ml-px" : "";
  return [base, active, rounded, borderFix];
};
</script>
