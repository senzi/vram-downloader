<!-- VRAM Downloader Component -->
<template>
  <div class="container">
    <!-- Header -->
    <header class="header">
      <h1>VRAM Downloader</h1>
      <p>为您的AI模型提供虚拟显存解决方案</p>
    </header>

    <div class="content-wrapper">
      <div class="main-content">
        <!-- Package Grid -->
        <div class="grid grid-2">
          <div v-for="pkg in packages" :key="pkg.size" class="card package">
            <div class="package-content">
              <div class="package-info">
                <h3>{{ pkg.name }}</h3>
                <p class="description">{{ pkg.description }}</p>
                <div class="price">
                  <span class="current-price">¥{{ pkg.price.toFixed(2) }}</span>
                  <span class="original-price">¥{{ (pkg.price * 1.4).toFixed(2) }}</span>
                </div>
              </div>
              <div class="package-icon">
                <img :src="pkg.icon" :alt="pkg.name" />
              </div>
            </div>
            <button class="btn btn-secondary" :disabled="isDownloading" @click="startDownload(pkg.size)">
              {{ isDownloading ? '下载中...' : `立即下载 ${pkg.size}GB` }}
            </button>
          </div>
        </div>

        <!-- Custom Section -->
        <div class="card custom-card">
          <div class="custom-content">
            <div class="custom-header">
              <h3>自定义VRAM大小</h3>
              <p class="download-time">
                预计下载时间: {{ calculateDownloadTime() }}
              </p>
            </div>
            <div class="range-container">
              <input type="range" v-model="customSize" min="8" max="128" step="1" />
              <span class="size-display">{{ customSize }}GB</span>
            </div>
          </div>
          <button class="btn btn-primary" :disabled="isDownloading" @click="startDownload(customSize)">
            {{ isDownloading ? '下载中...' : '下载自定义VRAM' }}
          </button>
        </div>
      </div>

      <!-- Footer -->
      <footer class="footer">
        <div class="footer-content">
          <p class="copyright">© 2025 VRAM Downloader</p>
          <div class="footer-links">
            <a href="https://github.com/senzi/vram-downloader" target="_blank" rel="noopener noreferrer">
              GitHub
            </a>
            <span class="separator">|</span>
            <span>MIT License</span>
            <span class="separator">|</span>
            <span>By senzi & Windsurf</span>
          </div>
        </div>
      </footer>
    </div>

    <!-- Download Progress Modal -->
    <div v-if="showProgress || showModal" class="modal-overlay" @click="closeModal">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <button class="close-button" @click="closeModal">
            <svg viewBox="0 0 24 24">
              <path d="M18 6L6 18M6 6l12 12" />
            </svg>
          </button>
        </div>

        <!-- Download Progress -->
        <div v-if="showProgress" class="modal-body">
          <div class="progress-container">
            <div class="progress-bar">
              <div class="progress-fill" :style="{ width: `${downloadProgress}%` }"></div>
            </div>
            <div class="progress-text">{{ downloadProgress }}%</div>
          </div>
          <p class="status-text">{{ downloadMessages[currentMessageIndex] }}</p>
          <button class="btn btn-secondary cancel-button" @click="closeModal">
            取消下载
          </button>
        </div>

        <!-- Download Complete -->
        <div v-else class="modal-body completion-content">
          <div class="success-icon">
            <svg viewBox="0 0 24 24">
              <path d="M20 6L9 17l-5-5" />
            </svg>
          </div>
          <h3 class="completion-message">下载完成！</h3>
          <p class="completion-note">
            这是一个模拟下载器，用来调侃当今AI大模型动辄需要几十GB显存的现状 😄<br>
            显存是硬件资源，并不能通过下载来增加哦！<br>
            您可以查看已下载的文件了解更多信息~
          </p>
          <button class="btn btn-primary confirm-button" @click="closeModal">
            我明白了
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onUnmounted } from 'vue'

const packages = [
  {
    name: '入门版',
    description: '轻松加载7B模型',
    size: 16,
    price: 299.99,
    icon: '/gpu-basic.svg'
  },
  {
    name: '进阶版',
    description: '完美支持13B模型',
    size: 24,
    price: 499.99,
    icon: '/gpu-pro.svg'
  },
  {
    name: '专业版',
    description: '流畅运行33B模型',
    size: 48,
    price: 899.99,
    icon: '/gpu-advanced.svg'
  },
  {
    name: '企业版',
    description: '为70B模型而生',
    size: 80,
    price: 1499.99,
    icon: '/gpu-enterprise.svg'
  }
]

const downloadMessages = [
  '正在分配虚拟显存空间...',
  '正在初始化显存缓存...',
  '正在配置虚拟地址...',
  '正在优化内存访问...',
  '正在进行最终处理...'
]

const customSize = ref(32)
const showModal = ref(false)
const showProgress = ref(false)
const isDownloading = ref(false)
const downloadProgress = ref(0)
const lastDownloadSize = ref(0)
const currentMessageIndex = ref(0)
let downloadTimer = null
let messageTimer = null

const calculateDownloadTime = () => {
  const size = customSize.value
  const minutes = Math.ceil(size / 16)
  return `${minutes} 秒`
}

const startDownload = async (size) => {
  if (isDownloading.value) return

  isDownloading.value = true
  showProgress.value = true
  lastDownloadSize.value = size
  downloadProgress.value = 0
  currentMessageIndex.value = 0

  const duration = Math.max(5000, size * 100) // 最少5秒，每GB 100ms
  const steps = 50
  const interval = duration / steps

  messageTimer = setInterval(() => {
    currentMessageIndex.value = (currentMessageIndex.value + 1) % downloadMessages.length
  }, duration / 6)

  for (let i = 1; i <= steps; i++) {
    if (!isDownloading.value) break // 检查是否取消下载
    await new Promise(resolve => {
      downloadTimer = setTimeout(resolve, interval)
    })
    downloadProgress.value = Math.round((i / steps) * 100)
  }

  clearInterval(messageTimer)
  messageTimer = null

  if (isDownloading.value) { // 只有在未取消的情况下才完成下载
    // 准备下载文件
    const content = `恭喜您！您已成功下载 ${size}GB 虚拟显存！\n\n` +
      `这是一个模拟下载器，用来调侃当今AI大模型对显存的巨大需求。\n` +
      `实际上，显存是硬件资源，并不能通过下载来增加哦！😄\n\n` +
      `下载时间: ${new Date().toLocaleString()}`

    const blob = new Blob([content], { type: 'text/plain' })
    const url = URL.createObjectURL(blob)
    const a = document.createElement('a')
    a.href = url
    a.download = `VRAM_${size}GB_${Math.random().toString(36).substring(7)}.txt`
    document.body.appendChild(a)
    a.click()
    document.body.removeChild(a)
    URL.revokeObjectURL(url)

    // 显示完成模态框
    await new Promise(resolve => setTimeout(resolve, 800))
    showProgress.value = false
    showModal.value = true
  }

  isDownloading.value = false
  downloadProgress.value = 0
}

const closeModal = () => {
  if (isDownloading.value) {
    // 取消下载
    isDownloading.value = false
    if (downloadTimer) {
      clearTimeout(downloadTimer)
      downloadTimer = null
    }
    if (messageTimer) {
      clearInterval(messageTimer)
      messageTimer = null
    }
  }
  showModal.value = false
  showProgress.value = false
  downloadProgress.value = 0
}

onUnmounted(() => {
  // 清理定时器
  if (downloadTimer) {
    clearTimeout(downloadTimer)
  }
  if (messageTimer) {
    clearInterval(messageTimer)
  }
})
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  padding: 1.5rem;
}

.content-wrapper {
  width: 100%;
  max-width: 800px;
  margin-top: 2rem;
  display: flex;
  flex-direction: column;
  gap: 3rem;
}

.header {
  text-align: center;
}

.header h1 {
  font-size: 2.5rem;
  font-weight: bold;
  margin-bottom: 0.5rem;
  color: var(--primary-color);
  text-shadow: 0 0 10px rgba(118, 185, 0, 0.3);
}

.header p {
  font-size: 1.125rem;
  color: var(--text-light);
}

.main-content {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.grid {
  display: grid;
  gap: 1.5rem;
}

.grid-2 {
  grid-template-columns: repeat(2, 1fr);
}

.card {
  background: var(--card-bg);
  border: 1px solid var(--border-color);
  border-radius: 1rem;
  padding: 1.5rem;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.3);
  transition: all 0.3s ease;
}

.card:hover {
  border-color: var(--primary-color);
  box-shadow: 0 0 20px rgba(118, 185, 0, 0.2);
}

.package {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.package-content {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 1rem;
}

.package-info {
  flex: 1;
}

.package-info h3 {
  font-size: 1.5rem;
  font-weight: bold;
  margin-bottom: 0.5rem;
  color: var(--text-color);
}

.description {
  color: var(--text-light);
  margin-bottom: 1rem;
  font-size: 1rem;
}

.price {
  font-weight: bold;
  display: flex;
  align-items: baseline;
  gap: 0.5rem;
}

.current-price {
  font-size: 1.5rem;
  color: var(--primary-color);
  text-shadow: 0 0 10px rgba(118, 185, 0, 0.3);
}

.original-price {
  font-size: 0.875rem;
  color: var(--price-original);
  text-decoration: line-through;
}

.package-icon {
  width: 4rem;
  height: 4rem;
  filter: drop-shadow(0 0 5px rgba(118, 185, 0, 0.3));
}

.package-icon img {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.btn {
  width: 100%;
  padding: 0.75rem;
  font-size: 1rem;
  border: none;
  border-radius: 0.5rem;
  cursor: pointer;
  transition: all 0.3s ease;
  background: linear-gradient(90deg, var(--primary-color) 0%, var(--primary-hover) 100%);
  color: var(--text-color);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3);
}

.btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
  background: var(--text-disabled);
}

.btn-secondary {
  background: linear-gradient(90deg, var(--secondary-color) 0%, var(--secondary-hover) 100%);
}

.custom-card {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.custom-content {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.custom-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.custom-header h3 {
  font-size: 1.25rem;
  font-weight: bold;
  color: var(--text-color);
}

.range-container {
  display: flex;
  align-items: center;
  gap: 1rem;
}

input[type="range"] {
  flex: 1;
  height: 0.5rem;
  border-radius: 0.25rem;
  background: var(--border-color);
  cursor: pointer;
  appearance: none;
}

input[type="range"]::-webkit-slider-thumb {
  appearance: none;
  width: 1.25rem;
  height: 1.25rem;
  border-radius: 50%;
  background: var(--primary-color);
  box-shadow: 0 0 10px rgba(118, 185, 0, 0.3);
  cursor: pointer;
  border: 2px solid var(--text-color);
}

.size-display {
  font-size: 1rem;
  font-weight: bold;
  color: var(--text-color);
  min-width: 4rem;
}

.download-time {
  color: var(--text-light);
  font-size: 0.875rem;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: var(--modal-bg);
  backdrop-filter: blur(5px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: var(--card-bg);
  border: 1px solid var(--border-color);
  border-radius: 1rem;
  padding: 2rem;
  width: 90%;
  max-width: 400px;
  min-height: 300px;
  position: relative;
  display: flex;
  flex-direction: column;
  box-shadow: 0 0 20px rgba(118, 185, 0, 0.15);
}

.modal-header {
  position: absolute;
  top: 1rem;
  right: 1rem;
}

.close-button {
  width: 2rem;
  height: 2rem;
  border: none;
  background: none;
  cursor: pointer;
  color: var(--text-light);
  padding: 0.25rem;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.close-button:hover {
  color: var(--text-color);
  background: rgba(255, 255, 255, 0.1);
}

.close-button svg {
  width: 1.25rem;
  height: 1.25rem;
  stroke: currentColor;
  stroke-width: 2;
  stroke-linecap: round;
  stroke-linejoin: round;
  fill: none;
}

.modal-body {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 1.5rem;
  margin-top: 1rem;
}

.progress-container {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 1rem;
  align-items: center;
}

.progress-bar {
  width: 100%;
  height: 0.5rem;
  background: var(--border-color);
  border-radius: 1rem;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, var(--primary-color) 0%, var(--primary-hover) 100%);
  transition: width 0.3s ease;
}

.progress-text {
  font-size: 1.5rem;
  font-weight: bold;
  color: var(--primary-color);
  text-shadow: 0 0 10px rgba(118, 185, 0, 0.3);
}

.status-text {
  color: var(--text-light);
  font-size: 1rem;
  text-align: center;
}

.completion-content {
  text-align: center;
  padding: 1rem 0;
}

.success-icon {
  width: 4rem;
  height: 4rem;
  color: var(--success);
  margin: 0 auto;
  filter: drop-shadow(0 0 5px rgba(118, 185, 0, 0.3));
}

.success-icon svg {
  width: 100%;
  height: 100%;
  stroke: currentColor;
  stroke-width: 2;
  stroke-linecap: round;
  stroke-linejoin: round;
  fill: none;
}

.completion-message {
  font-size: 1.5rem;
  font-weight: bold;
  color: var(--text-color);
  margin-bottom: 0.5rem;
  text-shadow: 0 0 10px rgba(118, 185, 0, 0.3);
}

.completion-note {
  color: var(--text-light);
  margin-bottom: 1.5rem;
  line-height: 1.6;
}

.confirm-button {
  min-width: 120px;
}

.cancel-button {
  margin-top: 1rem;
}

.footer {
  margin-top: 4rem;
  padding: 1.5rem;
  text-align: center;
  color: var(--text-light);
}

.footer-content {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  align-items: center;
}

.footer-links {
  display: flex;
  gap: 0.75rem;
  align-items: center;
  flex-wrap: wrap;
  justify-content: center;
}

.footer-links a {
  color: var(--primary-color);
  text-decoration: none;
  transition: color 0.2s;
}

.footer-links a:hover {
  color: var(--primary-hover);
  text-shadow: 0 0 10px rgba(118, 185, 0, 0.3);
}

.separator {
  color: var(--border-color);
}

@media (max-width: 768px) {
  .grid-2 {
    grid-template-columns: 1fr;
  }

  .custom-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.5rem;
  }

  .package-info h3 {
    font-size: 1.25rem;
  }

  .modal-content {
    width: 95%;
    min-height: 280px;
    padding: 1.5rem;
  }

  .footer {
    margin-top: 3rem;
    padding: 1rem;
  }
}
</style>
