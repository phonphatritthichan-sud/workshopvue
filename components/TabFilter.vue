<template>
  <div class="flex mb-4 rounded-md shadow-sm w-full max-w-md mx-auto">
    <button
      v-for="(scope, index) in ['all', 'upcoming', 'past']"
      :key="scope"
      :class="[
        'flex-1 px-6 py-2 border font-bold transition cursor-pointer text-center',
        scopeFilter === scope
          ? 'bg-brand text-white'
          : 'bg-white text-brand hover:bg-blue-100',
        index === 0 ? 'rounded-l-md' : '',
        index === 2 ? 'rounded-r-md' : '',
        index !== 0 ? '-ml-px' : '',
      ]"
      @click="$emit('update:scopeFilter', scope)"
    >
      <!-- ['all', 'upcoming', 'past'] เวลาพวกค่าที่อยู่ใน HTML อย่าประกาศค่าเอาไว้เเบบนี้มใันจะดูยาก เเละรก
    ไปสร้างตัวเเปรเอาไว้ใน function เเล้วค่อยใช้ เช่น
    const tabs = ['all', 'upcoming', 'past'];
     v-for="(tab, index) in tabs"
     -->

      <!--  :class="[
        'flex-1 px-6 py-2 border font-bold transition cursor-pointer text-center',
        scopeFilter === scope
          ? 'bg-brand text-white'
          : 'bg-white text-brand hover:bg-blue-100',
        index === 0 ? 'rounded-l-md' : '',
        index === 2 ? 'rounded-r-md' : '',
        index !== 0 ? '-ml-px' : '',
      ]"
      
      ถ้าเงื่อนไขมากกว่า 1 , 2 ตามความเหมาะสม มากกว่านั้นสร้าง function ไปจัดการเเล้วค่อยคืนค่ามา
      เช่น classTabs(tab, index)
       -->
      {{ tabLabel[scope] }}
    </button>
  </div>
</template>

<script setup>
defineProps({ scopeFilter: String });
defineEmits(["update:scopeFilter"]);
const tabLabel = { all: "All", upcoming: "Upcoming", past: "Launched" };

// ปกติเเล้วเราจะไม่เก็บ key ที่ไม่ตรงกัน ควรเป็นแบบนี้
// const tabs = { name: "All", name: "Upcoming", name: "Launched" };
// เพราะ ใน html เวลามัน loop (v-for="(tab, index) in tabs") ก็ดึง
// tab.name ไปใช้ได้เลย จะไม่ใช้ tabLabel[scope] แบบนี้
</script>
