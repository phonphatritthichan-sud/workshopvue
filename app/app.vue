<template>
  <div class="app">
    <header>
      <!-- ส่วนหัวของหน้าเว็บ ใช้เก็บ title, tabs และ controls -->
      <h1>🚀 SpaceX Launches</h1>
      <!-- แสดงหัวข้อใหญ่ของหน้า -->
      <div class="tabs">
        <!-- กล่องเก็บปุ่ม tabs (all, upcoming, past) -->
        <button
          v-for="s in ['all', 'upcoming', 'past']"
          :key="s"
          :class="['tab', scope === s && 'active']"
          @click="scope = s"
        >
          {{ tabLabel[s] }}
        </button>

        <!-- วน loop สร้างปุ่มจาก array ที่มีค่า 'all', 'upcoming', 'past' -->

        <!-- :key="s"-->
        <!--กำหนด key เพื่อให้ Vue track element แต่ละตัวได้ -->
        <!-- Vue ต้องใช้ key เพื่อ track ว่า element ไหนถูกแก้ไข, เพิ่ม, หรือลบ -->
        <!-- ถ้าเขียนแบบไม่ใส่ : → key="s" Vue จะมองว่า "s" คือ string ธรรมดา ไม่ใช่ตัวแปร -->
        <!--ทุกปุ่มจะได้ key="s" เหมือนกันหมด → Vue แยกไม่ออกว่า element ไหนคือของจริง อาจมี bug เวลา render -->

        <!-- :class="['tab', scope === s && 'active']" -->
        <!-- เป็นการ bind class แบบ dynamic (ใช้ : เป็น shorthand ของ v-bind) -->
        <!-- ใส่ class "tab" และถ้า scope ตรงกับ s ให้เพิ่ม "active" -->
        <!-- : บอกว่า class นี้ผูกกับ expression (โค้ด JavaScript) -->
        <!-- ['tab', scope === s && 'active'] → เป็น array ที่ Vue จะประเมินค่าออกมาเป็น class string -->
        <!-- ถ้าเขียนโดยไม่ใส่ : Vue จะไม่ประเมิน expression -->

        <!-- @click="scope = s" -->
        <!-- @click คือ event binding (ย่อจาก v-on:click) -->
        <!-- เมื่อคลิกปุ่ม จะเปลี่ยนค่าตัวแปร scope เป็น s -->
        <!-- เวลากด จะรันโค้ด JavaScript ที่อยู่ใน "" -->
        <!-- ตรงนี้ไม่จำเป็นต้องใส่ : เพราะ @ เองก็คือ dynamic event handler อยู่แล้ว -->

        <!-- สรุป : (shorthand ของ v-bind) = บอก Vue ว่า ค่าใน attribute นี้มาจากตัวแปร/โค้ด JS ไม่ใช่ string ธรรมดา -->
        <!-- ถ้าไม่ใส่ : Vue จะมองว่าเป็น ข้อความคงที่ → ทำให้โค้ด dynamic ไม่ทำงาน -->

        <!-- {{ tabLabel[s] }} -->
        <!-- แสดง label ของแต่ละ tab จาก object tabLabel เช่น { all: "ทั้งหมด", upcoming: "กำลังจะปล่อย", past: "ที่แล้ว" } -->
      </div>
      <div class="controls">
        <!-- กล่องเก็บ input และ select สำหรับค้นหาและเรียงลำดับ -->
        <input v-model="q" type="search" placeholder="ค้นหาด้วยชื่อ…" />
        <!-- กล่องค้นหาแบบ text ใช้ v-model ผูกค่ากับตัวแปร q เวลาผู้ใช้พิมพ์ ระบบจะอัพเดต q อัตโนมัติ -->
        <select v-model="sortBy">
          <!-- dropdown สำหรับเลือกว่าจะเรียงลำดับตามอะไร -->
          <option value="date">เรียงตามเวลา</option>
          <!-- ตัวเลือกเรียงตามเวลา -->
          <option value="name">เรียงตามชื่อ</option>
          <!-- ตัวเลือกเรียงตามชื่อ -->
        </select>
        <select v-model="order">
          <!-- dropdown สำหรับเลือกทิศทางการเรียง -->
          <option value="desc">มาก → น้อย</option>
          <!-- เรียงจากมากไปน้อย (ใหม่ → เก่า, A → Z กลับกัน) -->
          <option value="asc">น้อย → มาก</option>
          <!-- เรียงจากน้อยไปมาก (เก่า → ใหม่, A → Z) -->
        </select>
      </div>
    </header>

    <main>
      <div v-if="loading">กำลังโหลดข้อมูล…</div>
      <!-- // แสดงข้อความ "กำลังโหลดข้อมูล…" ถ้าตัวแปร loading เป็น true -->
      <!-- // ถ้า loading เป็น false ส่วนนี้จะไม่ถูก render -->

      <article
        v-for="l in filtered"
        :key="l.id"
        class="card"
        @click="openModal(l)"
      >
        <!-- วนลูปผ่าน array ที่ชื่อ filtered และใช้ l เป็นตัวแทนแต่ละ item -->
        <!-- ใส่ key แบบ dynamic (ตาม id ของ item) เพื่อให้ Vue track node ได้ดีขึ้น -->
        <!-- ใส่ class "card" ให้ article แต่ละตัว ใช้สำหรับ styling -->
        <!-- ผูก event เมื่อคลิกที่การ์ด จะเรียกฟังก์ชัน openModal(l) -->

        <div class="thumb">
          <!-- ส่วนของรูปภาพ thumbnail -->
          <img v-if="pickImage(l.links)" :src="pickImage(l.links)" />
          <!-- // ถ้า pickImage(l.links) มีค่าจริง → แสดงรูป โดยใส่ src เป็นค่าที่ฟังก์ชัน pickImage คืนมา -->
          <span v-else>ไม่มีรูปภาพ</span>
          <!-- ถ้า pickImage(l.links) ไม่มีค่า → แสดงข้อความ "ไม่มีรูปภาพ" -->
        </div>
        <div class="content">
          <!-- ส่วนที่เก็บรายละเอียดข้อมูลใน card -->
          <div class="header">
            <!-- ส่วนหัวของ card (ใช้จัด layout ของชื่อ + meta) -->
            <h2 class="name">{{ l.name }}</h2>
            <!-- แสดงชื่อ launch (ใช้ data binding {{ }} ดึงค่ามาจาก l.name) -->
            <div class="meta">
              <!-- พื้นที่ด้านขวา แสดงวันที่และสถานะ (badge) -->
              <!-- แสดงจำนวน crew ถ้ามี -->
              <span v-if="l.crew?.length" class="crew-badge"
                >{{ l.crew.length }} crews</span
              >
              <span class="badge">{{ formatDate(l.date_utc) }}</span>
              <!-- แสดงวันที่ของ launch โดยผ่านฟังก์ชัน formatDate -->

              <span
                class="state_upcoming"
                v-html="badge(l.success, l.upcoming)"
              ></span>
              <!-- แสดง badge สถานะ launch โดยใช้ v-html -->
              <!-- // badge(l.success, l.upcoming) จะคืนค่าเป็น HTML string -->
              <!-- // เช่น <span class="badge ok">Launched</span> -->
            </div>
          </div>
        </div>
      </article>
    </main>

    <!-- Modal popup -->
    <div v-if="show" class="modal-backdrop" @click.self="closeModal">
      <!-- div หลักของ modal 
       - v-if="show" → แสดง modal ก็ต่อเมื่อค่า show = true 
       - class="modal-backdrop" → ใส่สไตล์พื้นหลังดำโปร่งแสง 
       - @click.self="closeModal" → ถ้าคลิกตรง backdrop (ไม่ใช่ใน modal) จะปิด modal -->

      <div class="modal">
        <!-- กล่อง modal หลักที่อยู่ตรงกลางหน้าจอ -->

        <header class="modal-header">
          <!-- ส่วนหัวของ modal (แถบด้านบน) -->
          <h2>{{ modal?.name }}</h2>
          <!-- แสดงชื่อของ launch โดยใช้ optional chaining (?.) กัน error กรณี modal ไม่มีค่า -->
          <!-- (2.1 ชื่อ) -->

          <button class="close-btn" @click="closeModal">✖</button>
          <!-- ปุ่มปิด modal ที่มุมขวา -->
        </header>

        <div class="modal-content">
          <!-- เนื้อหาภายใน modal -->

          <!-- 2.2 เวลา -->
          <p class="section">
            {{ new Date(modal.date_utc).toString() }}
          </p>
          <!-- แสดงวัน-เวลาในการปล่อย โดยแปลงจาก modal.date_utc เป็น Date แล้วแสดงเป็น string -->

          <!-- 2.3 หัวข้อนักบิน -->
          <h3 class="section-title">Crews</h3>
          <!-- หัวข้อ "Crews" -->

          <!-- 2.4 รูปและชื่อของนักบิน -->
          <div class="crew-list">
            <!-- กล่องเก็บ list ของ crew -->
            <div v-for="cid in modal.crew" :key="cid" class="crew-item">
              <!-- วน loop รายชื่อนักบิน (crew) ตาม id ที่อยู่ใน modal.crew -->
              <img
                v-if="crew[cid]?.image"
                :src="crew[cid].image"
                :alt="crew[cid]?.name"
              />
              <!-- แสดงรูปนักบิน ถ้ามี image -->
              <span>{{ crew[cid]?.name }}</span>
              <!-- แสดงชื่อนักบิน -->
            </div>
            <span v-if="!modal.crew?.length">—</span>
            <!-- ถ้าไม่มี crew เลย (ความยาว array = 0) จะแสดงขีด "—" -->
          </div>

          <!-- 2.5 หัวข้อจรวด -->
          <h3 class="section-title">Rockets</h3>
          <!-- หัวข้อ "Rockets" -->

          <!-- 2.6 ชื่อจรวด -->
          <p class="section">{{ rockets[modal.rocket]?.name || "—" }}</p>
          <!-- แสดงชื่อจรวด โดยใช้ rockets[modal.rocket].name ถ้าไม่มีจะแสดง "—" -->

          <!-- 2.7 รูปจรวด -->
          <div
            v-if="rockets[modal.rocket]?.flickr_images?.length"
            class="rocket-image"
          >
            <!-- ถ้ามีรูปจรวดใน flickr_images -->
            <img
              :src="rockets[modal.rocket].flickr_images[0]"
              alt="Rocket Image"
            />
            <!-- แสดงรูปแรกของจรวด -->
          </div>

          <!-- 2.8 หัวข้อพื้นที่ปล่อย -->
          <h3 class="section-title">Launchpad</h3>
          <!-- หัวข้อ "Launchpad" -->

          <!-- 2.9 สถานที่ปล่อย -->
          <p class="section">{{ pads[modal.launchpad]?.full_name || "—" }}</p>
          <!-- แสดงชื่อเต็มของ launchpad ถ้าไม่มีจะแสดง "—" -->

          <!-- 2.10 หัวข้อรายละเอียด -->
          <h3 class="section-title">Details</h3>
          <!-- หัวข้อ "Details" -->

          <!-- 2.11 รายละเอียด -->
          <p class="section">{{ modal.details || "—" }}</p>
          <!-- แสดงรายละเอียดการปล่อย ถ้าไม่มีข้อมูลจะใช้ "—" -->
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
// นำเข้า function หลักจาก Vue Composition API
// - ref ใช้สร้างตัวแปร reactive (ค่าที่ Vue จะ track การเปลี่ยนแปลง)
// - computed ใช้สร้างค่าที่คำนวณได้อัตโนมัติเมื่อ dependency เปลี่ยน
// - onMounted ใช้กำหนดโค้ดที่จะทำงานหลังจาก component render เสร็จแล้ว

const API = "https://api.spacexdata.com/v4"; // กำหนด URL ของ SpaceX API เวอร์ชัน v4 เอาไว้เรียกข้อมูล
const launches = ref([]); // ตัวแปร reactive เก็บข้อมูลการปล่อยจรวด (launches) เป็น array เริ่มต้นว่าง []
const rockets = ref({}); // ตัวแปร reactive เก็บข้อมูลจรวด (rockets) โดยเก็บในรูปแบบ object
const pads = ref({}); // ตัวแปร reactive เก็บข้อมูลพื้นที่ปล่อย (launchpads) เก็บในรูปแบบ object
const crew = ref({}); // ตัวแปร reactive เก็บข้อมูลนักบินอวกาศ (crew) ในรูปแบบ object
const loading = ref(true); // ตัวแปร reactive บอกสถานะว่ากำลังโหลดข้อมูลอยู่หรือไม่ (ค่าเริ่มต้น true)
const q = ref(""); // ตัวแปร reactive สำหรับเก็บข้อความที่ผู้ใช้พิมพ์ค้นหา (query string)
const scope = ref("all"); // ตัวแปร reactive สำหรับเก็บ scope หรือ filter (ค่าเริ่มต้น "all")
const sortBy = ref("date"); // ตัวแปร reactive สำหรับเก็บเงื่อนไขการเรียงข้อมูล (ค่าเริ่มต้นตาม "date")
const order = ref("desc"); // ตัวแปร reactive สำหรับเก็บลำดับการเรียง (descending = จากใหม่ไปเก่า)
const show = ref(false); // ตัวแปร reactive สำหรับควบคุมการแสดงผล popup/modal (false = ซ่อน)
const modal = ref(null); // ตัวแปร reactive สำหรับเก็บข้อมูล launch ที่ถูกเลือกมาแสดงใน modal

const tabLabel = {
  all: "All", // กำหนด key = all ให้แสดงข้อความ "All"
  upcoming: "upcoming", // กำหนด key = upcoming ให้แสดงข้อความ "upcoming"
  past: "Launched", // กำหนด key = past ให้แสดงข้อความ "Launched"
};

const formatDate = (iso) =>
  new Date(iso).toLocaleString(
    undefined, //แปลงข้อความ iso ให้เป็น object วันที่ของ JavaScript
    // แปลงวันที่ให้เป็น string ตามรูปแบบท้องถิ่นของ browser ผู้ใช้ (undefined หมายถึงใช้ locale เริ่มต้นของระบบ)
    {
      dateStyle: "medium", //กำหนดรูปแบบการแสดงผล วันที่แบบย่อ และเวลาแบบสั้น เช่น "Aug 22, 2025, 9:00 PM"
      timeStyle: "short",
      //ฟังก์ชันนี้ทำหน้าที่ แปลงวันที่ ISO → วันที่อ่านง่ายสำหรับผู้ใช้
    }
  );

const badge = (status, upcoming) => {
  if (upcoming) return `<span class='badge warn'>upcoming</span>`;
  // ถ้า upcoming เป็น true → คืนค่า <span> ที่มี class 'badge warn' และข้อความ "upcoming"
  if (status === true) return `<span class='badge ok'>launches</span>`;
  // ถ้า status เป็น true (ภารกิจสำเร็จ) → คืนค่า <span> ที่มี class 'badge ok' และข้อความ "launches"
  if (status === false) return `<span class='badge bad'>ล้มเหลว</span>`;
  // ถ้า status เป็น false (ภารกิจล้มเหลว) → คืนค่า <span> ที่มี class 'badge bad' และข้อความ "ล้มเหลว"
  return `<span class='badge'>ไม่ทราบ</span>`;
  // ถ้าไม่เข้าเงื่อนไขใดเลย → คืนค่า <span> ที่มี class 'badge' และข้อความ "ไม่ทราบ"
};

const pickImage = (links) => {
  if (!links) return "";
  // ถ้าไม่มีค่า links (เป็น null หรือ undefined) → คืนค่าว่าง ""
  if (links.patch?.small) return links.patch.small;
  // ถ้าใน links มีค่า patch.small → คืน URL ของ patch ขนาดเล็ก (โลโก้/สัญลักษณ์)
  if (links.flickr?.original?.length) return links.flickr.original[0];
  // ถ้าใน links มี flickr.original ที่เป็น array และมีรูป → คืนรูปแรกจาก array
  return "";
  // ถ้าไม่เข้าเงื่อนไขใดเลย → คืนค่าว่าง ""
};

const filtered = computed(() => {
  //ประกาศตัวแปร filtered เป็น computed property (ค่าที่จะคำนวณใหม่อัตโนมัติเมื่อ dependency เปลี่ยน เช่น launches.value, q.value, scope.value ฯลฯ)
  let list = launches.value.filter(
    (
      l //กำหนดตัวแปร list โดยใช้ .filter() เพื่อกรองข้อมูลจาก launches.value (ข้อมูลการปล่อยยานทั้งหมด)
    ) => {
      if (scope.value === "upcoming" && !l.upcoming) return false;
      // ถ้าเลือกดู scope = "upcoming" แต่ launch นี้ไม่ใช่ upcoming → ตัดออก
      if (scope.value === "past" && l.upcoming) return false;
      // ถ้าเลือกดู scope = "past" แต่ launch นี้ยัง upcoming อยู่ → ตัดออก
      if (q.value && !l.name.toLowerCase().includes(q.value.toLowerCase()))
        return false; // ถ้ามีการค้นหา (q.value ไม่ว่าง) และชื่อ launch ไม่ตรงกับคำค้นหา → ตัดออก
      return true; // ถ้าไม่เข้าเงื่อนไขตัดออก → เก็บ launch นี้ไว้ใน list
    }
  );
  list.sort((a, b) => {
    //นำ list ที่ผ่านการกรองแล้วมาจัดเรียง (sort)
    if (sortBy.value === "name") {
      //ถ้าต้องการเรียงตามชื่อ → เช็คค่าที่ผู้ใช้เลือก
      return order.value === "asc"
        ? a.name.localeCompare(b.name)
        : b.name.localeCompare(a.name);
      // ถ้า order = "asc" → เรียงชื่อ A → Z
      // ถ้า order = "desc" → เรียงชื่อ Z → A
    } else {
      const da = new Date(a.date_utc).getTime();
      const db = new Date(b.date_utc).getTime();
      return order.value === "asc" ? da - db : db - da;
      // ถ้าไม่ได้เรียงตามชื่อ → จะเรียงตามวันเวลา (date_utc)
      // new Date(...).getTime() แปลงวันที่เป็นตัวเลข timestamp (ms) เพื่อใช้เปรียบเทียบ
    }
  });
  return list;
  // หลังจากกรองและเรียงเรียบร้อยแล้ว → คืนค่า list ที่ผ่านการจัดการแล้ว
  // เนื่องจาก filtered เป็น computed → ค่านี้จะอัปเดตอัตโนมัติทุกครั้งที่ launches.value, q.value, scope.value, sortBy.value, หรือ order.value เปลี่ยน
});

const fetchByIds = async (endpoint, ids) => {
  // ประกาศฟังก์ชันแบบ async ชื่อ fetchByIds
  // รับพารามิเตอร์ 2 ตัว:
  //   endpoint = ชื่อ resource เช่น "crew", "rockets"
  //   ids = เซตของ id ที่ต้องการดึงข้อมูล
  if (!ids.size) return {};
  // ถ้า ids ไม่มีข้อมูล (size = 0) → คืนค่า object ว่าง {} ทันที
  // ป้องกันการยิง API โดยไม่จำเป็น
  const body = {
    query: { _id: { $in: Array.from(ids) } },
    options: { pagination: false },
    // สร้าง body สำหรับ request
    // query: กรองข้อมูลโดยใช้ _id ที่อยู่ใน ids (แปลง Set → Array ด้วย Array.from)
    // options: ปิดการแบ่งหน้า (pagination) → ดึงข้อมูลทั้งหมดที่ตรงเงื่อนไข
  };
  const res = await fetch(`${API}/${endpoint}/query`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(body),
    // เรียก API โดยใช้ fetch
    // URL = `${API}/${endpoint}/query` เช่น "https://api.spacexdata.com/v4/crew/query"
    // ใช้ POST ส่ง body ที่เตรียมไว้
    // headers: ระบุว่าเป็น JSON
  });
  const json = await res.json();
  // แปลงผลลัพธ์จาก response (res) ให้เป็น object JSON
  const map = {};
  // ประกาศ object ว่างไว้เก็บข้อมูลในรูปแบบ key-value (id → document)
  (json.docs || []).forEach((d) => (map[d.id] = d));
  // วนลูปใน json.docs (หรือถ้าไม่มี docs → ใช้ array ว่าง [])
  // map[d.id] = d → เก็บ document แต่ละตัว โดยใช้ id เป็น key
  // เช่น { "abc123": {id: "abc123", name: "..."} }
  return map;
  // คืนค่า object ที่เก็บข้อมูลเป็นรูปแบบ key-value
};

const bootstrap = async () => {
  // ประกาศฟังก์ชัน async ชื่อ bootstrap (เอาไว้โหลดข้อมูลครั้งแรก)
  const res = await fetch(`${API}/launches`);
  // เรียก API ที่ endpoint /launches (เช่น "https://api.spacexdata.com/v4/launches")
  // ได้ค่า response กลับมาเก็บใน res
  const data = await res.json();
  // แปลง response (res) ให้อยู่ในรูปแบบ JSON แล้วเก็บในตัวแปร data
  // data ตอนนี้คือ array ของ launch objects
  launches.value = data;
  // เก็บข้อมูล launches ที่ได้มาไว้ใน state (reactive) ชื่อ launches
  // .value → เพราะ launches ถูกสร้างจาก ref() ของ Vue

  const rocketIds = new Set();
  const padIds = new Set();
  const crewIds = new Set();
  // สร้าง Set ว่างไว้ 3 ตัว สำหรับเก็บ id ที่ไม่ซ้ำ:
  // rocketIds = เก็บรหัส rocket
  // padIds = เก็บรหัส launchpad
  // crewIds = เก็บรหัส crew
  data.forEach((l) => {
    if (l.rocket) rocketIds.add(l.rocket);
    if (l.launchpad) padIds.add(l.launchpad);
    if (Array.isArray(l.crew)) l.crew.forEach((id) => crewIds.add(id));
    // วนลูปข้อมูลแต่ละ launch (l)
    // - ถ้ามี rocket → เก็บ id ของ rocket ลงใน rocketIds
    // - ถ้ามี launchpad → เก็บ id ของ launchpad ลงใน padIds
    // - ถ้ามี crew (เป็น array) → วนลูปเก็บแต่ละ id ลงใน crewIds
    // ใช้ Set เพื่อป้องกัน id ซ้ำกัน
  });

  rockets.value = await fetchByIds("rockets", rocketIds);
  // ใช้ฟังก์ชัน fetchByIds ไปดึงข้อมูล rocket ตาม id ที่อยู่ใน rocketIds
  // แล้วเก็บเป็น map (id → rocket object) ไว้ใน state rockets
  pads.value = await fetchByIds("launchpads", padIds);
  // ดึงข้อมูล launchpad ตาม id ที่อยู่ใน padIds
  // แล้วเก็บใน state pads
  crew.value = await fetchByIds("crew", crewIds);
  // ดึงข้อมูล crew ตาม id ที่อยู่ใน crewIds
  // แล้วเก็บใน state crew

  loading.value = false;
  // เมื่อโหลดข้อมูลทั้งหมดเสร็จ → เปลี่ยน state loading เป็น false
  // เพื่อบอก UI ว่าโหลดข้อมูลเรียบร้อยแล้ว
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
  background: rgb(233, 233, 233);
  min-height: 100vh;
}
header {
  padding: 16px;
  background: rgb(233, 233, 233);
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
  border: 1px solid #4d36ca;
  border-radius: 6px;
  background-color: white;
  color: #4d36ca;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.2s;
}

.tab:hover {
  background-color: #e6f0ff; /* ฟ้าอ่อนเมื่อ hover */
}

.tab.active {
  background-color: #4d36ca; /* น้ำเงิน */
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
  width: 70%;
  margin: 0 auto 12px; /* 🟢 กึ่งกลางแนวนอน + margin-bottom */
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
  flex: 0 0 40px;
  height: 40px;
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
  font-size: 0.9rem;
  font-weight: bold;
  margin: 0;
  color: #333;
}

.meta {
  display: flex;
  gap: 8px;
  font-size: 0.85rem;
  color: #000000;
}

.state_upcoming {
  font-weight: bold; /* ตัวหนา */
  color: #4d36ca; /* ตัวอักษรสีน้ำเงิน */
}

.badge {
  background: #eee;
  padding: 2px 8px;
  border-radius: 4px;
}
.crew-badge {
  background-color: #4d36ca; /* พื้นหลังสีน้ำเงิน */
  color: white; /* ตัวอักษรสีขาว */
  font-weight: bold;
  padding: 2px 8px;
  border-radius: 12px; /* ทำเป็นแคปซูล */
  font-size: 0.85rem;
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
/* พื้นหลังมืด */
.modal-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 50;
}

/* กล่อง popup */
.modal {
  background: #fff;
  width: 70%;
  max-height: 90vh;
  overflow-y: auto;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.3);
  display: flex;
  flex-direction: column;
  gap: 16px;
  align-items: center; /* 🟢 จัดให้เนื้อหาชิดกลาง */
  text-align: center; /* 🟢 ตัวหนังสืออยู่กลาง */
}

/* หัวข้อด้านบน */
.modal-header {
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
  padding-bottom: 8px;
  background-color: #ffffff;
}

.modal-header h2 {
  margin: 0;
  font-size: 1.4rem;
  font-weight: bold;
  color: #000000;
  background-color: ;
  text-align: center;
}

.close-btn {
  position: absolute; /* 🟢 วางปุ่ม close อิสระ */
  right: 0;
  top: 50%;
  transform: translateY(-50%);
  background: none;
  border: none;
  font-size: 1.2rem;
  cursor: pointer;
}

/* section */
.section-title {
  display: inline-block; /* ทำให้ขนาดพอดีกับข้อความ */
  padding: 4px 12px; /* ระยะห่างด้านบน/ล่าง, ซ้าย/ขวา */
  border-radius: 999px; /* ทำเป็นแคปซูลเต็ม ๆ */
  background-color: #4d36ca; /* พื้นหลังสีน้ำเงิน */
  color: white; /* ตัวอักษรสีขาว */
  /* font-weight: bold; */
  font-size: 1rem;
  margin: 8px 0;
}

.section {
  margin: 4px 0;
  font-size: 1rem;
  color: #334155;
}

/* รูปจรวด */
.rocket-image {
  margin: 10px 0;
}
.rocket-image img {
  max-width: 100%;
  border-radius: 8px;
}

/* Crew */
.crew-list {
  display: flex;
  flex-wrap: wrap;
  gap: 50px;
  margin-top: 8px;
  justify-content: center; /* 🟢 จัดให้นักบินอยู่กลาง */
  color: #000000;
}

.crew-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: auto;
  gap: 4px;
}

.crew-item img {
  width: 60px;
  height: 60px;
  object-fit: cover;
  border-radius: 50%;
  border: 2px solid #334155;
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
