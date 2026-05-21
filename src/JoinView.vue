<template>
  <div class="join-container" :class="{ 'dark-mode': isDark }">
    <div class="join-card ios-card">
      <div class="join-header">
        <div class="join-title">🔥 录入我的拼团信息</div>
        <div class="join-status" :class="statusClass">
          <span class="status-dot"></span>
          <span class="status-text">{{ statusText }}</span>
        </div>
      </div>

      <div v-if="!submitted">
        <div class="join-section">
          <label class="ios-label">头像（选填）</label>
          <div class="join-avatar-row">
            <div
              class="join-avatar-slot"
              :class="{ 'has-image': form.avatar }"
              :style="form.avatar ? { backgroundImage: `url(${form.avatar})` } : null"
              tabindex="0"
              @click="triggerAvatarPicker"
              @paste="handleAvatarPaste"
              @keydown.delete.prevent="form.avatar = ''"
              @keydown.backspace.prevent="form.avatar = ''"
            >
              <span v-if="!form.avatar" class="avatar-placeholder">＋</span>
              <button
                v-if="form.avatar"
                class="avatar-clear"
                @click.stop="form.avatar = ''"
              >✕</button>
            </div>
            <div class="avatar-tip">
              <div>点击上传 / 拍照</div>
              <div class="avatar-tip-sub">或粘贴图片</div>
            </div>
            <input
              ref="avatarInputRef"
              type="file"
              accept="image/*"
              capture="user"
              style="display: none"
              @change="handleAvatarFile"
            />
          </div>
        </div>

        <div class="join-section">
          <label class="ios-label">姓名 <span class="required">*</span></label>
          <input
            v-model="form.name"
            placeholder="输入你的名字"
            class="ios-input"
            maxlength="20"
          />
        </div>

        <div class="join-section">
          <label class="ios-label">ID（选填）</label>
          <input
            v-model="form.userId"
            placeholder="游戏 ID / 备注"
            class="ios-input"
            maxlength="32"
          />
        </div>

        <div class="join-section">
          <label class="ios-label">需求精灵 <span class="required">*</span></label>
          <div class="elf-options">
            <button
              type="button"
              class="elf-option"
              :class="{ active: form.needElf === 'elf1' }"
              @click="form.needElf = 'elf1'"
            >{{ elf1Name }}</button>
            <button
              type="button"
              class="elf-option"
              :class="{ active: form.needElf === 'elf2' }"
              @click="form.needElf = 'elf2'"
            >{{ elf2Name }}</button>
            <button
              type="button"
              class="elf-option"
              :class="{ active: form.needElf === 'any' }"
              @click="form.needElf = 'any'"
            >都行</button>
          </div>
        </div>

        <div v-if="errorText" class="join-error">{{ errorText }}</div>

        <button
          class="ios-btn ios-btn-primary ios-btn-large submit-btn"
          :disabled="!canSubmit || sending"
          @click="submit"
        >
          {{ sending ? '提交中...' : '提交给主机' }}
        </button>
      </div>

      <div v-else class="join-success">
        <div class="success-icon">✓</div>
        <div class="success-title">已提交</div>
        <div class="success-sub">
          你的信息「{{ lastSubmittedName }}」已发送到主机。<br />
          可以关闭页面，或继续录入下一个人。
        </div>
        <button class="ios-btn ios-btn-secondary" @click="resetForm">再录一个</button>
      </div>
    </div>

    <div class="join-footer">
      数据仅在浏览器之间传输，不会保存到任何服务器
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, onBeforeUnmount } from 'vue'
import Peer from 'peerjs'

const props = defineProps({
  sessionId: { type: String, required: true },
  elf1Name: { type: String, default: '雪熊' },
  elf2Name: { type: String, default: '火龙' },
})

const isDark = ref(false)
const peer = ref(null)
const conn = ref(null)
const status = ref('connecting')
const errorText = ref('')
const submitted = ref(false)
const sending = ref(false)
const lastSubmittedName = ref('')
const avatarInputRef = ref(null)

const elf1Name = computed(() => props.elf1Name || '雪熊')
const elf2Name = computed(() => props.elf2Name || '火龙')

const form = reactive({
  name: '',
  userId: '',
  avatar: '',
  needElf: 'elf1',
})

const statusText = computed(() => {
  switch (status.value) {
    case 'connecting': return '连接主机中...'
    case 'ready': return '已连接，可以提交'
    case 'sent': return '已发送'
    case 'error': return '连接失败：' + errorText.value
    default: return ''
  }
})

const statusClass = computed(() => {
  if (status.value === 'ready' || status.value === 'sent') return 'ok'
  if (status.value === 'error') return 'err'
  return 'pending'
})

const canSubmit = computed(() => status.value === 'ready' && form.name.trim().length > 0)

if (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches) {
  isDark.value = true
}

function setupPeer() {
  try {
    peer.value = new Peer()
  } catch (e) {
    status.value = 'error'
    errorText.value = e?.message || '初始化失败'
    return
  }

  peer.value.on('open', () => {
    const c = peer.value.connect(props.sessionId, { reliable: true })
    conn.value = c
    c.on('open', () => {
      status.value = 'ready'
      errorText.value = ''
    })
    c.on('error', (err) => {
      status.value = 'error'
      errorText.value = err?.message || '连接错误'
    })
    c.on('close', () => {
      if (status.value !== 'sent') {
        status.value = 'error'
        errorText.value = '主机已断开'
      }
    })
  })

  peer.value.on('error', (err) => {
    status.value = 'error'
    errorText.value = err?.message || '网络错误'
  })
}

function triggerAvatarPicker() {
  if (avatarInputRef.value) {
    avatarInputRef.value.value = ''
    avatarInputRef.value.click()
  }
}

async function handleAvatarFile(event) {
  const file = event.target.files && event.target.files[0]
  event.target.value = ''
  if (!file || !file.type.startsWith('image/')) return
  try {
    form.avatar = await fileToCompressedAvatar(file)
  } catch (e) {
    errorText.value = '头像处理失败'
  }
}

async function handleAvatarPaste(event) {
  const items = event.clipboardData && event.clipboardData.items
  if (!items) return
  for (const item of items) {
    if (item.type && item.type.startsWith('image/')) {
      const file = item.getAsFile()
      if (file) {
        event.preventDefault()
        try {
          form.avatar = await fileToCompressedAvatar(file)
        } catch (e) {
          errorText.value = '头像处理失败'
        }
        return
      }
    }
  }
}

function fileToDataUrl(file) {
  return new Promise((resolve, reject) => {
    const r = new FileReader()
    r.onload = (e) => resolve(String(e.target.result || ''))
    r.onerror = () => reject(new Error('读取失败'))
    r.readAsDataURL(file)
  })
}

async function fileToCompressedAvatar(file, maxSize = 128) {
  const dataUrl = await fileToDataUrl(file)
  return new Promise((resolve) => {
    const img = new Image()
    img.onload = () => {
      const w0 = img.naturalWidth || img.width
      const h0 = img.naturalHeight || img.height
      if (!w0 || !h0) return resolve(dataUrl)
      const side = Math.min(w0, h0)
      const sx = (w0 - side) / 2
      const sy = (h0 - side) / 2
      const target = Math.min(maxSize, side)
      const canvas = document.createElement('canvas')
      canvas.width = target
      canvas.height = target
      const ctx = canvas.getContext('2d')
      ctx.drawImage(img, sx, sy, side, side, 0, 0, target, target)
      try { resolve(canvas.toDataURL('image/jpeg', 0.85)) } catch { resolve(dataUrl) }
    }
    img.onerror = () => resolve(dataUrl)
    img.src = dataUrl
  })
}

function submit() {
  if (!canSubmit.value || !conn.value) return
  sending.value = true
  errorText.value = ''
  try {
    conn.value.send({
      type: 'person',
      name: form.name.trim(),
      userId: form.userId.trim(),
      avatar: form.avatar,
      needElf: form.needElf,
    })
    lastSubmittedName.value = form.name.trim()
    submitted.value = true
    status.value = 'sent'
  } catch (e) {
    errorText.value = '发送失败：' + (e?.message || e)
  } finally {
    sending.value = false
  }
}

function resetForm() {
  form.name = ''
  form.userId = ''
  form.avatar = ''
  form.needElf = 'elf1'
  submitted.value = false
  if (conn.value && conn.value.open) {
    status.value = 'ready'
  } else {
    setupPeer()
  }
}

onMounted(() => {
  setupPeer()
})

onBeforeUnmount(() => {
  try { conn.value && conn.value.close && conn.value.close() } catch {}
  try { peer.value && peer.value.destroy && peer.value.destroy() } catch {}
})
</script>

<style scoped>
.join-container {
  min-height: 100vh;
  background: var(--ios-bg-gradient);
  padding: 20px 16px 40px;
  font-family: var(--ios-font);
}

.ios-card {
  background: var(--ios-card-solid);
  border-radius: var(--ios-radius);
  border: 1px solid var(--ios-card-border);
  box-shadow: var(--ios-shadow);
}

.ios-label {
  display: block;
  font-size: 13px;
  font-weight: 500;
  color: var(--ios-text-secondary);
  margin-bottom: 6px;
}

.ios-input {
  width: 100%;
  padding: 12px 14px;
  border-radius: var(--ios-radius-xs);
  border: 1px solid var(--ios-card-border);
  background: var(--ios-fill);
  color: var(--ios-text);
  font-size: 16px;
  font-family: var(--ios-font);
  outline: none;
  transition: all 0.3s var(--ios-ease);
}

.ios-input:focus {
  border-color: var(--ios-blue);
  box-shadow: 0 0 0 3px rgba(0, 122, 255, 0.12);
  background: var(--ios-card-solid);
}

.ios-input::placeholder {
  color: var(--ios-text-tertiary);
}

.ios-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 10px 18px;
  border-radius: 20px;
  border: none;
  font-size: 14px;
  font-weight: 600;
  font-family: var(--ios-font);
  cursor: pointer;
  transition: all 0.3s var(--ios-spring);
}

.ios-btn:active {
  transform: scale(0.96);
}

.ios-btn-primary {
  background: var(--ios-blue);
  color: #fff;
}

.ios-btn-secondary {
  background: var(--ios-fill-secondary);
  color: var(--ios-blue);
}

.ios-btn-large {
  padding: 14px 28px;
  font-size: 17px;
  border-radius: var(--ios-radius);
}

.join-card {
  max-width: 480px;
  margin: 0 auto;
  padding: 20px;
}

.join-header {
  margin-bottom: 18px;
  border-bottom: 1px solid var(--ios-separator);
  padding-bottom: 14px;
}

.join-title {
  font-size: 19px;
  font-weight: 700;
  color: var(--ios-text);
  letter-spacing: -0.3px;
  margin-bottom: 8px;
}

.join-status {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 13px;
  font-weight: 500;
  padding: 4px 10px;
  border-radius: 10px;
}

.join-status .status-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
}

.join-status.pending {
  background: rgba(255, 149, 0, 0.1);
  color: var(--ios-orange);
}

.join-status.pending .status-dot {
  background: var(--ios-orange);
  animation: pulse 1.5s ease-in-out infinite;
}

.join-status.ok {
  background: rgba(52, 199, 89, 0.1);
  color: var(--ios-green);
}

.join-status.ok .status-dot {
  background: var(--ios-green);
}

.join-status.err {
  background: rgba(255, 59, 48, 0.1);
  color: var(--ios-red);
}

.join-status.err .status-dot {
  background: var(--ios-red);
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.4; }
}

.join-section {
  margin-bottom: 16px;
}

.required {
  color: var(--ios-red);
  font-weight: 600;
}

.join-avatar-row {
  display: flex;
  align-items: center;
  gap: 14px;
}

.join-avatar-slot {
  position: relative;
  width: 72px;
  height: 72px;
  border-radius: 50%;
  background: var(--ios-card-solid);
  border: 1.5px dashed var(--ios-card-border);
  background-size: cover;
  background-position: center;
  cursor: pointer;
  outline: none;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  overflow: hidden;
  transition: all 0.25s var(--ios-ease);
}

.join-avatar-slot:hover,
.join-avatar-slot:focus-visible {
  border-color: var(--ios-blue);
  background-color: rgba(0, 122, 255, 0.04);
}

.join-avatar-slot.has-image {
  border-style: solid;
}

.avatar-placeholder {
  font-size: 28px;
  color: var(--ios-text-tertiary);
  font-weight: 300;
  line-height: 1;
}

.avatar-clear {
  position: absolute;
  top: 4px;
  right: 4px;
  width: 22px;
  height: 22px;
  border-radius: 50%;
  border: none;
  background: rgba(0, 0, 0, 0.55);
  color: #fff;
  font-size: 11px;
  cursor: pointer;
}

.avatar-tip {
  font-size: 13px;
  color: var(--ios-text-secondary);
  line-height: 1.5;
}

.avatar-tip-sub {
  font-size: 11px;
  color: var(--ios-text-tertiary);
}

.elf-options {
  display: flex;
  gap: 8px;
}

.elf-option {
  flex: 1;
  padding: 12px 8px;
  border-radius: var(--ios-radius-xs);
  border: 1px solid var(--ios-card-border);
  background: var(--ios-fill);
  color: var(--ios-text-secondary);
  font-size: 14px;
  font-weight: 600;
  font-family: var(--ios-font);
  cursor: pointer;
  transition: all 0.25s var(--ios-ease);
}

.elf-option.active {
  background: var(--ios-blue);
  border-color: var(--ios-blue);
  color: #fff;
}

.join-error {
  background: rgba(255, 59, 48, 0.08);
  border: 1px solid rgba(255, 59, 48, 0.15);
  color: var(--ios-red);
  padding: 10px 12px;
  border-radius: var(--ios-radius-xs);
  font-size: 13px;
  margin-bottom: 14px;
}

.submit-btn {
  width: 100%;
  margin-top: 8px;
}

.submit-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.join-success {
  text-align: center;
  padding: 28px 0 12px;
}

.success-icon {
  width: 64px;
  height: 64px;
  margin: 0 auto 16px;
  border-radius: 50%;
  background: var(--ios-green);
  color: #fff;
  font-size: 36px;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
}

.success-title {
  font-size: 20px;
  font-weight: 700;
  color: var(--ios-text);
  margin-bottom: 8px;
}

.success-sub {
  font-size: 14px;
  color: var(--ios-text-secondary);
  line-height: 1.6;
  margin-bottom: 18px;
}

.join-footer {
  text-align: center;
  font-size: 12px;
  color: var(--ios-text-tertiary);
  margin-top: 20px;
  padding: 0 16px;
}
</style>
