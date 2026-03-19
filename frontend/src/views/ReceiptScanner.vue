<template>
  <div class="receipt-scanner-page">
    <!-- 模式选择 -->
    <div v-if="includeUnassigned === null" class="mode-dialog">
      <div class="mode-card">
        <h2 class="mode-title">选择扫描模式</h2>
        <div class="mode-btns">
          <button class="btn-mode primary" @click="handleConfirmMode(true)">开始扫描</button>
          <button class="btn-mode ghost" @click="$router.push('/')">取消</button>
        </div>
      </div>
    </div>

    <!-- 无小票 -->
    <div v-else-if="receipts.length === 0" class="mode-dialog">
      <div class="mode-card">
        <h3>没有可扫描的小票</h3>
        <button class="btn-mode primary" @click="$router.push('/')">返回列表</button>
      </div>
    </div>

    <!-- 扫描展示 -->
    <template v-else>
      <div class="scanner-main">
        <div class="scanner-header">
          <button @click="$router.push('/')">返回</button>
          <span>{{ currentIndex + 1 }} / {{ receipts.length }}</span>
        </div>

        <div class="code-area">
          <swiper :slides-per-view="1" @slideChange="onSlideChange" @swiper="onSwiperInit">
            <swiper-slide v-for="(receipt, index) in receipts" :key="receipt.id">
              <div class="code-card" @click="toggleCodeType">
                <canvas :ref="el => setBarcodeRef(el, index)" v-show="codeType === 'barcode'"></canvas>
                <canvas :ref="el => setQrRef(el, index)" v-show="codeType === 'qr'"></canvas>
                <p>{{ receipt.ticketNumber }}</p>
              </div>
            </swiper-slide>
          </swiper>
        </div>

        <div class="scanner-footer">
          <button @click="currentIndex--" :disabled="currentIndex === 0">上一个</button>
          <button @click="markAbnormal" class="btn-warn">标记异常</button>
          <button @click="currentIndex++" :disabled="currentIndex === receipts.length - 1">下一个</button>
        </div>
      </div>
    </template>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, nextTick } from "vue";
import { useRoute } from "vue-router";
import { Swiper, SwiperSlide } from "swiper/vue";
import "swiper/css";
import QRCode from "qrcode";
import JsBarcode from "jsbarcode";
import { loadData, updateReceipt } from "../utils/receiptStorage.js";

const route = useRoute();
const ownerId = computed(() => route.params.ownerId || "");
const receipts = ref([]);
const currentIndex = ref(0);
const includeUnassigned = ref(null);
const codeType = ref("barcode");
const qrRefs = ref([]);
const barcodeRefs = ref([]);
const swiperInstance = ref(null);

const setQrRef = (el, index) => { if (el) qrRefs.value[index] = el; };
const setBarcodeRef = (el, index) => { if (el) barcodeRefs.value[index] = el; };
const onSwiperInit = (s) => { swiperInstance.value = s; };
const onSlideChange = (s) => { currentIndex.value = s.activeIndex; };

watch(currentIndex, (val) => {
  if (swiperInstance.value && swiperInstance.value.activeIndex !== val) swiperInstance.value.slideTo(val);
});

const handleConfirmMode = (inc) => { includeUnassigned.value = inc; };
const toggleCodeType = () => { codeType.value = codeType.value === "qr" ? "barcode" : "qr"; };

const generateCodes = async () => {
  await nextTick();
  receipts.value.forEach(async (r, i) => {
    try {
      if (qrRefs.value[i]) await QRCode.toCanvas(qrRefs.value[i], r.ticketNumber, { width: 300 });
      if (barcodeRefs.value[i]) JsBarcode(barcodeRefs.value[i], r.ticketNumber, { format: "CODE128", width: 2, height: 100 });
    } catch (e) {}
  });
};

const markAbnormal = () => {
  const r = receipts.value[currentIndex.value];
  if (r) {
    updateReceipt(r.id, { isAbnormal: true });
    r.isAbnormal = true;
    if (currentIndex.value < receipts.value.length - 1) currentIndex.value++;
  }
};

const loadReceipts = () => {
  if (includeUnassigned.value === null) return;
  const data = loadData();
  let filtered = data.receipts;
  if (ownerId.value) {
    filtered = filtered.filter(r => r.owner === ownerId.value || (includeUnassigned.value && !r.owner));
  } else {
    filtered = filtered.filter(r => !r.owner);
  }
  receipts.value = filtered;
  generateCodes();
};

watch(includeUnassigned, loadReceipts);
onMounted(loadReceipts);
</script>

<style scoped>
.receipt-scanner-page { height: 100vh; display: flex; flex-direction: column; background: #f3f4f6; }
.mode-dialog { flex: 1; display: flex; align-items: center; justify-content: center; }
.mode-card { background: white; padding: 2rem; border-radius: 1rem; text-align: center; }
.scanner-main { flex: 1; display: flex; flex-direction: column; }
.scanner-header { padding: 1rem; background: white; display: flex; justify-content: space-between; }
.code-area { flex: 1; display: flex; align-items: center; justify-content: center; padding-bottom: 80px; }
.code-card { background: white; padding: 2rem; border-radius: 1rem; text-align: center; }
.scanner-footer { position: fixed; bottom: 0; width: 100%; display: flex; padding: 1rem; background: white; gap: 1rem; }
.scanner-footer button { flex: 1; padding: 0.75rem; border-radius: 0.5rem; }
.btn-warn { background: #ef4444; color: white; border: none; }
</style>