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
EOF
git add frontend/src/views/ReceiptScanner.vue && git commit -m "fix(scanner): clean overwrite to fix corrupted template tags" && git push origin master`, and this is the output of running that command instead:
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
heredoc>         <div class="mode-icon">
heredoc> k=        <div class="mode-ic
heredoc>             <svg xmlns="http://wNa            <polygon points="13 2 3 1
4 12 14 11 22 21 10 12 10 13 2"/>
heredoc>           </svg>
heredoc>         </div>
heredoc>         <h2 class="mode-title">选择扫nd          </svg>
heredoc>         </div>
heredoc>         <h2 class="mode-title">选择n>        </div>
heredoc> mp        <h2 c          <p class="mode-desc">
heredoc>           <template v-i'/          <template v-if="ow</            正在
为 <span class=<!          </template>
heredoc>           <template v-else>
heredoc>             正在为未指定持有 <div class="mode-card e            正在为 
未<ffffffff>e          </template>
heredoc>         </p>
heredoc>         <p v-if="owne/h        </p>
heredoc> >请先<ffffffff>       <p <ffffffff><ffffffff>        <div class="mode-b
tns">
heredoc>           <t<template>
heredoc>   <div class="receipt-scanner-page"><ffffffff>         <t<template>
heredoc>   <div     <div class="receipt<ffffffff><ffffffff>   <!-- 模式选择 -->
heredoc>     <div       <div v-nf  <div clr-ma    <div <ffffffff><ffffffff>    <
div v-if="includeUn        <div           <div class="modck" @click="$router.pus
h('/')">
heredoc>             k=        <div class="mode-ic
heredoc> 20            <svg xmlns="http0"          </svg>
heredoc>         </div>
heredoc>         <h2 class="mode-title">选择扫nd          </svg>
heredoc>         </di="        </div>
heredoc> 12        <h2 c          </div>
heredoc>         <h2 class="mode-title">选择n>   >
heredoc>         <h2 cutmp        <h2 c          <p class="mode-desc">
heredoc>                  <template v-i'/          <template cl          <templa
te v-else>
heredoc>             正在为未指定持有 <div class="mode-card e            正在为<
ffffffff>d            正在为未<ffffffff>g        </p>
heredoc>         <p v-if="owne/h        </p>
heredoc> >请先<ffffffff>       <p <ffffffff><ffffffff>        <div class="mode-b
tns">
heredoc>      rr        <p &&>请先<ffffffff>       <p <ffffffff><ffffffff>      
  <diver          <t<template>
heredoc>   <div class="receipt-scanner00  <div class="receiptgh  <div     <div c
lass="receipt<ffffffff><ffffffff>   <!-- 模式选择 -->
heredoc> r"     <div       <div v-nf  <div clr-ma    <div <ffffffff><ffffffff>  
  <div18            k=        <div class="mode-ic
heredoc> 20            <svg xmlns="http0"          </svg>
heredoc>         </div>
heredoc>         <h2 class="mode-title">选e 20            <svg xmlns="http0"7"/>

heredoc>              </div>
heredoc>         <h2 class="mode-title">齓        <h2 c<ffffffff><ffffffff>     
   </di="        </div>
heredoc> 12        <h2 c          </d  12        <h2 c          </ec        <h2 
class="mode-title":s        <h2 cutmp        <h2 c          <pee                
 <template v-i'/          <template cl       c            正在为未指定持有 <div 
class="mode-card e            正在<ffffffff>r        <p v-if="owne/h        </p>

heredoc> >请先<ffffffff>       <p <ffffffff><ffffffff>        <div class="mode-b
tns">
heredoc>      rr        <p &&>请先<ffffffff>    <d>请先<ffffffff>       <p <ffff
ffff><ffffffff>        <div       rr        <p &&>请先<ffffffff>       <p <fffff
fff><ffffffff>        <dk=  <div class="receipt-scanner00  <div class="receiptgh
  <div     <div class="d:r"     <div       <div v-nf  <div clr-ma    <div <fffff
fff><ffffffff>    <div18            k=        <div class="mode-ic
heredoc> 20     ">20            <svg xmlns="http0"          </svg>
heredoc>         </div>
heredoc>         <h2 class="mode-title">选e 2<d        </div>
heredoc>         <h2 class="mode-title"><ffffffff>r        <h2 c               <
/div>
heredoc>         <h2 class="mode-title">齓        <h2 c<ffffffff><ffffffff>     
         <div class12        <h2 c          </d  12        <h2 c          </ec  
      <h2 classde>请先<ffffffff>       <p <ffffffff><ffffffff>        <div class
="mode-btns">
heredoc>      rr        <p &&>请先<ffffffff>    <d>请先<ffffffff>       <p <ffff
ffff><ffffffff>        <div       rr        <p &&>请先<ffffffff>       <p <fffff
fff><ffffffff>        <dk=  <div class="receipt-scanner00  <div class="receiptgh
  <div     <div class="d:r"     <div       <div v-nf  <div clr-ma    <div <fffff
fff><ffffffff>          rr        <p &&>请先<ffffffff>    <d>请先<ffffffff>     
 s=20     ">20            <svg xmlns="http0"          </svg>
heredoc>         </div>
heredoc>         <h2 class="mode-title">选e 2<d        </div>
heredoc>         <h2 class="mode-title"><ffffffff>r        <h2 c               <
/div>
heredoc>         <h2 class="mode-title">齓        <h2 c<ffffffff><ffffffff>     
         <div class12        <h2 c          </d  12     r"        </div>
heredoc>         <h2 class="mode-title">选e 2<d  v1        <h2 cv1        <h2 cl
ass="mode-title"><ffffffff>r        <h2 c               <h2 class="mode-title"> 
齓        <h2 c<ffffffff><ffffffff>              <div       rr        <p &&>请先
<ffffffff>    <d>请先<ffffffff>       <p <ffffffff><ffffffff>        <div       
rr        <p &&>请先<ffffffff>       <p <ffffffff><ffffffff>        <dk=  <div c
lass="receipt-scanner00  <div class="receiptgh  <div     <div class=""         <
/div>
heredoc>         <h2 class="mode-title">选e 2<d        </div>
heredoc>         <h2 class="mode-title"><ffffffff>r        <h2 c               <
/div>
heredoc>         <h2 class="mode-title">齓        <h2 c<ffffffff><ffffffff>     
         <div class12        <h2 c          </d  12     r"        </div>
heredoc>         <h2 class="mode-title">选e 2<d  v1        <h2 cv1        <h2 cl
ass="mode-title"><ffffffff>r        <h2 c           <h2 c          <h2 class="mo
de-title"><ffffffff>r        <h2 c     as        <h2 class="mode-title">齓      
  <h2 c<ffffffff><ffffffff>              <div          <h2 class="mode-title">选
e 2<d  v1        <h2 cv1        <h2 class="mode-title"><ffffffff>r        <h2 c 
              <h2 c          <h2 class="mode-title">选e 2<d        </div>
heredoc>         <h2 class="mode-title"><ffffffff>r        <h2 c               <
/div>
heredoc>         <h2 class="mode-title">齓        <h2 c<ffffffff><ffffffff>     
         <div class12        <h2 c          </d  12     r"        </div>
heredoc>         <h2 class="mode-title">选e 2<d  v1        <h2 cv1        <h2 cl
ass="mode-title"><ffffffff>r        <h2 c           <h2 c          <h2 class="mo
de-ti '        <h2 class="mode-title"><ffffffff>r        <h2 c     
heredoc>          <h2 class="mode-title">齓        <h2 c<ffffffff><ffffffff>    
          <div==        <h2 class="mode-title">选e 2<d  v1        <h2 cv1       
 <h2 class="mode-title"><ffffffff>r        <h2 c           <h2 c              <h
2 class="mode-title"><ffffffff>r        <h2 c               </div>
heredoc>         <h2 class="mode-title">齓        <h2 c<ffffffff><ffffffff>     
         <div class12        <h2 c          </d  12     r"        </div>
heredoc>         <h2 class="mode-title">选e 2<d  v1        <h2 cv1        <h2 cl
ass="mode-title"><ffffffff>r        <h2 c           <h2 c          <h2 class="mo
de-ti '        <h2 class="mode-title"><ffffffff>r        <h2 c     
heredoc>          <h2 class="mode-mp        <h2 class="mode-title">齓        <h2
 c<ffffffff><ffffffff>              <div {        <h2 class="mode-title">选e 2<d
  v1        <h2 cv1        <h2 class="mode-title"><ffffffff>r        <h2= comput
ed(() => route.p         <h2 class="mode-title">齓        <h2 c<ffffffff><ffffff
ff>              <div==        <h2 class="mode-title">选e 2<d  v1        <h2 cv1
        <h2 class="mode-title"><ffffffff>r        <h2 c           <h2 c         
   b        <h2 class="mode-title">齓        <h2 c<ffffffff><ffffffff>          
    <div class12        <h2 c          </d  12     r"        </div>
heredoc>         <h2 class="mode-title">选e 2<d  v1        <h2 cv1        <h2 cl
ass="mode-title"><ffffffff>r        <h2 c           <h2 c          <h2 cle      
   <h2 class="mode-title">选e 2<d  v1        <h2 cv1        <h2 class="mode-titl
e"><ffffffff>r        <h2 c           <h2 c    =>         <h2 class="mode-mp    
    <h2 class="mode-title">齓        <h2 c<ffffffff><ffffffff>              <div
 {        <h2 class="mode-title">选e 2<d  v1        <h2 cv1        <h2 class="mo
de-title"><ffffffff>r        <h2= c<ffffffff><ffffffff>       <h2 class="mode-ti
tle">选e 2<d  v1        <h2 cv1        <h2 class="mode-title"><ffffffff>r       
 <h2 c           <h2 c          <h2 cle         <h2 class="mode-title">选e 2<d  
v1        <h2 cv1        <h2 class="mode-title"><ffffffff>r        <h2 c        
   <h2 c    =>         <h2 class="mode-mp        <h2 class="mode-title">齓      
  <h2 c<ffffffff><ffffffff>              <div {        <h2 class="mode-title">选
e 2<d  v1        <h2 cv1        <h2 class="mode-title"><ffffffff>r        <h2= c
<ffffffff><ffffffff>       <h2 class="mode-title">选e 2<d  v1        <h2 cv1    
    <h2 class="mode-ue = receipts.value.map((r) =>
heredoc>     r.id === currentReceipt.value.id ? { ...r, isAbnormal: true } : r
heredoc>   );
heredoc>   if (currentIndex.value < receipts.value.length - 1) {
heredoc>     currentIndex.value++;
heredoc>   }
heredoc> };
heredoc> 
heredoc> const generateCodes = async () => {
heredoc>   if (receipts.value.length === 0) return;
heredoc>   await nextTick();
heredoc>   receipts.value.forEach(async (receipt, index) => {
heredoc>     try {
heredoc>       const qrEl = qrRefs.value[index];
heredoc>       const barcodeEl = barcodeRefs.value[index];
heredoc>       if (qrEl) await QRCode.toCanvas(qrEl, receipt.ticketNumber, { wid
th: 400, margin: 2 });
heredoc>       if (barcodeEl) JsBarcode(barcodeEl, receipt.ticketNumber, { forma
t: 'CODE128', width: 3, height: 150, displayValue: true, fontSize: 20, margin: 1
0 });
heredoc>     } catch (err) { console.error(err); }
heredoc>   });
heredoc> };
heredoc> 
heredoc> watch(receipts, generateCodes, { deep: true });
heredoc> watch(includeUnassigned, (val) => { if (val !== null) generateCodes(); 
});
heredoc> 
heredoc> const handleKeyDown = (e) => {
heredoc>   if (includeUnassigned.value === null || receipts.value.length === 0) 
return;
heredoc>   if (e.key === 'ArrowLeft' && currentInd    r.id === currentReceiptva 
 );
heredoc>   if (currentIndex.value < receipts.value.length - 1) {
heredoc>     curei  i.v    currentIndex.value++;
heredoc>   }
heredoc> };
heredoc> 
heredoc> const generateCodes y   }
heredoc> };
heredoc> 
heredoc> const generateCoac};ar
heredoc> ) {  if (receipts.value.length === 0)e(  await nextTick();
heredoc>   receipts.value.forE=   receipts.value.f);    try {
heredoc>       const qrEl = qrRefs.value[index];
heredoc>  si      coue      const barcodeEl = barcodeRefs.vaDa      if (qrEl) aw
ait QRCode.toCanvas(qrEl, recerI      if (barcodeEl) JsBarcode(barcodeEl, receip
t.ticketNumber, { format: 'CODE128', width:  f    } catch (err) { console.error(
err); }
heredoc>   });
heredoc> };
heredoc> 
heredoc> watch(receipts, generateCodes, { deep: true });
heredoc> watch(includeUnassigned, (val) => { if (vowner);
heredoc>       fi  });
heredoc> };
heredoc> 
heredoc> watch(receipts, generateCodes,  };
heredoc> 
heredoc>  }
heredoc>   rwatch(includeUnassigned, (val) => { if (val !=in
heredoc> const handleKeyDown = (e) => {
heredoc>   if (includeUnassigned.value === null ||  c  if (includeUnassigned.val
ueer  if (e.key === 'ArrowLeft' && currentInd    r.id === currentReceiptva  );
heredoc>   );  if (currentIndex.value < receipts.value.length - 1) {
heredoc>     curei  i.v   wn    curei  i.v    currentIndex.value++;
heredoc>   }
heredoc> };
heredoc> 
heredoc> const  {  }
heredoc> };
heredoc> 
heredoc> const generateCodes y   }
heredoc> };
heredoc> 
heredoc> ll};va
heredoc> lab};
heredoc> 
heredoc> const generateCoac};le
heredoc> -di) {  if (receipts.valck  receipts.value.forE=   receipts.value.f);  
  try {
heredoc>     ,       const qrEl = qrRefs.value[index];
heredoc>  si      co
heredoc> } si      coue      const barcodeEl = b f  });
heredoc> };
heredoc> 
heredoc> watch(receipts, generateCodes, { deep: true });
heredoc> watch(includeUnassigned, (val) => { if (vowner);
heredoc>       fi  });
heredoc> };
heredoc> 
heredoc> watch(receipts, generateCodes,  };
heredoc> 
heredoc>  }
heredoc>   rwatch(includeUnassigned, (val) => { if (val !=in
heredoc> const handleKeyDown = t};
heredoc> 
heredoc> al
heredoc> gn:watch(includeUnassigned, (val) => { if (vowner0,      fi  });
heredoc> };
heredoc> 
heredoc> watch(receipts, generateCodes: };
heredoc> 
heredoc> watch(reground
heredoc>  }
heredoc>   rwatch(includeUnassigned, (, # 85const handleKeyDown = (e) => {
heredoc>   if (includeUnassiig  if (includeUnassigned.valuent  );  if (currentIn
dex.value < receipts.value.length - 1) {
heredoc>     curei  i.v   wn    curei  i.v    currentIndex.value++;
heredoc>   }
heredoc> };
heredoc> 
heredoc> const  {  }
heredoc> };
heredoc> 
heredoc> constt,    curei  i.v   wn    curei  i.v    currentIndex.value++;bk  }
heredoc> };
heredoc> 
heredoc> const  {  }
heredoc> };
heredoc> 
heredoc> const generateCodes y   }
heredoc> };
heredoc> 
heredoc> ll}
heredoc> .};de
heredoc> des};
heredoc> 
heredoc> const in
heredoc>  {
heredoc> };
heredoc> 
heredoc> ll};va
heredoc> lab};
heredoc> 
heredoc> const in
heredoc>  0 lab};5r
heredoc> con  f-di) {  if (receipts.
heredoc> 
heredoc>     ,       const qrEl = qrRefs.value[index];
heredoc>  si      co
heredoc> } si      coue     we si      co
heredoc> } si      coue      const barcod
heredoc>  } si      fl};
heredoc> 
heredoc> watch(receipts, generateCodes, { deep: tr}
heredoc> 
heredoc> .btwatch(includeUnassigned, (val) => { if (vowneriu      fi  });
heredoc> };
heredoc> 
heredoc> watch(receipts, generateCodes 5};
heredoc> 
heredoc> watch(re: 
heredoc> one
heredoc>  }
heredoc>   rwatch(iinter;
heredoc> }
heredoc> 
heredoc> .btn-mode.pr maconst handleKeyDown = t};
heredoc> 
heredoc> al
heredoc> gn:watch(includeUb82f6
heredoc> al
heredoc> gn:watch(includeUnaste;g}
heredoc> };
heredoc> 
heredoc> watch(receipts, generateCodes: };
heredoc> 
heredoc> watch(reground
heredoc>  }
heredoc>   rwate5
heredoc> 7eb
heredoc> watch(reground
heredoc>  }
heredoc>   rwatch(incl.ghost {
heredoc>   backgro nd  if (includeUnassiig  if (includeUnassigned.valuent  );  i
f (cu d    curei  i.v   wn    curei  i.v    currentIndex.value++;
heredoc>   }
heredoc> };
heredoc> 
heredoc> const  {  }
heredoc> };
heredoc> 
heredoc> constt,    curei  i.v   wn sh  }
heredoc> };
heredoc> 
heredoc> const  {  }
heredoc> };
heredoc> 
heredoc> constt,    curei  i.v   wn    cur;
heredoc> };ba
heredoc> kdr};
heredoc> 
heredoc> constt b
heredoc> ur(};
heredoc> 
heredoc> const  {  }
heredoc> };
heredoc> 
heredoc> const generateCodes y   }
heredoc> };
heredoc> 
heredoc> ll}
heredoc> .};de
heredoc> des};
heredoc> 
heredoc> cost
heredoc> fy-};
heredoc> 
heredoc> const pa
heredoc> e-b};
heredoc> 
heredoc> ll}
heredoc> .};de
heredoc> des};
heredoc> 
heredoc> con 1
heredoc> x s.}iddes}a(
heredoc> con 25 {
heredoc> };
heredoc> 
heredoc>  0.1)
heredoc> 
heredoc>   lab};iz
heredoc> con bo 0 lab}x;con  f-diba
heredoc>     ,       const qrEl = background: transparent;
heredoc>   border: none;
heredoc>   color:} si      cu} si      coue      const barcor  } si      fl};
heredoc> 
heredoc> watch(receiptser
heredoc> watch(receiptor: white;
heredoc>   font-weight: 600;
heredoc>   margin: 0;
heredoc>   d};
heredoc> 
heredoc> watch(receipts, generateCodes 5};
heredoc> 
heredoc> watch(re: 
heredoc> one
heredoc>  }
heredoc>   rwat
heredoc>  
heredoc> gap
heredoc> watch(re: 
heredoc> one
heredoc>  }
heredoc>   rwatch(iint;
heredoc>   height: 8p }
heredoc>   ba}
heredoc> 
heredoc> .btn-mode.pr80;
heredoc>  
heredoc> al
heredoc> gn:watch(includeUb82f6
heredoc> al
heredoc> gn:watch(  coloal
heredoc> gn:watch(includeUn, g.7};
heredoc> 
heredoc> watch(receipts, gene
heredoc>  
heredoc> mar
heredoc> watch(reground
heredoc>  }
heredoc>   rwate5
heredoc> 7eb
heredoc> pac }
heredoc>   rwate5
heredoc> 7 4 px7eb
heredoc> watwnwa-w }
heredoc>   rwatch(iex sh  backgro nd  if (ind:  }
heredoc> };
heredoc> 
heredoc> const  {  }
heredoc> };
heredoc> 
heredoc> constt,    curei  i.v   wn sh  }
heredoc> };
heredoc> 
heredoc> const  {  }
heredoc> };
heredoc> 
heredoc> constt,    curei  i.v   wn    cur;
heredoc> };ba
heredoc> kdr};
heredoc> 
heredoc> constt b
heredoc> uront-weight};
heredoc> 
heredoc> consttis
heredoc> lay};
heredoc> 
heredoc> const  {  }
heredoc> };
heredoc> 
heredoc> constt,      
heredoc> ust};
heredoc> 
heredoc> consttt:
heredoc> center;
heredoc>   gap: 0.5rem;
heredoc> }
heredoc> 
heredoc> .code-area {kdrfl
heredoc> con1;
heredoc> ur(};
heredoc> 
heredoc> ay
heredoc> conex;};
heredoc> 
heredoc> const re
heredoc> tio};
heredoc> 
heredoc> ll}
heredoc> .};de
heredoc> des};
heredoc> 
heredoc> cos: 
heredoc> ent.};
heredoc> des}st
heredoc> cosconfy-t:
heredoc> contere-b};
heredoc> 
heredoc> di
heredoc> ll}1re.}0 120px;
heredoc> converx s.: con 25 {
heredoc> };
heredoc> dt};
heredoc> 
heredoc>  0.;
heredoc> 
heredoc> 
heredoc> 
heredoc> .
heredoc>   lnercon bo 0{
heredoc>     ,       const qrEl = 10  border: none;
heredoc>   color:} si      cu} si      co
heredoc>    color:} si  n:
heredoc> watch(receiptser
heredoc> watch(receiptor: white;
heredoc>   font-weight: 600;
heredoc>   mar 10watch(receiptor00%;
heredoc> }
heredoc> 
heredoc> .code-card-contain  margin: 0;
heredoc>   d};:   d};
heredoc> 
heredoc> watcid
heredoc> watmin
heredoc> watch(re: 
heredoc> one
heredoc>  }
heredoc>   rwat
heredoc>  
heredoc> gap
heredoc> in(one
heredoc>  }
heredoc>   w, }00 x) 
heredoc> gapargiwaboone
heredoc>  }
heredoc>   m; } c rs  height: 8p     ba}
heredoc> 
heredoc> . 10;
heredoc> }
heredoc> 
heredoc> .btde- 
heredoc> al
heredoc> gn:watch( positial
heredoc> gn:watch(  coloal
heredoc> : g00gn:watch(include%;
heredoc> watch(receipts, gene
heredoc>  
heredoc>  0. 
heredoc> mar
heredoc> watch(regrounle: waes }
heredoc>   rwate5
heredoc> 7co e-7eb
heredoc> pacnepafl  rwd 7 4 px7nswatwnwa-ta  rwatch(i);};
heredoc> 
heredoc> const  {  }
heredoc> };
heredoc> 
heredoc> constt,    curei  i
heredoc>  
heredoc> pos};
heredoc> 
heredoc> consttol
heredoc> te;};
heredoc> 
heredoc> const  {  }
heredoc> };
heredoc> 
heredoc> constt,    
heredoc>  
heredoc> bac};
heredoc> 
heredoc> consttbi
heredoc> ity};ba
heredoc> kdr};
heredoc> 
heredoc> constt b
heredoc> uront-weightbokdrr-
heredoc> conus:uront-w  
heredoc> consttis
heredoc> lay;
heredoc>  lay};
heredoc> 
heredoc> it
heredoc> con ce};
heredoc> 
heredoc> consttst
heredoc> fy-ust};
heredoc> 
heredoc> constte
heredoc> con pacenter;1r  gap:
heredoc> .}
heredoc> 
heredoc> .code-areak {
heredoc>  con1;
heredoc> ur(};
heredoc> 
heredoc> ay
heredoc> teur180d
heredoc> ay
heredoc> 
heredoc> }
heredoc> c.c
heredoc> const d-ftio};
heredoc> 
heredoc> nv
heredoc> ll}.co.}-cdes}ba
heredoc> cosanvas {
heredoc>   max-width: 100%;
heredoc>   max-he
heredoc> di
heredoc> ll100%;
heredoc>  lobconverx s.: conin};
heredoc> dt};
heredoc> 
heredoc>  0.;
heredoc> 
heredoc> 
heredoc> 
heredoc> .
heredoc> 
heredoc>  dfo
heredoc>  0siz
heredoc> 
heredoc> 
heredoc> 1.25rem    ,       cot:  color:} si      cu} si      co
heredoc>    color
heredoc> }   color:} si  n:
heredoc> watch(receiptspwatch(receiptserkgwatch(receiptor0,  font-weight: 600;
heredoc>   px  mar 10watch(rece 1}
heredoc> 
heredoc> .code-card-contain  martop:   d};:   d};
heredoc> 
heredoc> watcid
heredoc> watmin
heredoc> win
heredoc> watcid
heredoc> wat;
heredoc>  watmierwatchusone
heredoc>  }
heredoc>    b }der:  
heredoc> gap
heredoc>   binkg }und:  ransparent;
heredoc>   color: rgba(255, 25
heredoc> . 10;
heredoc> }
heredoc> 
heredoc> .btde- 
heredoc> al
heredoc> gn:watch(5rem;
heredoc>   cursal
heredoc> gn:ingergn:watch(  coloalac: g00gn:watch(inunwatch(receipts, gene
heredoc>  3a 
heredoc>  0. 
heredoc> mar
heredoc> watch(re {
heredoc>  marnt-siz  rwate5
heredoc> 7co e-7eb
heredoc> pacgb7co e-725pacnepaf0.
heredoc> const  {  }
heredoc> };
heredoc> 
heredoc> constt,    curei  i
heredoc>  
heredoc> pos};
heredoc> 
heredoc> cpos};
heredoc> 
heredoc> constted
heredoc> 
heredoc>    
heredoc> pos};
heredoc> 
heredoc> consttol
heredoc>  0;
heredoc> 
heredoc> conght: 0;
heredoc>   ba
heredoc> conoun};
heredoc> 
heredoc> const, 0
heredoc>  0, 
heredoc> bac};
heredoc> 
heredoc> cback
heredoc> con-fiity};balukdr};
heredoc> );
heredoc> conadduront-wemconus:uront-w  
heredoc> coenconsttis
heredoc> lay;
heredoc> selay;
heredoc>  lm) la  
heredoc> it
heredoc> cex:c10
heredoc> constt.fofy-ust}ns
heredoc> consttsplcon paex.}
heredoc> 
heredoc> .code-areak {
heredoc>  c
heredoc> .
heredoc> tn- con1;
heredoc> ur(};: ur(};pa
heredoc> ay
heredoc> g: t.8ay
heredoc> 
heredoc> }
heredoc> c b
heredoc> ckgrocod:
heredoc> nv
heredoc> ll}.5, 255, l55cosanvas {
heredoc>   mar:  max-widd   max-he
heredoc> di
heredoc> ll1005,di
heredoc> ll10  lol lobcoitdt};
heredoc> 
heredoc>  0.;
heredoc> 
heredoc> 
heredoc> 
heredoc> .
heredoc> 
heredoc>  dfo
heredoc> px
heredoc>  0 fo
heredoc> 
heredoc> 
heredoc> size: 1r 0s
heredoc> }
heredoc> 
heredoc> 1.tn-a   color
heredoc> }   color:} si  n:
heredoc> watch(receiptspwatch(receine}   coliewatch(receiptspwa44  px  mar 10wat
ch(rece 1}
heredoc> 
heredoc> .code-card-contain  martop:   d};:   d};
heredoc>   
heredoc> .code-card-contain  mar-na
heredoc> watcid
heredoc> watmin
heredoc> win
heredoc> watcid
heredoc> wat;
heredoc>  watmieropawatmi 0win
heredoc> w
heredoc> 
heredoc> wahowat;
heredoc> s  wa f }
heredoc>    b }der:  
heredoc> g;
heredoc>   cgap
heredoc>   binkg25   2  color: rgba(255, 25
heredoc> . 1op. 10;
heredoc> }
heredoc> 
heredoc> .btde- 
heredoc> al
heredoc> gn: center;al
heredoc> gn:tyge>  cursal
heredoc> gn:ifrgn:ing/sr 3a 
heredoc>  0. 
heredoc> mar
heredoc> watch(re {
heredoc>  marnt-siz  rwate5
heredoc> 7co e-7eb
heredoc> pacgb7co e o 0.wrmar twafi marnt-sied7co e-7eb
heredoc> pacgb7c& pacgb7co oconst  {  }
heredoc> }