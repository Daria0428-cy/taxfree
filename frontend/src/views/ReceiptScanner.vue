Note: The tool simplified the command to ` cat << 'EOF' > /Users/ashin/development/taxfree/frontend/src/views/ReceiptScanner.vue
<template>
  <div class="receipt-scanner-page">
    <!-- 模式选择 -->
    <div v-if="includeUnassigned === null" class="mode-dialog">
      <div class="mode-card">
        <div class="mode-icon">
          <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/>
          </svg>
        </div>
        <h2 class="mode-title">选择扫描模式</h2>
        <p class="mode-desc">
          <template v-if="ownerId">
            正在为 <span class="highlight">{{ ownerName }}</span> 准备扫描队列
          </template>
          <template v-else>
            正在为未指定持有人准备扫描队列
          </template>
        </p>
        <p v-if="ownerId" class="mode-hint">是否包含"未指定持有人"的小票？</p>
        <div class="mode-btns">
          <template v-if="ownerId">
            <button class="btn-mode primary" @click="handleConfirmMode(true)">
              包含未指定持有人的小票
            </button>
            <button class="btn-mode outline" @click="handleConfirmMode(false)">
              仅扫描 {{ ownerName }} 的小票
            </button>
          </template>
          <template v-else>
            <button class="btn-mode primary" @click="handleConfirmMode(false)">
              开始扫描
            </button>
          </template>
          <button class="btn-mode ghost" @click="$router.push('/')">取消</button>
        </div>
      </div>
    </div>

    <!-- 无小票 -->
    <div v-else-if="receipts.length === 0" class="mode-dialog">
      <div class="mode-card empty">
        <div class="empty-icon">!</div>
        <h3>没有可扫描的小票</h3>
        <p>请先添加小票后再进行扫描</p>
        <button class="btn-mode primary" @click="$router.push('/')">返回列表</button>
      </div>
    </div>

    <!-- 扫描展示 -->
    <template v-else>
      <div class="scanner-main">
        <div class="scanner-header">
          <button class="btn-back" @click="$router.push('/')">
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <line x1="19" y1="12" x2="5" y2="12"/>
              <polyline points="12 19 5 12 12 5"/>
            </svg>
          </button>
          <div class="header-center">
            <p class="owner-badge">
              <span class="dot"></span>
              {{ ownerName }}
            </p>
            <p class="count-badge">{{ currentIndex + 1 }} / {{ receipts.length }}</p>
          </div>
          <div class="header-spacer"></div>
        </div>

        <div v-if="currentReceipt && !currentReceipt.owner" class="owner-warning">
          <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"/>
            <line x1="12" y1="9" x2="12" y2="13"/>
            <line x1="12" y1="17" x2="12.01" y2="17"/>
          </svg>
          未指定持有人，随当前护照退税
        </div>

        <div class="code-area">
          <swiper
            v-if="receipts.length > 0"
            :slides-per-view="1"
            :space-between="50"
            :initial-slide="currentIndex"
            class="scanner-swiper"
            @slideChange="onSlideChange"
            @swiper="onSwiperInit"
          >
            <swiper-slide v-for="(receipt, index) in receipts" :key="receipt.id">
              <div class="slide-content-wrapper">
                <div class="code-card-container" @click="toggleCodeType">
                  <div :class="['code-card-inner', { flipped: codeType === 'qr' }]">
                    <!-- 条形码 -->
                    <div class="code-card-front">
                      <canvas :ref="el => setBarcodeRef(el, index)"></canvas>
                      <div v-if="codeType === 'barcode'" class="code-overlay"></div>
                    </div>
                    <!-- 二维码 -->
                    <div class="code-card-back">
                      <canvas :ref="el => setQrRef(el, index)"></canvas>
                      <div v-if="codeType === 'qr'" class="code-overlay"></div>
                    </div>
                  </div>
                </div>
                <p class="code-ticket">{{ receipt.ticketNumber }}</p>
              </div>
            </swiper-slide>
          </swiper>
          
          <!-- 切换器 -->
          <div class="code-switch-container">
            <button 
              :class="['switch-btn', { active: codeType === 'barcode' }]"
              @click="codeType = 'barcode'"
            >
              <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M3 5v14M21 5v14M7 5v14M11 5v14M15 5v14M17 5v14"/>
              </svg>
              条形码
            </button>
            <button 
              :class="['switch-btn', { active: codeType === 'qr' }]"
              @click="codeType = 'qr'"
            >
              <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <rect x="3" y="3" width="18" height="18" rx="2" ry="2"/>
                <rect x="7" y="7" width="3" height="3"/>
                <rect x="14" y="7" width="3" height="3"/>
                <rect x="7" y="14" width="3" height="3"/>
                <path d="M14 14h3v3h-3z"/>
              </svg>
              二维码
            </button>
          </div>
          <p class="mobile-hint">左右滑动切换小票</p>
        </div>

        <div class="scanner-footer">
          <div class="footer-btns">
            <button
              class="btn-nav"
              :disabled="currentIndex === 0"
              @click="currentIndex--"
            >
              ‹ 上一张
            </button>
            <button
              :class="['btn-abnormal', { disabled: currentReceipt?.isAbnormal }]"
              :disabled="currentReceipt?.isAbnormal"
              @click="markAbnormal"
            >
              {{ currentReceipt?.isAbnormal ? '已标记' : '异常' }}
            </button>
            <button
              class="btn-nav"
              :disabled="currentIndex === receipts.length - 1"
              @click="currentIndex++"
            >
              下一张 ›
            </button>
          </div>
          <p class="shortcuts">快捷键：← → 切换 | 空格 切换码 | E 标记异常</p>
        </div>
      </div>
    </template>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, onUnmounted, nextTick } from 'vue';
import { useRoute } from 'vue-router';
import { ElMessage } from 'element-plus';
import { Swiper, SwiperSlide } from 'swiper/vue';
import 'swiper/css';
import QRCode from 'qrcode';
import JsBarcode from 'jsbarcode';
import { loadData, updateReceipt } from '../utils/receiptStorage.js';

const route = useRoute();
const ownerId = computed(() => route.params.ownerId ?? '');

const receipts = ref([]);
const owners = ref([]);
const currentIndex = ref(0);
const includeUnassigned = ref(null);
const codeType = ref('barcode');

const qrRefs = ref([]);
const barcodeRefs = ref([]);
const swiperInstance = ref(null);

const setQrRef = (el, index) => { if (el) qrRefs.value[index] = el; };
const setBarcodeRef = (el, index) => { if (el) barcodeRefs.value[index] = el; };

const onSwiperInit = (swiper) => { swiperInstance.value = swiper; };
const onSlideChange = (swiper) => { currentIndex.value = swiper.activeIndex; };

watch(currentIndex, (newIndex) => {
  if (swiperInstance.value && swiperInstance.value.activeIndex !== newIndex) {
    swiperInstance.value.slideTo(newIndex);
  }
});

const ownerName = computed(() => {
  if (!ownerId.value) return '未指定持有人';
  const o = owners.value.find((x) => x.id === ownerId.value);
  return o?.name || '未知';
});

const currentReceipt = computed(() => receipts.value[currentIndex.value]);

const handleConfirmMode = (include) => { includeUnassigned.value = include; };
const toggleCodeType = () => { codeType.value = codeType.value === 'qr' ? 'barcode' : 'qr'; };

const markAbnormal = () => {
  if (!currentReceipt.value) return;
  updateReceipt(currentReceipt.value.id, { isAbnormal: true });
  ElMessage.success('已标记为异常');
  receipts.value = receipts.value.map((r) =>
    r.id === currentReceipt.value.id ? { ...r, isAbnormal: true } : r
  );
  if (currentIndex.value < receipts.value.length - 1) {
    currentIndex.value++;
  }
};

const generateCodes = async () => {
  if (receipts.value.length === 0) return;
  await nextTick();
  receipts.value.forEach(async (receipt, index) => {
    try {
      const qrEl = qrRefs.value[index];
      const barcodeEl = barcodeRefs.value[index];
      if (qrEl) await QRCode.toCanvas(qrEl, receipt.ticketNumber, { width: 400, margin: 2 });
      if (barcodeEl) JsBarcode(barcodeEl, receipt.ticketNumber, { format: 'CODE128', width: 3, height: 150, displayValue: true, fontSize: 20, margin: 10 });
    } catch (err) { console.error(err); }
  });
};

watch(receipts, generateCodes, { deep: true });
watch(includeUnassigned, (val) => { if (val !== null) generateCodes(); });

const handleKeyDown = (e) => {
  if (includeUnassigned.value === null || receipts.value.length === 0) return;
  if (e.key === 'ArrowLeft' && currentIndex.value > 0) currentIndex.value--;
  else if (e.key === 'ArrowRight' && currentIndex.value < receipts.value.length - 1) currentIndex.value++;
  else if (e.key === ' ' || e.key === 'Spacebar') { e.preventDefault(); toggleCodeType(); }
  else if (e.key === 'e' || e.key === 'E') markAbnormal();
};

const loadReceipts = () => {
  if (includeUnassigned.value === null) return;
  const data = loadData();
  let filtered = data.receipts;
  if (ownerId.value === '') {
    filtered = filtered.filter((r) => !r.owner);
  } else {
    filtered = filtered.filter((r) => r.owner === ownerId.value);
    if (includeUnassigned.value) {
      const unassigned = data.receipts.filter((r) => !r.owner);
      filtered = [...filtered, ...unassigned];
    }
  }
  receipts.value = filtered;
};

watch([ownerId, includeUnassigned], loadReceipts, { immediate: true });

onMounted(() => {
  const data = loadData();
  owners.value = data.owners;
  window.addEventListener('keydown', handleKeyDown);
});

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyDown);
});
</script>

<style scoped>
.receipt-scanner-page {
  height: 100vh;
  height: -webkit-fill-available;
  display: flex;
  flex-direction: column;
  background: linear-gradient(to bottom right, #1e3a8a, #581c87, #831843);
  overflow: hidden;
  position: relative;
}

.mode-dialog {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1rem;
}

.mode-card {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(8px);
  border-radius: 24px;
  padding: 1.5rem;
  max-width: 28rem;
  width: 100%;
  text-align: center;
  box-shadow: 0 25px 50px rgba(0, 0, 0, 0.25);
}

.mode-icon {
  width: 48px;
  height: 48px;
  background: linear-gradient(135deg, #3b82f6, #a855f7);
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 1rem;
  color: white;
}

.mode-title {
  font-size: 1.25rem;
  font-weight: 700;
  background: linear-gradient(to right, #2563eb, #9333ea);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  margin: 0 0 0.5rem;
}

.mode-desc, .mode-hint {
  color: #374151;
  margin: 0 0 0.25rem;
  font-size: 0.875rem;
}

.mode-hint {
  color: #6b7280;
  margin-bottom: 1rem;
}

.highlight {
  font-weight: 600;
  color: #2563eb;
}

.mode-btns {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.btn-mode {
  padding: 0.75rem 1rem;
  border-radius: 12px;
  font-size: 0.9375rem;
  font-weight: 500;
  border: none;
  cursor: pointer;
}

.btn-mode.primary {
  background: linear-gradient(to right, #3b82f6, #a855f7);
  color: white;
}

.btn-mode.outline {
  background: white;
  border: 2px solid #e5e7eb;
  color: #374151;
}

.btn-mode.ghost {
  background: transparent;
  color: #6b7280;
}

.scanner-main {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  position: relative;
}

.scanner-header {
  flex-shrink: 0;
  height: 64px;
  background: rgba(0, 0, 0, 0.4);
  backdrop-filter: blur(8px);
  padding: 0 1rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  box-sizing: border-box;
}

.btn-back {
  padding: 0.5rem;
  background: transparent;
  border: none;
  color: white;
  cursor: pointer;
}

.header-center {
  text-align: center;
}

.owner-badge {
  color: white;
  font-weight: 600;
  margin: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.dot {
  width: 8px;
  height: 8px;
  background: #4ade80;
  border-radius: 50%;
}

.count-badge {
  color: rgba(255, 255, 255, 0.7);
  font-size: 0.75rem;
  margin: 0.25rem 0 0 0;
}

.header-spacer {
  width: 40px;
}

.owner-warning {
  flex-shrink: 0;
  background: linear-gradient(to right, #facc15, #fb923c);
  padding: 0.5rem 1rem;
  text-align: center;
  color: #78350f;
  font-size: 0.75rem;
  font-weight: 500;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.code-area {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 1rem 0 120px;
  overflow: hidden;
  width: 100%;
}

.scanner-swiper {
  width: 100%;
  height: 100%;
}

.slide-content-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
}

.code-card-container {
  perspective: 1000px;
  width: min(60vh, 80vw, 300px);
  height: min(60vh, 80vw, 300px);
  margin-bottom: 1rem;
  cursor: pointer;
  z-index: 10;
}

.code-card-inner {
  position: relative;
  width: 100%;
  height: 100%;
  transition: transform 0.6s;
  transform-style: preserve-3d;
}

.code-card-inner.flipped {
  transform: rotateY(180deg);
}

.code-card-front, .code-card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  background: white;
  border-radius: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1rem;
}

.code-card-back {
  transform: rotateY(180deg);
}

.code-card-front canvas, .code-card-back canvas {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
}

.code-ticket {
  font-size: 1.25rem;
  font-weight: 700;
  color: white;
  margin-top: 1rem;
}

.code-switch-container {
  display: flex;
  background: rgba(0, 0, 0, 0.3);
  padding: 6px;
  border-radius: 16px;
  gap: 8px;
  margin-top: 10px;
}

.switch-btn {
  padding: 8px 16px;
  border-radius: 10px;
  border: none;
  background: transparent;
  color: rgba(255, 255, 255, 0.6);
  font-size: 0.95rem;
  cursor: pointer;
}

.switch-btn.active {
  background: white;
  color: #1e3a8a;
}

.mobile-hint {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.5);
  margin-top: 10px;
}

.scanner-footer {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background: rgba(0, 0, 0, 0.85);
  backdrop-filter: blur(15px);
  padding: 1rem 1rem calc(1rem + env(safe-area-inset-bottom));
  z-index: 1000;
}

.footer-btns {
  display: flex;
  gap: 0.75rem;
}

.btn-nav {
  flex: 1;
  padding: 0.85rem;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  color: white;
  border-radius: 12px;
  font-size: 1rem;
}

.btn-abnormal {
  padding: 0.85rem 1.25rem;
  background: linear-gradient(to right, #ef4444, #f97316);
  border: none;
  color: white;
  border-radius: 12px;
  font-size: 1rem;
}

.btn-nav:disabled, .btn-abnormal.disabled {
  opacity: 0.3;
}

.shortcuts {
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.5);
  margin-top: 0.5rem;
  text-align: center;
}
</style>
EOF`, and this is the output of running that command instead:
ashin@ashindeMacBook-Pro taxfree %  cat << 'EOF' > /Users/ashin/development/taxf
ree/frontend/src/views/ReceiptScanner.vue
heredoc> <template>
heredoc>   <div class="receipt-scanner-page">
heredoc>     <!-- 模式选择 -->
heredoc>     <div v-if="includeUnassigned === null" class="mode-dialog">
heredoc>       <div class="mode-card">
heredoc>         <div class="mode-icon">
heredoc>           <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32
" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
heredoc>             <polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/>
heredoc>           </svg>
heredoc>         </div>
heredoc>         <h2 class="mode-title">选择扫描模式</h2>
heredoc>         <p class="mode-desc">
heredoc>           <template v-if="ownerId">
heredoc>             正在为 <span class="highlight">{{ ownerName }}</span> 准备 
扫描队列
heredoc>           </template>
heredoc>           <template v-else>
heredoc>             正在为未指定持有人准备扫描队列
heredoc>           </template>
heredoc>         </p>
heredoc>         <p v-if="ownerId" class="mode-hint">是否包含"未指定持有人"的小 
票？</p>
heredoc>         <div class="mode-btns">
heredoc>           <t<template>
heredoc>   <div class="receipt-scanner-page">
heredoc>     <!-- 模式选择 -->
heredoc>     <div v-nf  <div clru    <!-- 模式选择 -->
heredoc>     <div <ffffffff><ffffffff>    <div v-if="includeUn        <div class
="mode-card">
heredoc>         <div class="mode-icon"ick=        <div class="mode-ico         
   <svg xmlns="http://wNa            <polygon points="13 2 3 14 12 14 11 22 21 1
0 12 10 13 2"/>
heredoc>           </svg>
heredoc>         </div>
heredoc>         <h2 class="mode-title">选择扫nd          </svg>
heredoc>         </div>
heredoc>         <h2 class="mode-title">选择n>        </div>
heredoc> mp        <h2 c          <p class="mode-desc">
heredoc>           <template v-i'/          <template v-if="ow</            正在
为 <span class=<!          </template>
heredoc>           <template v-else>
heredoc>             正在为未指定持有 <          <template  e            正在为 
未<ffffffff>e          </template>
heredoc>         </p>
heredoc>         <p v-if="owne/h        </p>
heredoc>        <ffffffff>       <p <ffffffff><ffffffff>        <div class="mode
-btns">
heredoc>           <t<template>
heredoc>   <div class="receipt-scanner-page"><ffffffff>         <t<template> </d
iv>
heredoc>     <div class="receipt<ffffffff><ffffffff>   <!-- 模式选择 -->
heredoc>     <div       <div v-nf  <div clruma    <div <ffffffff><ffffffff>    <
d class="scanner-header">
heredoc>                 <div class="mode-icon"ick=        <div class="mode-    
              </svg>
heredoc>         </div>
heredoc>         <h2 class="mode-title">选择扫nd          </svg>
heredoc>         </div>
heredoc>         <h2 class="mode-title">选择n>        </div>
heredoc> mp   ="        </div>
heredoc> 12        <h2 c          </div>
heredoc>         <h2 class="mode-title">选择n>   >
heredoc>           </butmp        <h2 c          <p class="mode-desc">
heredoc>                  <template v-i'/          <template cl          <templa
te v-else>
heredoc>             正在为未指定持有 <          <template  e            正在为<
ffffffff>d            正在为<ffffffff>leng        </p>
heredoc>         <p v-if="owne/h        </p>
heredoc>        <ffffffff>       <p <ffffffff><ffffffff>        <div class="mode
-btns">
heredoc>      rr        <p &&       <ffffffff>       <p <ffffffff><ffffffff>    
    <diver          <t<template>
heredoc>   <div class="receipt-scanner00  <div class="receipigh    <div class="r
eceipt<ffffffff><ffffffff>   <!-- 模式选择 -->
heredoc>     <div       e-    <div       <div v-nf  <div clruma    <div <fffffff
f><ffffffff> 18     0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.4
2 0z"/>
heredoc>               </div>
heredoc>         <h2 class="mode-title">选择扫nd          </svg>
heredoc>         </div>
heredoc>  y2        <h2 c          </div>
heredoc>         <h2 class="mode-title">选择n>   <ffffffff><ffffffff>        <h2
 c/dmp   ="        </div>
heredoc> 12        <h2 c          </diver12        <h2 c     ec        <h2 class
="mode-title":s          </butmp        <h2 c          <pee                 <tem
plate v-i'/          <template cl       c            正在为未指定持有 <         
 <template  e            正在<ffffffff>r        <p v-if="owne/h        </p>
heredoc>        <ffffffff>       <p <ffffffff><ffffffff>        <div class="mode
-btns">
heredoc>      rr        <p &&       <ffffffff>    <d       <ffffffff>       <p <
ffffffff><ffffffff>        <div       rr        <p &&       <ffffffff>       <p 
<ffffffff><ffffffff>        <dk=  <div class="receipt-scanner00  <div class="rec
eipigh    <div class="receiptd:    <div       e-    <div       <div v-nf  <div c
lruma    <div <ffffffff><ffffffff> 18     0 0 0 1.71 3h16.94a2 2 0 0 0 1d-      
        </div>
heredoc>         <h2 class="mode-title">选择扫nd          </svg>
heredoc>         </div>
heredoc>  y2        <h2 c          </dive === 'barcode'        <h2 class="ay    
    </div>
heredoc>  y2        <h2 c          </div>
heredoc>         <!- y2        <h->        <h2 class="mode-title">="12        <h
2 c          </diver12        <h2 c     ec        <h2 class="mode-t><       <fff
fffff>       <p <ffffffff><ffffffff>        <div class="mode-btns">
heredoc>      rr        <p &&       <ffffffff>    <d       <ffffffff>       <p <
ffffffff><ffffffff>        <div       rr        <p &&       <ffffffff>       <p 
<ffffffff><ffffffff>        <dk=  <div class="receipt-scanner00  <div class="rec
eipigh    <div class="receiptd:    <div       e-    <div       <div v-nf  <div c
       rr        <p &&       <ffffffff>    <d       <ffffffff>      s=        <h
2 class="mode-title">选择扫nd          </svg>
heredoc>         </div>
heredoc>  y2        <h2 c          </dive === 'barcode'        <h2 class="ay    
    </div>
heredoc>  y2        <h2 c          </div>
heredoc>         <!- y2        <h->        <h2 class="mode-title">="12        <h
2 c          </diver12        <h2 c     ec        <h2 class="mode-t><          <
/div>
heredoc>  y2        <h2 c          </dive === 'barc14 y2        <h   y2        <
h2 c          </div>
heredoc>         <!- y2        <h->        <h2 class="moto        <!- y2        
<h->     ch     rr        <p &&       <ffffffff>    <d       <ffffffff>       <p
 <ffffffff><ffffffff>        <div       rr        <p &&       <ffffffff>       <
p <ffffffff><ffffffff>        <dk=  <div class="receipt-scanner00  <div class="r
ecex="0 0 24 24" fill=        </div>
heredoc>  y2        <h2 c          </dive === 'barcode'        <h2 class="ay    
    </div>
heredoc>  y2        <h2 c          </div>
heredoc>         <!- y2        <h->        <h2 class="mode-title">="12        <h
2 c          </diver12        <h2 c     ec        <h2 class="mode-t><          <
/div>
heredoc>  y2        <h2 c          </dive === 'barc14 y2        <h   y2        <
h2 c        y2        <h   y2        <h2 c          </div>
heredoc>         <!- y2        <h->        <h2 class="mo<ffffffff><ffffffff>    
   <!- y2        <h->     
heredoc>   y2        <h2 c          </dive === 'barc14 y2        <h   y2        
<h2 c          </div>
heredoc>         <!- y2        <h->        <h2 class="moto        <!- y2nd      
  <!- y2        <h->        <h2 class="moto        <!- y2        <h->     ch    
 rr    y2        <h2 c          </dive === 'barcode'        <h2 class="ay       
 </div>
heredoc>  y2        <h2 c          </div>
heredoc>         <!- y2        <h->        <h2 class="mode-title">="12        <h
2 c          </diver12        <h2 c     ec        <h2 class="mode-t><          <
/div>
heredoc>  y2        <h2 c <ffffffff>y2        <h2 c          </div>
heredoc>         <!- y2        <h->        <h2 class="mo          <!- y2        
<h->     ex y2        <h2 c          </dive === 'barc14 y2        <h   y2       
 <h2 c        y2        <h   y2        <h2 c          </div>
heredoc>         <!- y2        <h-<p        <!- y2        <h->        <h2 class=
"mo<ffffffff><ffffffff>       <!- y2        <h->     
heredoc>   y2        <h2 c          </dive === 'barc14 y>
heredoc>   y2        <h2 c          </dive === 'barc14 y2        <h   y2        
<h2 c   on        <!- y2        <h->        <h2 class="moto        <!- y2nd     
   <!- y2        <h-> } y2        <h2 c          </div>
heredoc>         <!- y2        <h->        <h2 class="mode-title">="12        <h
2 c          </diver12        <h2 c     ec        <h2 class="mode-t><          <
/div>
heredoc>  y2        <h2 c <ffffffff>y2        <h2 c          </te        <!- y2 
       <h->     co y2        <h2 c <ffffffff>y2        <h2 c          </div>
heredoc>         <!- y2        <h->        <h2 class="mo          <!- y2        
<h->     ex y2        <h2 c     l)        <!- y2        <h->        <h2 class="m
o    r        <!- y2        <h-<p        <!- y2        <h->        <h2 class="mo
<ffffffff><ffffffff>       <!- y2        <h->     
heredoc>   y2        <h2 c          </dive === 'barc14 y>
heredoc>   y2        <h2 c          </dive === 'barc14 y2     de  y2        <h2 
c          </dive === 'barc14 y>
heredoc>   y2        <h2 c          </dive === 'barc14 y2        <wi  y2        
<h2 c          </dive === 'barc14 yex        <!- y2        <h->        <h2 class
="mode-title">="12        <h2 c          </diver12        <h2 c     ec        <h
2 class="mode-t><          </div>
heredoc>  y2        <h2 c <ffffffff>y2        <h2 c          </t>  y2        <h2
 c <ffffffff>y2        <h2 c          </te        <!- y2        <h->     co y2  
      <h2 c <ffffffff>y2        <h2 c          </div>
heredoc>         <!- y2        c        <!- y2        <h->        <h2 class="mo 
         <!- y2        <h->     ex y2        <h2 c     l)        <!- y2        <
h-> e   y2        <h2 c          </dive === 'barc14 y>
heredoc>   y2        <h2 c          </dive === 'barc14 y2     de  y2        <h2 
c          </dive === 'barc14 y>
heredoc>   y2        <h2 c          </dive === 'barc14 y2        <wi  y2        
<h2 c          </dive === 'barc14 yex  <ffffffff><ffffffff> y2        <h2 c     
     </dive === 'barc14 yal  y2        <h2 c          </dive === 'barc14 y2     
   <wi  y2        <h2 c          </divef (currentIn y2        <h2 c <ffffffff>y2
        <h2 c          </t>  y2        <h2 c <ffffffff>y2        <h2 c          
</te        <!- y2        <h->     co y2        <h2 c <ffffffff>y2        <h2 c 
         </div>
heredoc>         <!- y2        c        <!- y2        <h->        <h2 class="mo 
        va        <!- y2        c        <!- y2        <h->        <h2 class="mo
          <!- y2        <h->     ex y2        <h2 c     l)        <!- y2        
<h-> e   y2        <h2 c        co  y2        <h2 c          </dive === 'barc14 
y2     de  y2        <h2 c          </dive === 'barc14 y>
heredoc>   y2        <h2 c          </dive === 'barc14 y2        <wi  y2        
<h2 c          </dive === 'baeC  y2        <h2 c          </dive === 'barc14 y2 
       <wi  y2        <h2 c          </dive === 'barcst        <!- y2        c  
      <!- y2        <h->        <h2 class="mo         va        <!- y2        c 
       <!- y2        <h->        <h2 class="mo          <!- y2        <h->     e
x y2        <h2 c     l)        <!- y2        <h-> e   y2        <h2 c        co
  y2        <h2 c          </dive === 'barc14 y2     de  y2        <h2 c        
  </dive === 'barc14 y>
heredoc>   y2        <h2 c          </dive === 'barc14 y2        <wi  y2        
<h2 c  st  y2        <h2 c          </dive === 'barc14 y2        <wi  y2        
<h2 c          </dive === 'baeC  y2        <h2 c          </dive === 'barc14 y2 
       <wi  y2        <h2 c          </dive === 'barcst        <!- y2        c  
      <!- y2        <h->        <h2 class="mo         va        <!- y2        c 
       <!- y2        <h->        <h2 class="mo          ((  y2        <h2 c     
     </dive === 'barc14 y2        <wi  y2        <h2 c  st  y2        <h2 c     
     </dive === 'barc14 y2        <wi  y2        <h2 c          </dive === 'baeC
  y2        <h2 c          </dive === 'barc14 y2        <wi  y2        <h2 c    
      </dive === 'barcst        <!- y2        c        <!- y2        <h->       
 <h2 class="mo         va        <!- y2        c        <!- y2        <h->      
  <h2 class="mo          ((  y2        <h2 c          </dive === 'barc14 y2     
   <wi  y2        <h2 c  st  y2        <h2 c          </dive === 'barc1tom right
, #1e3a8a, #581c87, #831843);
heredoc>   overflow: hidden;
heredoc>   position: relative;
heredoc> }
heredoc> 
heredoc> .mode-dialog {
heredoc>   flex: 1;
heredoc>   display: flex;
heredoc>   align-items: center;
heredoc>   justify-content: center;
heredoc>   padding: 1rem;
heredoc> }
heredoc> 
heredoc> .mode-card {
heredoc>   background: rgba(255, 255, 255, 0.95);
heredoc>   backdrop-filter: blur(8px);
heredoc>   border-radius: 24px;
heredoc>   padding: 1.5rem;
heredoc>   max-width: 28rem;
heredoc>   width: 100%;
heredoc>   text-align: center;
heredoc>   box-shadow: 0 25px 50px rgba(0, 0, 0, 0.25);
heredoc> }
heredoc> 
heredoc> .mode-icon {
heredoc>   width: 48px;
heredoc>   height: 48px;
heredoc>   background: linear-gradient(135deg, #3b82f6, #a855f7);
heredoc>   border-radius: 12px;
heredoc>   display: flex;
heredoc>   align-items: center;
heredoc>   justify-content: center;
heredoc>   margin: 0 auto 1rem;
heredoc>   color: white;
heredoc> }
heredoc> 
heredoc> .mode-title {
heredoc>   font-size: 1.25rem;
heredoc>   font-weight: 700;
heredoc>   background: linear-gradient(to right, #2563eb, #9333ea);
heredoc>   -webkit-background-clip: text;
heredoc>   -webkit-text-fill-color: transparent;
heredoc>   margin: 0 0 0.5rem;
heredoc> }
heredoc> 
heredoc> .mode-desc, .mode-hint {
heredoc>   color: #374151;
heredoc>   margin: 0 0 0.25rem;
heredoc>   font-size: 0.875rem;
heredoc> }
heredoc> 
heredoc> .mode-hint {
heredoc>   color: #6b7280;
heredoc>   margin-bottom: 1rem;
heredoc> }
heredoc> 
heredoc> .  overflow: hidden;
heredoc>   position: relol  position: relatiod}
heredoc> 
heredoc> .mode-dialog {
heredoc>   flex;  flex: 1;
heredoc>   ti  displayn;  align-items: 
heredoc> }  justify-content: ceng  padding: 1rem;
heredoc> }
heredoc> 
heredoc> .modera}
heredoc> 
heredoc> .mode-card {ont-s  backgrounre  backdrop-filter: blur(8px);
heredoc>   border-    border-radius: 24px;
heredoc>   pade.  padding: 1.5rem;
heredoc>   d:  max-width: 28reto  width: 100%;
heredoc>   ta8  text-align:r:  box-shadow: 0 25px.o}
heredoc> 
heredoc> .mode-icon {
heredoc>   width: 48px;
heredoc>   height 2px solid   width: 48co  height: 48p
heredoc> }  background: os  border-radius: 12px;
heredoc>   display: flex;
heredoc>   align-items: an  display: flex;
heredoc>   al
heredoc>    align-items: 
heredoc>    justify-content: cemn  margin: 0 auto 1rem;
heredoc>   os  color: white;
heredoc> }
heredoc> 
heredoc> .msc}
heredoc> 
heredoc> .mode-title
heredoc>   fl  font-size:;
heredoc>   font-weight: 700;ack  background: line 0  -webkit-background-clip: te
xt;
heredoc>   -webkit-text-fill-colo    -webkit-text-fill-color: tranen  margin: 0 
0 0.5rem;
heredoc> }
heredoc> 
heredoc> .mode-desc, .
heredoc>  }
heredoc> 
heredoc> .mode-desc, .modesolid  color: #374151;
heredoc>   mar1)  margin: 0 0 0.bo  font-size: 0.875reck }
heredoc> 
heredoc> .mode-hint {
heredoc>   col  bac  color: #6an  margin-bottom:r:}
heredoc> 
heredoc> .  overflow: hidde;
heredoc>   c  position: relol  .h
heredoc> .mode-dialog {
heredoc>   flex;  flex: 1;
heredoc>   t}
heredoc> 
heredoc>   flex;  flex{
heredoc>   ti  displayn;   }  justify-content: ceng  pad;
heredoc> }
heredoc> 
heredoc> .modera}
heredoc> 
heredoc> .mode-card {ont-s  backgro
heredoc>   ju
heredoc> .mode-ont  border-    border-radius: 24px;
heredoc>   pade.  padding: 1.5remht  pade.  padding: 1.5rem;
heredoc>   d:  bo  d:  max-width: 28reto un  ta8  text-align:r:  box-shadow: 0 5
,
heredoc> .mode-icon {
heredoc>   width: 48px;
heredoc>   height 2px 5re  width: 48
heredoc> .  height 2px  {}  background: os  border-radius: 12px;
heredoc>   dihr  display: flex;
heredoc>   align-items: an  dito  align-items: ,   al
heredoc>    align-items: 
heredoc>    justify;
heredoc>    ex   justify-conr;
heredoc>   os  color: white;
heredoc> }
heredoc> 
heredoc> .msc}
heredoc> 
heredoc> .mode-title
heredoc>   ft-}
heredoc> 
heredoc> .msc}
heredoc> 
heredoc> .mode-tiplay:
heredoc> .mox;
heredoc>   fl  fontem  font-weight: 7ti  -webkit-text-fill-colo    -webkit-text-
fill-color: tranen  margin: 0 0 fl}
heredoc> 
heredoc> .mode-desc, .
heredoc>  }
heredoc> 
heredoc> .mode-desc, .modesolid  color: #374151;
heredoc>   mar1)  margin: 0
heredoc>   pa }
heredoc> 
heredoc> .mode-de0 
heredoc> 20p  mar1)  margin: 0 0 0.bo  font-size: }
heredoc> 
heredoc> .mode-hint {
heredoc>   col  bac  color: #6an  margin-bo;
heredoc> }  col  bac nt
heredoc> .  overflow: hidde;
heredoc>   c  position: relolect  c  position: relgn.mode-dialog {
heredoc>   flex; fy  flex;   cente  t}
heredoc> 
heredoc>   flex;  f;
heredoc> 
heredoc>   eig  ti  display.c}
heredoc> 
heredoc> .modera}
heredoc> 
heredoc> .mode-card {ont-s  backgro
heredoc>   ju
heredoc> .width:
heredoc> .mode-vh,  ju
heredoc> .mode-ont  border-  mi.mo0v  pade.  padding: 1.5remht  pade.  paddin c 
 d:  bo  d:  max-width: 28reto un  ta8  text-alig {.mode-icon {
heredoc>   width: 48px;
heredoc>   height 2px 5re  width: 48
heredoc> .  height 2px  ns  width: 48    height 2px le.  height 2px  {}  backgro
rd  dihr  display: flex;
heredoc>   align-items: an  dito  align-e-  align-items: an  did-   align-items:
 
heredoc>    justify;
heredoc>    ex   justif
heredoc>     justify;
heredoc>         ex  e-vis  os  color: white;
heredoc> ac}
heredoc> 
heredoc> .msc}
heredoc> 
heredoc> .mode-tiborde
heredoc> .modiu  ft-}
heredoc> 
heredoc> .m d
heredoc> .mscy: 
heredoc> .mo;
heredoc>  .mox;
heredoc>   fl  :   fler
heredoc> .mode-desc, .
heredoc>  }
heredoc> 
heredoc> .mode-desc, .modesolid  color: #374151;
heredoc>   mar1)  margin: 0
heredoc>   pa }
heredoc> 
heredoc> .mode-de0 
heredoc> 20p  mar-ca }
heredoc> 
heredoc> .mode-deva
heredoc> , .  mar1)  margk canvas {
heredoc>   max-width: 100  pa }
heredoc> 
heredoc> .mode-de010
heredoc> .mod ob20p  mar1 c
heredoc> .mode-hint {
heredoc>   col  bac  color: #6an  ma25rem;
heredoc>   font-we}  col  bac nt
heredoc> .  overflow: hidde;n-.  overfl;
heredoc> }
heredoc> 
heredoc> .  c  position: relne  flex; fy  flex;   cente  t}
heredoc> 
heredoc>   flex;  f;
heredoc> 
heredoc>   eig  ti  d p
heredoc>   flex;  f;
heredoc> 
heredoc>   eig  ti  dis: 1
heredoc>   eig  ti: 8
heredoc> .modera}
heredoc> 
heredoc> .mode-carpx;
heredoc> .mode-itc  ju
heredoc> .width:
heredoc> .mode-vh,  jpx.wi b.mode-ra.mode-ont  b    width: 48px;
heredoc>   height 2px 5re  width: 48
heredoc> .  height 2px  ns  width: 48    height 2px le.  height 2px  {}  backgro
rd  dihr  display: ct  height 2px ro.  height 2px  ns  width: 8a  align-items: a
n  dito  align-e-  align-items: an  did-   align-items: 
heredoc>    justify;
heredoc>    ex   jx;   justify;
heredoc>    ex   justif
heredoc>     justify;
heredoc>         ex  e-vis  os  color: wt:   ex   jugr    justify;
heredoc>  0        ex 
heredoc>  ac}
heredoc> 
heredoc> .msc}
heredoc> 
heredoc> .mode-tiborde
heredoc> .modiu  fdi
heredoc> .: 1
heredoc> .mo1re.modlc(1rem + 
heredoc> .m d
heredoc> .mscyea-.mset.mo;
heredoc>  m) .m    fl ex.mode-desc, .oo }
heredoc> 
heredoc> .mode-de d
heredoc> spl  mar1)  margin: 0
heredoc>   pa }
heredoc> 
heredoc> 
heredoc> 
heredoc> .btn-nav {
heredoc>   pa }
heredoc> 
heredoc> .mode-de0in
heredoc> .mod85r20p  mar-kg
heredoc> .mode-deva
heredoc> 255, .  mar15,  max-width: 100  pa }
heredoc> 
heredoc> .d 
heredoc> .mode-5, 255, 255, 0.2).mod ob20p w.mode-hint {
heredoc>   codi  col  bac  f  font-we}  col  bac nt
heredoc> .  overf {.  overflow: hidde;n-.25}
heredoc> 
heredoc> .  c  position: relne  flexient(
heredoc>   flex;  f;
heredoc> 
heredoc>   eig  ti  d p
heredoc>   flex;  f;
heredoc> 
heredoc>   eig  lor
heredoc>   eig  ti bo  flex;  f;
heredoc> 
heredoc> 12
heredoc>   eig  ti-si  eig  ti: 8
heredoc> .mon-.modera}
heredoc> 
heredoc> .d,
heredoc> .mode-bno.mode-itc  ed.width:
heredoc> .mod: .mode-
heredoc> 
heredoc>   height 2px 5re  width: 48
heredoc> .  height 2px  ns  width: 25.  height 2px  ns  width: :    justify;
heredoc>    ex   jx;   justify;
heredoc>    ex   ju