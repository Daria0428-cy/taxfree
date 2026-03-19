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
git add frontend/src/views/ReceiptScanner.vue && git commit -m "fix(scanner): final clean overwrite to fix corrupted template tags" && git push origin master`, and this is the output of running that command instead:
ashin@ashindeMacBook-Pro taxfree %  rm /Users/ashin/development/taxfree/frontend
/src/views/ReceiptScanner.vue && cat << 'EOF' > /Users/ashin/development/taxfree
/frontend/src/views/ReceiptScanner.vue
cmdand heredoc> <template>
cmdand heredoc>   <div class="receipt-scanner-page">
cmdand heredoc>     <!-- 模式选择 -->
cmdand heredoc>     <div v-if="includeUnassigned === null" class="mode-dialog">
cmdand heredoc>       <div class="mode-card">
cmdand heredoc>         <div class="mode-icon">
cmdand heredoc>           <svg xmlns="http://www.w3.org/2000/svg" width="32" hei
ght="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">

cmdand heredoc>             <polygon points="13 2 3 14 12 14 11 22 21 10 12 10 1
3 2"/>
cmdand heredoc>           </svg>
cmdand heredoc>         </div>
cmdand heredoc>         <h2 class="mode-title">选择扫描模式</h2>
cmdand heredoc>         <p class="mode-desc">
cmdand heredoc>           <template v-if="ownerId">
cmdand heredoc>             正在为 <span class="highlight">{{ ownerName }}</span
> 准备扫描队列
cmdand heredoc>           </template>
cmdand heredoc>           <template v-else>
cmdand heredoc>             正在为未指定持有人准备扫描队列
cmdand heredoc>           </template>
cmdand heredoc>         </p>
cmdand heredoc>         <p v-if="ownerId" class="mode-hint">是否包含"未<template
>
cmdand heredoc>   <div class="receipt-scanner-page">
cmdand heredoc>     <!-- 模式选择 -->
cmdand heredoc>     <div v-if="includeUnassigned === null" class="mode-dialog">
cmdand heredoc>       <div class="mode-cnf  <div clru    <!-- 模式选择 -->
cmdand heredoc>     <div <ffffffff><ffffffff>    <div v-if="includeUn        <di
v class="mode-card"ton class="btn-mode outline" @click=        <div class="mode-
ico            <svg xmlns="http://wNa            <polygon points="13 2 3 14 12 1
4 11 22 21 10 12 10 13 2"/>
cmdand heredoc>           </svg>
cmdand heredoc>         </div>
cmdand heredoc>         <h2 class="mode-title">选择<ffffffff>nd          </svg>
cmdand heredoc>         </div>
cmdand heredoc>         <h2 clas<ffffffff>
cmdand heredoc>             </button>        </div>
cmdand heredoc> mp        <h2 c          <p class="mode-desc">
cmdand heredoc>           <template v-i'/          <template v-if="ow</         
   正在为 <span class=<!          </template>
cmdand heredoc>           <template v-else>
cmdand heredoc>             正在为未<ffffffff>g">
cmdand heredoc>       <          <template  e            正在为未<ffffffff>e    
      </template>
cmdand heredoc>         </p>
cmdand heredoc>         <p v-if="owne/h        </p>
cmdand heredoc>        <ffffffff>       <p <ffffffff><ffffffff>  <div class="rec
eipt-scanner-page">
cmdand heredoc>     <!-- 模式选择 -->
cmdand heredoc>     <d$r    <!-- 模式选择 -->
cmdand heredoc>     <div on    <div v-if="    </div>
cmdand heredoc>       <div class="mode-cnf  <div clru    <!-- 模式选择 -->as    
<div <ffffffff><ffffffff>    <div v-if="includeUn        <div class="mode-c     
       </svg>
cmdand heredoc>         </div>
cmdand heredoc>         <h2 class="mode-title">选择<ffffffff>nd          </svg>
cmdand heredoc>         </div>
cmdand heredoc>         <h2 clas<ffffffff>
cmdand heredoc>             </button>        </div>
cmdand heredoc> mp        <h2 c          <p class="mode-desc">
cmdand heredoc>           <template v-i'/="        </ y2="12        <h2 c       
   </div>
cmdand heredoc>         <h2 clas<ffffffff>
cmdand heredoc>             </button>   >
cmdand heredoc>         <h2 cut            </budimp        <h2 c          <p cla
ss=      class="owner-badge">
cmdand heredoc>               <span cl          <template v-else>
cmdand heredoc>             正在为未<ffffffff>g">
cmdand heredoc>       <          <template  e            正在为<ffffffff>d      
      正在为未<ffffffff>g      <          <template  e          </p>
cmdand heredoc>         <p v-if="owne/h        </p>
cmdand heredoc>        <ffffffff>       <p <ffffffff><ffffffff>  <divrr        <
p &&       <ffffffff>       <p <ffffffff><ffffffff>  <div classer    <!-- 模式选
择 -->
cmdand heredoc>     <d$r    <!-- 模式选择 -vg    <d$r  6" height="16" v    <div 
on    <div v-if="    </tr      <div class="mode-cnf  <div cl>
cmdand heredoc>         </div>
cmdand heredoc>         <h2 class="mode-title">选择<ffffffff>nd          </svg>
cmdand heredoc>         </div>
cmdand heredoc>         <h2 clas<ffffffff>
cmdand heredoc>             </button>        </div>"9        <h2 c"1        </di
v>
cmdand heredoc>         <h2 clas<ffffffff>
cmdand heredoc>             </button>   />        <h2 csv            </bu<ffffff
ff>p        <h2 c          <p class<ffffffff><ffffffff>         <template v-i'/=
"        </ y2="12 ea        <h2 clas<ffffffff>
cmdand heredoc>             </button>   >
cmdand heredoc>         <h2 cut            <:s            </bu1"        <h2 cut 
        ee              <span cl          <template v-else>
cmdand heredoc>             正在为未<ffffffff>g">
cmdand heredoc>       <                  正在为未<ffffffff>g">
cmdand heredoc>       <          er      <          <template  e          <p v-i
f="owne/h        </p>
cmdand heredoc>        <ffffffff>       <p <ffffffff><ffffffff>  <divrr        <
p &&       <ffffffff>       <p <ffffffff><ffffffff>  <div classer    <!-- er    
   <ffffffff>       <p <ffffffff><ffffffff>  <divrr    e-    <d$r    <!-- 模式选
择 -vg    <d$r  6" height="16" v    <div on    <div v-if="    </tr      <divco  
      <= 'qr' }]">
cmdand heredoc>                     <!-- 条形码 -->
cmdand heredoc>                     <div class="code-card-front">
cmdand heredoc>                             <h2 cf=        </div>
cmdand heredoc>         <h2 clas<ffffffff>
cmdand heredoc>             </button>             <h2 cif            </buba     
   <h2 clas<ffffffff>
cmdand heredoc>             </button>   />        <h2 csv                       
 <!            </button>   >
cmdand heredoc>         <h2 cut            <:s            </bu1"        <h2 cut 
        ee              <span cl          <template v-else>
cmdand heredoc>                   <h2 cut          '            正在为未<fffffff
f>g">
cmdand heredoc>       <                  正在为未<ffffffff>g">
cmdand heredoc>       <          er      <          <template          <        
          正<ffffffff>i      <          er      <          <templ>
cmdand heredoc>        <ffffffff>       <p <ffffffff><ffffffff>  <divrr        <
p &&       <ffffffff>       <p <ffffffff><ffffffff>  <div classer    <--        
            <!-- 条形码 -->
cmdand heredoc>                     <div class="code-card-front">
cmdand heredoc>                             <h2 cf=        </div>
cmdand heredoc>         <h2 clas<ffffffff>
cmdand heredoc>             </button>             <h2 cif            </buba     
   <h2 clas<ffffffff>
cmdand heredoc>             </busv                    <div class="code-0        
                     <rrentColor" stroke-wi        <h2 clas<ffffffff>
cmdand heredoc>             </button>         4M            <4M15  v14M17 5v14"/
>
cmdand heredoc>               </svg>
cmdand heredoc>               条形码
cmdand heredoc>             </bu        <h2 cut            <:s            </bu1"
        <h2 cut         ee              <sp }                  <h2 cut          
'            正在为未<ffffffff>g">
cmdand heredoc>       <                  正在为未<ffffffff>g">
cmdand heredoc>       <    ei      <                  正在为未<ffffffff>g">
cmdand heredoc>       <          er  ro      <          er      <          <temp
ly=       <ffffffff>       <p <ffffffff><ffffffff>  <divrr        <p &&       <f
fffffff>       <p <ffffffff><ffffffff>  <div classer    <--                    <
!-- 条形码 -x=                    <div class="code-card-front">
cmdand heredoc>                             <h2 cf=        </div>
cmdand heredoc>         <h2 clas<ffffffff>
cmdand heredoc>      h3                            <h2 cf=        </二<ffffffff>
       <h2 clas<ffffffff>
cmdand heredoc>             </button>                       </bubi            </
busv                    <div class="code-0                   ca            </but
ton>         4M            <4M15  v14M17 5v14"/>
cmdand heredoc>               </svg>
cmdand heredoc>               条形码
cmdand heredoc>             ="              </svg>
cmdand heredoc>               条形码
cmdand heredoc>             </bu                   条<ffffffff><ffffffff>       
    </bu              <                  正在为未<ffffffff>g">
cmdand heredoc>       <    ei      <                  正在为未<ffffffff>g">
cmdand heredoc>       <          er  ro      <          er      <          <temp
ly=              <    ei      <                  <ffffffff>         <          e
r  ro      <          er      <       '                            <h2 cf=      
  </div>
cmdand heredoc>         <h2 clas<ffffffff>
cmdand heredoc>      h3                            <h2 cf=        </二<ffffffff>
       <h2 clas<ffffffff>
cmdand heredoc>             </button>                       </bubi            </
busv                    <div cla/b        <h2 clas<ffffffff>
cmdand heredoc>      h3                       tc     h3         <ffffffff><fffff
fff>           </button>                       </bubi            </busv         
            </svg>
cmdand heredoc>               条形码
cmdand heredoc>             ="              </svg>
cmdand heredoc>               条形码
cmdand heredoc>             </bu                   条<ffffffff><ffffffff>       
    </bu              <              sa              条<ffffffff>l            ="
        Sw              条形码
cmdand heredoc>          po            </bu      t       <    ei      <         
         正在为未<ffffffff>g">
cmdand heredoc>       <          er  ro      <          er      <  ec      <    
      er  ro      <          er      <      Id        <h2 clas<ffffffff>
cmdand heredoc>      h3                            <h2 cf=        </二<ffffffff>
       <h2 clas<ffffffff>
cmdand heredoc>             </button>                       </bubi            </
busv                    <div cla/b        <h2 clas<ffffffff>
cmdand heredoc>      h3        ns    rcodeRefs = re            </button>        
               </bubi            </busv      {     h3                       tc  
   h3         <ffffffff><ffffffff>           </button>                       </b
ubi            el;               条形码
cmdand heredoc>             ="              </svg>
cmdand heredoc>               条形码
cmdand heredoc>             </bu                   条<ffffffff><ffffffff>       
    </bu   nd            ="        nd              条形码
cmdand heredoc>          ns            </bu      ns         po            </bu  
    t       <    ei      <                  正在为未<ffffffff>g">
cmdand heredoc>       <          er  ro      <          er      <  ec      <    
 '      <          er  ro      <          er      <  ec      <          er  ro  
    <      rn      h3                            <h2 cf=        </二<ffffffff>  
     <h2 clas<ffffffff>
cmdand heredoc>             </button>                       </bubi =            
 </button>                       </bubi            </busv      =     h3        n
s    rcodeRefs = re            </button>                       </bubi           
 </busv      {    en            ="              </svg>
cmdand heredoc>               条形码
cmdand heredoc>             </bu                   条<ffffffff><ffffffff>       
    </bu   nd            ="        nd              条形码
cmdand heredoc>          ns            </bu      ns         po            </bu  
    t       < );              条形码
cmdand heredoc>          pt            </bu                 ns            </bu  
    ns         po            </bu      t       <    ei      <                  <
ffffffff>
cmdand heredoc>       <          er  ro      <          er      <  ec      <    
 '      <          er  ro      <          er      <  ec                    </but
ton>                       </bubi =             </button>                       
</bubi            </busv      =     h3        ns    rcodeRefs = re            </
button>                       </bubi            </busv 3              条形码
cmdand heredoc>             </bu                   条<ffffffff><ffffffff>       
    </bu   nd            ="        nd              条形码
cmdand heredoc>          ns            </bu      ns         po            </bu  
    t       < );              条形码
cmdand heredoc>          pt            </bu           in            </bu      ==
         ns            </bu      ns         po            </bu      t       < );
              条形码
cmdand heredoc>      In         pt            </bu                 ns           
 </bu      ns         po            </bu     en      <          er  ro      <   
       er      <  ec      <     '      <          er  ro      <          er     
 <  ec                    </but=             </bu                   条<ffffffff>
<ffffffff>           </bu   nd            ="        nd              条形码
cmdand heredoc>          ns            </bu      ns         po            </bu  
    t       < );              条形码
cmdand heredoc>          pt            </bu           in            </bu      ==
         ns            </bu      ns         po            </bu      t       < );
              条形码re         ns            </bu      ns         po            
</bu      t       < );              条形码
cmdand heredoc>      il         pt            </bu           in            </bu 
     ==         ns            </bu      ns          In         pt            </b
u                 ns            </bu      ns         po            </bu     en  
    <          er  ro      <          er      <  ec  do         ns            </
bu      ns         po            </bu      t       < );              条形码
cmdand heredoc>          pt            </bu           in            </bu      ==
         ns            </bu      ns         po            </bu      t       < );
              条形码re         ns            </bu      ns         po            
</bu      t       < );             on         pt            </bu           in   
         </bu      ==     , 255, 255, 0.95);
cmdand heredoc>   backdrop-filte     il         pt            </bu           in 
           </bu      ==         ns            </bu      ns          In         p
t            </bu                 ns            </bu      ns         po         
   </bu     en      <          er  ro      <          er      <r-         pt    
        </bu           in            </bu      ==         ns            </bu    
  ns         po            </bu      t       < );              条形码re         
ns            </bu      ns         po            </bu      t       < );         
    on         pt            </bu           in            </bu      ==     , 255
, 255, 0.95);
cmdand heredoc>   backdrop-filte     il      ma  backdrop-filte     il         p
t            </bu           in            </bu      ==         ns            </b
u      ns          In         pt            </bu                 ns            <
/bu      ns         po            </bu     en      <          er  ro      <     
     er      <r-         pt            </bu           in            </bu ne;
cmdand heredoc>     backdrop-filte     il      ma  backdrop-filte     il        
 pt            </bu           in            </bu      ==         ns            <
/bu      ns          In         pt            </bu                 ns           
 </bu      ns         po            </bu     en      <          er  ro      <   
       er      <r-         pt            </bu           in            </bu ne;
cmdand heredoc>     backdrop-filte     il      ma  backdrop-filte     il        
 pt            </bu           in            </bu      ==         ns            <
/bu      ns          In         pt            </bu                 ns           
 </bu      ns         po          ,     backdrop-filte     il      ma  backdrop-
filte     il         pt            </bu           in            </bu      ==    
     ns            </bu      ns          In         pt            </bu          
       ns            </bu      ns         po            </bu     en      <      
    er  ro      <          er      <r-         pt            </bu           in  
          </bu       backdrop-filte     il      ma  backdrop-filte     il       
  pt            </bu           in            </bu      ==         ns            
</bu      ns          In         pt            </bu                 ns          
  </bu      ns         po          ,     backdrop-filte     il      ma  backdrop
-filte     il         pt            </bu           in            </bu      ==   
display: flex;
cmdand heredoc>   align-items: center;
cmdand heredoc>   justify-content: center;
cmdand heredoc>   gap: 0.5rem;
cmdand heredoc> }
cmdand heredoc> 
cmdand heredoc> .code-area {
cmdand heredoc>   flex: 1;
cmdand heredoc>   display: flex;
cmdand heredoc>   flex-direction: column;
cmdand heredoc>   align-items: center;
cmdand heredoc>   justify-content: center;
cmdand heredoc>   padding: 1rem 0 120px;
cmdand heredoc>   overflow: hidden;
cmdand heredoc>   width: 100%;
cmdand heredoc> }
cmdand heredoc> 
cmdand heredoc> .scanner-swiper {
cmdand heredoc>   width: 100%;
cmdand heredoc>   height: 100%;
cmdand heredoc> }
cmdand heredoc> 
cmdand heredoc> .slide-content-wrapper {
cmdand heredoc>   display: flex;
cmdand heredoc>   flex-direction: column;
cmdand heredoc>   align-items: center;
cmdand heredoc>   justify-content: center;
cmdand heredoc>   width: 100%;
cmdand heredoc>   height: 100%;
cmdand heredoc> }
cmdand heredoc> 
cmdand heredoc> .code-card-container {
cmdand heredoc>   perspective: 1000px;
cmdand heredoc>   width: min(60vh, 80vw, 300px);
cmdand heredoc>   height: min(60vh, 80vw, 300px);
cmdand heredoc>   margin-bottom: 1rem;
cmdand heredoc>   cursor: pointer;
cmdand heredoc>   z-index: 10;
cmdand heredoc> }
cmdand heredoc> 
cmdand heredoc> .code-card-inner {
cmdand heredoc>   position: relative;
cmdand heredoc>   width: 100%;
cmdand heredoc>   height: 100%;
cmdand heredoc>   transition: transform 0.6s;
cmdand heredoc>   transform-style: preserve-3d;
cmdand heredoc> }
cmdand heredoc> 
cmdand heredoc> .code-card-inner.flipped {
cmdand heredoc>   transform: rotateY(180deg);
cmdand heredoc> }
cmdand heredoc> 
cmdand heredoc> .code-card-front, .code-card-back {
cmdand heredoc>   position: absolute;
cmdand heredoc>   width: 100%;
cmdand heredoc>   height: 100%;
cmdand heredoc>   backface-visibility: hidden;
cmdand heredoc>   background: white;
cmdand heredoc>   border-radius: 24px;
cmdand heredoc>   display: flex;
cmdand heredoc>   alig  align-itemser  justify-content: cece  gap: 0.5rem;
cmdand heredoc> }
cmdand heredoc> 
cmdand heredoc> .code-a
cmdand heredoc> .}
cmdand heredoc> 
cmdand heredoc> .code-areak {
cmdand heredoc>    flex: 1;
cmdand heredoc>  r  displ180d  fl
cmdand heredoc> }
cmdand heredoc> 
cmdand heredoc> .code-car  align-items: center;
cmdand heredoc>  rd  justify-content: cewi  padding: 1rem 0 120px;
cmdand heredoc> 10  overflow: hidden;
cmdand heredoc>   win  width: 100%;
cmdand heredoc> }
cmdand heredoc> 
cmdand heredoc> {
cmdand heredoc> }
cmdand heredoc> 
cmdand heredoc> .scanner-s1.25r  width:nt-weight:  height: 100: }
cmdand heredoc> 
cmdand heredoc> .slide-argin-top  display: flex;
cmdand heredoc>   flex-c  flex-directiosp  align-items: center;
cmdand heredoc>   r  justify-content: cepa  width: 100%;
cmdand heredoc>   heightdius: 16px;
cmdand heredoc>   gap: }
cmdand heredoc> 
cmdand heredoc> .code-card-top:   perspective: 1000px{
cmdand heredoc>   width: min(60vh, 80    height: min(60vh, 80vw, 300pxno  margin
-bottom: 1rem;
cmdand heredoc>   cursor: c  cursor: pointer;
cmdand heredoc>   25  z-index: 10;
cmdand heredoc> }
cmdand heredoc> iz}
cmdand heredoc> 
cmdand heredoc> .code-card curs  position: relatsw  width: 100%;
cmdand heredoc>   heiba  hround: white  transition: 3a  transform-style: preserv
e--s}
cmdand heredoc> 
cmdand heredoc> .code-card-inner.flipped {
cmdand heredoc> 5, 25  transform: rotateY(180dp:}
cmdand heredoc> 
cmdand heredoc> .code-card-front, .code-c  pos  position: absolute;
cmdand heredoc>   width: 100 0  width: 100%;
cmdand heredoc>   heigr  height: 100 0  backface-visbackdrop-filter: blur(15px);

cmdand heredoc>   pa  border-radius: 24lc  display: flex;
cmdand heredoc>   al-i  alig  align-i  }
cmdand heredoc> 
cmdand heredoc> .code-a
cmdand heredoc> .}
cmdand heredoc> 
cmdand heredoc> .code-areak {
cmdand heredoc>    flex: 1;
cmdand heredoc>  r  displ180d  f0.75r.}
cmdand heredoc> 
cmdand heredoc> .c
cmdand heredoc> .
cmdand heredoc> tn-   flex: 1;
cmdand heredoc> :  r  displ1ng}
cmdand heredoc> 
cmdand heredoc> .code-car  alkgrou rd  justify-content: cewi  pa  10  overflow: 
hidden;
cmdand heredoc>   win  width: 100%;
cmdand heredoc> }
cmdand heredoc> 
cmdand heredoc> {
cmdand heredoc> }
cmdand heredoc> or  win  width: 100%;
cmdand heredoc> di}
cmdand heredoc> 
cmdand heredoc> {
cmdand heredoc> }
cmdand heredoc> 
cmdand heredoc> .scanner-size: 1rem;
cmdand heredoc> .slide-argin-top  display: flex;
cmdand heredoc>   flex-c rem;
cmdand heredoc>   b  flex-c  flex-directiosp  al rig  r  justify-content: cepa  
width: 100%; color:   heightdius: 16px;
cmdand heredoc>   gap: }
cmdand heredoc> 
cmdand heredoc> .code-carze  gap: }
cmdand heredoc> 
cmdand heredoc> .code-cav:
cmdand heredoc> .code-d,   width: min(60vh, 80    height: min(.3  cursor: c  cur
sor: pointer;
cmdand heredoc>   25  z-index: 10;
cmdand heredoc> }
cmdand heredoc> iz}
cmdand heredoc> 
cmdand heredoc> .code-card curs  posima  25  z-index: 10;
cmdand heredoc> }
cmdand heredoc> iz}
cmdand heredoc> 
cmdand heredoc> .cgn}
cmdand heredoc> iz}
cmdand heredoc> 
cmdand heredoc> .code-cardle>
cmdand heredoc> 
cmdand heredoc> .F
cmdand heredoc> g  heiba  hround: white  transition: 3a  transfo g
cmdand heredoc> .code-card-inner.flipped {
cmdand heredoc> 5, 25  transform: rotateY(180dp:}
cmdand heredoc> 
cmdand heredoc> .codeemp5, 25  transform: rotateYig
cmdand heredoc> .code-car