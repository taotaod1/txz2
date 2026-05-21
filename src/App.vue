<template>
  <div class="app-container" :class="{ 'dark-mode': isDark }">
    <div class="app-content">
      <div class="content-wrapper">

        <section class="ios-card config-section">
          <div class="section-header">
            <h2 class="section-title">档次选择</h2>
          </div>
          <div class="segment-control">
            <button
              class="segment-btn"
              :class="{ active: tier === 'normal' }"
              @click="tier = 'normal'"
            >
              <span class="segment-text">普通版</span>
              <span class="segment-price">{{ PRICE.normal.pass }}/{{ PRICE.normal.coupon }}元</span>
            </button>
            <button
              class="segment-btn"
              :class="{ active: tier === 'premium' }"
              @click="tier = 'premium'"
            >
              <span class="segment-text">豪华版</span>
              <span class="segment-price">{{ PRICE.premium.pass }}/{{ PRICE.premium.coupon }}元</span>
            </button>
          </div>
          <div class="elf-name-row">
            <div class="elf-name-field">
              <label class="ios-label">精灵1名称</label>
              <input v-model="elfName1" placeholder="精灵1" class="ios-input" />
            </div>
            <div class="elf-name-field">
              <label class="ios-label">精灵2名称</label>
              <input v-model="elfName2" placeholder="精灵2" class="ios-input" />
            </div>
          </div>
        </section>

        <section class="ios-card people-section">
          <div class="section-header">
            <h2 class="section-title">👥 人物列表</h2>
            <div class="section-actions">
              <button class="ios-btn ios-btn-primary" @click="addPerson">
                <span class="btn-icon">＋</span> 添加
              </button>
              <button class="ios-btn ios-btn-secondary" @click="loadSampleData">
                示例数据
              </button>
            </div>
          </div>

          <div v-if="people.length === 0" class="empty-state">
            <div class="empty-icon">👤</div>
            <p class="empty-text">还没有添加人物</p>
            <p class="empty-sub">点击「添加」或「示例数据」开始</p>
          </div>

          <TransitionGroup name="ios-list" tag="div" class="person-list">
            <div
              v-for="(person, index) in people"
              :key="person.id"
              class="person-row"
            >
              <div class="person-fields">
                <div class="person-field name-field">
                  <label class="field-label">姓名</label>
                  <input v-model="person.name" placeholder="输入姓名" class="ios-input" />
                </div>
                <div class="person-field elf-field">
                  <label class="field-label">需求精灵</label>
                  <div class="elf-radio-group">
                    <button
                      class="elf-radio-btn"
                      :class="{ active: person.needElf === 'elf1' }"
                      @click="person.needElf = 'elf1'"
                    >{{ elfName1 || '精灵1' }}</button>
                    <button
                      class="elf-radio-btn"
                      :class="{ active: person.needElf === 'elf2' }"
                      @click="person.needElf = 'elf2'"
                    >{{ elfName2 || '精灵2' }}</button>
                    <button
                      class="elf-radio-btn"
                      :class="{ active: person.needElf === 'any' }"
                      @click="person.needElf = 'any'"
                    >都行</button>
                  </div>
                </div>
                <div class="person-field head-field">
                  <label class="field-label">车头</label>
                  <button
                    class="ios-toggle"
                    :class="{ active: person.isHead }"
                    @click="onHeadToggle(person)"
                  >
                    <span class="toggle-knob"></span>
                  </button>
                </div>
                <div class="person-field tail-field">
                  <label class="field-label">车尾</label>
                  <button
                    class="ios-toggle"
                    :class="{ active: person.isTail }"
                    @click="onTailToggle(person)"
                  >
                    <span class="toggle-knob"></span>
                  </button>
                </div>
              </div>
              <button class="person-delete" @click="removePerson(index)">✕</button>
            </div>
          </TransitionGroup>
        </section>

        <section v-if="people.length >= 2" class="ios-card friend-section">
          <div class="section-header">
            <h2 class="section-title">🤝 好友关系</h2>
            <button class="ios-btn ios-btn-secondary ios-btn-sm" @click="setAllFriends">
              全选
            </button>
          </div>
          <p class="section-hint">勾选互为好友的组合，生成后会提示需加好友的对</p>
          <div class="friend-grid">
            <label
              v-for="pair in friendPairs"
              :key="pair.key"
              class="friend-chip"
              :class="{ active: isFriendPair(pair.idA, pair.idB) }"
            >
              <input
                type="checkbox"
                :checked="isFriendPair(pair.idA, pair.idB)"
                @change="(e) => toggleFriend(pair.idA, pair.idB, e.target.checked)"
                class="friend-checkbox"
              />
              <span class="friend-chip-text">{{ pair.nameA }} ↔ {{ pair.nameB }}</span>
            </label>
          </div>
        </section>

        <div class="action-bar">
          <button
            class="ios-btn ios-btn-large ios-btn-primary"
            :disabled="people.length < 2"
            @click="doGenerate"
          >
            生成传火方案
          </button>
          <button class="ios-btn ios-btn-large ios-btn-ghost" @click="resetAll">
            重置
          </button>
        </div>

        <Transition name="ios-alert">
          <div v-if="errorMsg" class="ios-alert ios-alert-error">
            <div class="alert-content">
              <span class="alert-icon">⚠️</span>
              <span class="alert-text">{{ errorMsg }}</span>
            </div>
            <button class="alert-close" @click="errorMsg = ''">✕</button>
          </div>
        </Transition>

        <TransitionGroup name="ios-card-anim" tag="div" class="result-section">
          <template v-if="planResult && planResult.success">

            <div key="export-bar" class="export-bar">
              <button class="ios-btn ios-btn-primary export-btn" :disabled="exporting" @click="exportAllCards">
                <span v-if="exporting" class="export-spinner"></span>
                {{ exporting ? '正在生成...' : '导出图片' }}
              </button>
            </div>

            <div ref="exportContainer" key="export-container" class="export-container">
              <div class="export-header">
                <div class="export-title">🔥 洛克王国通行证拼团方案</div>
                <div class="export-tier-badge">{{ tier === 'normal' ? '普通版' : '豪华版' }}</div>
              </div>

            <section key="summary" class="ios-card summary-section">
              <div class="section-header">
                <h2 class="section-title">📊 费用总览</h2>
              </div>
              <div class="stat-grid">
                <div class="stat-item">
                  <div class="stat-value">{{ planResult.chain.length }}<span class="stat-unit">人</span></div>
                  <div class="stat-label">参与人数</div>
                </div>
                <div class="stat-item">
                  <div class="stat-value">{{ planResult.totalGamePayment }}<span class="stat-unit">元</span></div>
                  <div class="stat-label">游戏总支付</div>
                </div>
                <div class="stat-item stat-highlight">
                  <div class="stat-value">{{ planResult.resultCards[0].perPerson }}<span class="stat-unit">元</span></div>
                  <div class="stat-label">每人均摊</div>
                </div>
                <div class="stat-item">
                  <div class="stat-value stat-green">{{ planResult.savings }}<span class="stat-unit">元</span></div>
                  <div class="stat-label">节省总额</div>
                </div>
              </div>
            </section>

            <section key="chain" class="ios-card chain-section">
              <div class="section-header">
                <h2 class="section-title">🔗 传火链条</h2>
              </div>
              <div class="chain-flow">
                <template v-for="(item, idx) in planResult.chainWithElf" :key="item.person.id">
                  <div class="chain-node">
                    <div class="chain-avatar" :class="idx === 0 ? 'head' : idx === planResult.chainWithElf.length - 1 ? 'tail' : 'mid'">
                      {{ item.person.name.charAt(0) }}
                    </div>
                    <div class="chain-name">{{ item.person.name }}</div>
                    <div class="chain-elf">{{ getElfName(item.assignedElf) }}</div>
                  </div>
                  <div v-if="idx < planResult.chainWithElf.length - 1" class="chain-arrow">
                    <svg width="24" height="24" viewBox="0 0 24 24" fill="none">
                      <path d="M5 12H19M19 12L13 6M19 12L13 18" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                    </svg>
                  </div>
                </template>
              </div>
            </section>

            <div
              v-if="planResult.friendWarnings && planResult.friendWarnings.length > 0"
              key="friend-warn"
              class="ios-alert ios-alert-warning"
            >
              <div class="alert-content">
                <span class="alert-icon">🤝</span>
                <div class="alert-detail">
                  <span class="alert-text">以下人员需先加好友才能赠送副券</span>
                  <div class="alert-tags">
                    <span v-for="(w, idx) in planResult.friendWarnings" :key="idx" class="alert-tag">
                      {{ w.from }} ↔ {{ w.to }}
                    </span>
                  </div>
                </div>
              </div>
            </div>

            <section
              v-for="card in planResult.resultCards"
              :key="'card-' + card.person.id"
              class="ios-card result-card"
              :class="{
                'card-head': card.role === '源头',
                'card-tail': card.role === '车尾',
              }"
            >
              <div class="result-header">
                <div class="result-identity">
                  <div class="result-avatar" :class="card.role === '源头' ? 'head' : card.role === '车尾' ? 'tail' : 'mid'">
                    {{ card.person.name.charAt(0) }}
                  </div>
                  <div>
                    <div class="result-name">{{ card.person.name }}</div>
                    <div class="result-elf-badge">{{ card.myElfName }}</div>
                  </div>
                </div>
                <span class="role-badge" :class="card.role">{{ card.role }}</span>
              </div>

              <div class="ios-divider"></div>

              <div class="cost-list">
                <div
                  v-for="(item, idx) in card.items"
                  :key="idx"
                  class="cost-row"
                  :class="item.type"
                >
                  <span class="cost-label">{{ item.label }}</span>
                  <span v-if="item.type === 'expense'" class="cost-value">+{{ item.amount }} 元</span>
                  <span v-else-if="item.type === 'info-tag'" class="cost-value info">副券激活</span>
                </div>
                <div class="cost-row subtotal">
                  <span class="cost-label">对游戏实际支付</span>
                  <span class="cost-value">{{ card.gamePayment }} 元</span>
                </div>
              </div>

              <div class="net-row">
                <span class="net-label">最终均摊支出</span>
                <span class="net-value">{{ card.netExpense }} 元</span>
              </div>

              <div class="ios-divider"></div>

              <div class="transfer-section">
                <div class="transfer-title">转账指令</div>
                <div v-if="card.transfers.length === 0" class="transfer-none">
                  无需额外转账
                </div>
                <div
                  v-for="(t, idx) in card.transfers"
                  :key="idx"
                  class="transfer-item"
                  :class="t.direction"
                >
                  <span class="transfer-icon">{{ t.direction === 'out' ? '↗️' : '↙️' }}</span>
                  <div class="transfer-info">
                    <span class="transfer-action">
                      {{ t.direction === 'out' ? `转给「${t.to}」` : `「${t.from}」转给你` }}
                    </span>
                    <span class="transfer-amount">{{ t.amount }} 元</span>
                  </div>
                </div>
              </div>

              <div class="ios-divider"></div>

              <div class="friend-status">
                <div class="friend-status-title">好友状态</div>
                <div class="friend-status-list">
                  <div
                    v-for="(h, idx) in card.friendHints"
                    :key="idx"
                    class="friend-status-item"
                    :class="h.isFriend ? 'ok' : 'need'"
                  >
                    <span class="friend-status-icon">{{ h.isFriend ? '✅' : '⚠️' }}</span>
                    <span>{{ h.type === 'prev' ? '上家' : '下家' }}：{{ h.name }}</span>
                    <span class="friend-status-label">{{ h.isFriend ? '已是好友' : '需加好友' }}</span>
                  </div>
                </div>
              </div>
            </section>

            </div>

          </template>
        </TransitionGroup>

      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, nextTick } from 'vue'
import { generatePlan, PRICE } from './utils/calculator.js'
import html2canvas from 'html2canvas'

const tier = ref('normal')
const elfName1 = ref('精灵1')
const elfName2 = ref('精灵2')
const isDark = ref(false)

let nextId = 1
const people = reactive([])
const friendships = reactive(new Map())
const planResult = ref(null)
const errorMsg = ref('')
const exporting = ref(false)
const exportContainer = ref(null)

if (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches) {
  isDark.value = true
}
window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', (e) => {
  isDark.value = e.matches
})

const friendPairs = computed(() => {
  const pairs = []
  for (let i = 0; i < people.length; i++) {
    for (let j = i + 1; j < people.length; j++) {
      const a = people[i]
      const b = people[j]
      pairs.push({
        key: `${a.id}-${b.id}`,
        idA: a.id,
        idB: b.id,
        nameA: a.name || `人物${i + 1}`,
        nameB: b.name || `人物${j + 1}`,
      })
    }
  }
  return pairs
})

function getFriendKey(idA, idB) {
  return idA < idB ? `${idA}-${idB}` : `${idB}-${idA}`
}

function isFriendPair(idA, idB) {
  return !!friendships.get(getFriendKey(idA, idB))
}

function toggleFriend(idA, idB, val) {
  const key = getFriendKey(idA, idB)
  if (val) {
    friendships.set(key, true)
  } else {
    friendships.delete(key)
  }
}

function setAllFriends() {
  for (let i = 0; i < people.length; i++) {
    for (let j = i + 1; j < people.length; j++) {
      friendships.set(getFriendKey(people[i].id, people[j].id), true)
    }
  }
}

function addPerson() {
  people.push({ id: nextId++, name: '', needElf: 'elf1', isHead: false, isTail: false })
}

function removePerson(index) {
  const removed = people.splice(index, 1)[0]
  const keysToDelete = []
  for (const [key] of friendships) {
    if (key.includes(`${removed.id}-`) || key.includes(`-${removed.id}`)) {
      keysToDelete.push(key)
    }
  }
  keysToDelete.forEach((k) => friendships.delete(k))
}

function onHeadToggle(person) {
  person.isHead = !person.isHead
  if (person.isHead) {
    people.forEach((p) => {
      if (p.id !== person.id) p.isHead = false
    })
  }
}

function onTailToggle(person) {
  person.isTail = !person.isTail
  if (person.isTail) {
    people.forEach((p) => {
      if (p.id !== person.id) p.isTail = false
    })
  }
}

function getElfName(elf) {
  if (elf === 'elf1') return elfName1.value || '精灵1'
  if (elf === 'elf2') return elfName2.value || '精灵2'
  return '都行'
}

function loadSampleData() {
  people.length = 0
  friendships.clear()
  nextId = 1
  const sample = [
    { name: '小明', needElf: 'elf1', isHead: false, isTail: false },
    { name: '小红', needElf: 'any', isHead: false, isTail: false },
    { name: '小刚', needElf: 'elf1', isHead: false, isTail: true },
  ]
  sample.forEach((p) => { people.push({ id: nextId++, ...p }) })
  friendships.set(getFriendKey(1, 2), true)
  elfName1.value = '圣光独角兽'
  elfName2.value = '暗影猎手'
  planResult.value = null
  errorMsg.value = ''
}

function resetAll() {
  people.length = 0
  friendships.clear()
  nextId = 1
  tier.value = 'normal'
  elfName1.value = '精灵1'
  elfName2.value = '精灵2'
  planResult.value = null
  errorMsg.value = ''
}

function buildFriendMatrix() {
  const matrix = []
  for (const [key] of friendships) {
    const [idA, idB] = key.split('-').map(Number)
    matrix.push([idA, idB])
  }
  return matrix
}

function doGenerate() {
  errorMsg.value = ''
  planResult.value = null
  if (people.length < 2) { errorMsg.value = '至少需要 2 个人才能生成传火方案'; return }
  if (people.some((p) => !p.name.trim())) { errorMsg.value = '请为所有人填写姓名'; return }
  if (people.some((p) => !p.needElf)) { errorMsg.value = '请为所有人选择需求精灵版本'; return }
  const result = generatePlan(
    [...people], tier.value,
    { elf1: elfName1.value || '精灵1', elf2: elfName2.value || '精灵2' },
    buildFriendMatrix()
  )
  if (!result.success) { errorMsg.value = result.error; return }
  planResult.value = result
}

async function exportAllCards() {
  if (!exportContainer.value || exporting.value) return
  exporting.value = true
  try {
    await nextTick()
    const el = exportContainer.value
    const canvas = await html2canvas(el, {
      scale: 2,
      backgroundColor: null,
      useCORS: true,
      logging: false,
      onclone: (doc) => {
        const cloned = doc.querySelector('.export-container')
        if (cloned) {
          cloned.style.padding = '24px 20px'
          cloned.style.background = isDark.value
            ? 'linear-gradient(180deg, #1C1C1E 0%, #000000 100%)'
            : 'linear-gradient(180deg, #F2F2F7 0%, #E5E5EA 100%)'
          cloned.style.borderRadius = '0'
          cloned.style.width = el.offsetWidth + 'px'
        }
      },
    })
    const link = document.createElement('a')
    const names = planResult.value.chain.map((p) => p.name).join('-')
    link.download = `传火方案_${names}.png`
    link.href = canvas.toDataURL('image/png')
    link.click()
  } catch (e) {
    console.error('导出失败:', e)
    errorMsg.value = '导出图片失败，请重试'
  } finally {
    exporting.value = false
  }
}
</script>

<style>
*, *::before, *::after {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  --ios-bg: #F2F2F7;
  --ios-bg-gradient: linear-gradient(180deg, #F2F2F7 0%, #E5E5EA 100%);
  --ios-card: rgba(255, 255, 255, 0.72);
  --ios-card-solid: #FFFFFF;
  --ios-card-border: rgba(60, 60, 67, 0.1);
  --ios-text: #1C1C1E;
  --ios-text-secondary: #8E8E93;
  --ios-text-tertiary: #AEAEB2;
  --ios-separator: rgba(60, 60, 67, 0.12);
  --ios-blue: #007AFF;
  --ios-green: #34C759;
  --ios-orange: #FF9500;
  --ios-red: #FF3B30;
  --ios-purple: #AF52DE;
  --ios-fill: rgba(120, 120, 128, 0.08);
  --ios-fill-secondary: rgba(120, 120, 128, 0.12);
  --ios-shadow: 0 2px 12px rgba(0, 0, 0, 0.04), 0 0 1px rgba(0, 0, 0, 0.06);
  --ios-shadow-elevated: 0 4px 24px rgba(0, 0, 0, 0.06), 0 0 1px rgba(0, 0, 0, 0.08);
  --ios-radius: 16px;
  --ios-radius-sm: 12px;
  --ios-radius-xs: 8px;
  --ios-font: -apple-system, BlinkMacSystemFont, 'SF Pro Display', 'SF Pro Text', 'Helvetica Neue', 'PingFang SC', 'Microsoft YaHei', sans-serif;
  --ios-spring: cubic-bezier(0.175, 0.885, 0.32, 1.275);
  --ios-ease: cubic-bezier(0.25, 0.46, 0.45, 0.94);
  --ios-ease-out: cubic-bezier(0.16, 1, 0.3, 1);
}

.dark-mode {
  --ios-bg: #1C1C1E;
  --ios-bg-gradient: linear-gradient(180deg, #1C1C1E 0%, #000000 100%);
  --ios-card: rgba(44, 44, 46, 0.78);
  --ios-card-solid: #2C2C2E;
  --ios-card-border: rgba(255, 255, 255, 0.08);
  --ios-text: #F5F5F7;
  --ios-text-secondary: #98989D;
  --ios-text-tertiary: #636366;
  --ios-separator: rgba(84, 84, 88, 0.36);
  --ios-blue: #0A84FF;
  --ios-green: #30D158;
  --ios-orange: #FF9F0A;
  --ios-red: #FF453A;
  --ios-purple: #BF5AF2;
  --ios-fill: rgba(120, 120, 128, 0.2);
  --ios-fill-secondary: rgba(120, 120, 128, 0.28);
  --ios-shadow: 0 2px 16px rgba(0, 0, 0, 0.2), 0 0 1px rgba(0, 0, 0, 0.24);
  --ios-shadow-elevated: 0 8px 32px rgba(0, 0, 0, 0.32), 0 0 1px rgba(0, 0, 0, 0.28);
}

html {
  -webkit-text-size-adjust: 100%;
  -webkit-tap-highlight-color: transparent;
}

body {
  background: #000;
  min-height: 100vh;
  font-family: var(--ios-font);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-rendering: optimizeLegibility;
}

#app {
  min-height: 100vh;
}
</style>

<style scoped>
.app-container {
  min-height: 100vh;
  background: var(--ios-bg-gradient);
  position: relative;
  overflow: hidden;
}

.app-container::before {
  content: '';
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: url('/bg.png') no-repeat center center;
  background-size: cover;
  z-index: 0;
  opacity: 0.35;
  transition: opacity 0.6s var(--ios-ease);
}

.dark-mode::before {
  opacity: 0.2;
}

.app-content {
  position: relative;
  z-index: 1;
  max-width: 680px;
  margin: 0 auto;
  padding: 20px 16px 48px;
}

.ios-card {
  background: var(--ios-card);
  backdrop-filter: blur(20px) saturate(180%);
  -webkit-backdrop-filter: blur(20px) saturate(180%);
  border-radius: var(--ios-radius);
  border: 1px solid var(--ios-card-border);
  box-shadow: var(--ios-shadow);
  padding: 20px;
  margin-bottom: 16px;
  transition: all 0.4s var(--ios-ease);
  will-change: transform, opacity;
}

.ios-card:hover {
  box-shadow: var(--ios-shadow-elevated);
}

.section-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 16px;
}

.section-title {
  font-size: 20px;
  font-weight: 700;
  color: var(--ios-text);
  letter-spacing: -0.4px;
}

.section-actions {
  display: flex;
  gap: 8px;
}

.section-hint {
  font-size: 13px;
  color: var(--ios-text-secondary);
  margin-bottom: 14px;
  line-height: 1.5;
}

.segment-control {
  display: flex;
  gap: 10px;
  margin-bottom: 18px;
}

.segment-btn {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  padding: 14px 12px;
  border-radius: var(--ios-radius-sm);
  border: 1px solid var(--ios-card-border);
  background: var(--ios-fill);
  color: var(--ios-text-secondary);
  cursor: pointer;
  transition: all 0.35s var(--ios-spring);
  font-family: var(--ios-font);
  will-change: transform;
}

.segment-btn:active {
  transform: scale(0.97);
}

.segment-btn.active {
  background: var(--ios-blue);
  border-color: var(--ios-blue);
  color: #fff;
  box-shadow: 0 4px 16px rgba(0, 122, 255, 0.28);
  transform: scale(1.02);
}

.segment-icon {
  font-size: 22px;
}

.segment-text {
  font-size: 15px;
  font-weight: 600;
}

.segment-price {
  font-size: 12px;
  opacity: 0.7;
}

.elf-name-row {
  display: flex;
  gap: 12px;
}

.elf-name-field {
  flex: 1;
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
  padding: 10px 14px;
  border-radius: var(--ios-radius-xs);
  border: 1px solid var(--ios-card-border);
  background: var(--ios-fill);
  color: var(--ios-text);
  font-size: 15px;
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
  gap: 4px;
  padding: 8px 16px;
  border-radius: 20px;
  border: none;
  font-size: 14px;
  font-weight: 600;
  font-family: var(--ios-font);
  cursor: pointer;
  transition: all 0.3s var(--ios-spring);
  will-change: transform;
}

.ios-btn:active {
  transform: scale(0.95);
}

.ios-btn-primary {
  background: var(--ios-blue);
  color: #fff;
}

.ios-btn-primary:hover {
  box-shadow: 0 4px 16px rgba(0, 122, 255, 0.28);
}

.ios-btn-secondary {
  background: var(--ios-fill-secondary);
  color: var(--ios-blue);
}

.ios-btn-sm {
  padding: 6px 12px;
  font-size: 13px;
}

.ios-btn-large {
  padding: 14px 28px;
  font-size: 17px;
  border-radius: var(--ios-radius);
}

.ios-btn-ghost {
  background: var(--ios-fill);
  color: var(--ios-text-secondary);
}

.btn-icon {
  font-size: 16px;
  font-weight: 400;
}

.empty-state {
  text-align: center;
  padding: 32px 0;
}

.empty-icon {
  font-size: 48px;
  margin-bottom: 8px;
}

.empty-text {
  font-size: 17px;
  font-weight: 600;
  color: var(--ios-text);
  margin-bottom: 4px;
}

.empty-sub {
  font-size: 14px;
  color: var(--ios-text-secondary);
}

.person-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.person-row {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  padding: 14px;
  background: var(--ios-fill);
  border-radius: var(--ios-radius-sm);
  border: 1px solid var(--ios-card-border);
  transition: all 0.35s var(--ios-ease);
}

.person-row:hover {
  background: var(--ios-fill-secondary);
}

.person-fields {
  flex: 1;
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  align-items: flex-end;
}

.person-field {
  min-width: 0;
}

.name-field {
  width: 120px;
  flex-shrink: 0;
}

.elf-field {
  flex: 1;
  min-width: 200px;
}

.head-field {
  width: auto;
  flex-shrink: 0;
}

.tail-field {
  width: auto;
  flex-shrink: 0;
}

.field-label {
  display: block;
  font-size: 12px;
  font-weight: 500;
  color: var(--ios-text-secondary);
  margin-bottom: 6px;
}

.elf-radio-group {
  display: flex;
  border-radius: var(--ios-radius-xs);
  overflow: hidden;
  border: 1px solid var(--ios-card-border);
}

.elf-radio-btn {
  padding: 7px 12px;
  font-size: 13px;
  font-weight: 500;
  font-family: var(--ios-font);
  border: none;
  background: var(--ios-fill);
  color: var(--ios-text-secondary);
  cursor: pointer;
  transition: all 0.25s var(--ios-ease);
  white-space: nowrap;
}

.elf-radio-btn + .elf-radio-btn {
  border-left: 1px solid var(--ios-card-border);
}

.elf-radio-btn.active {
  background: var(--ios-blue);
  color: #fff;
}

.ios-toggle {
  width: 51px;
  height: 31px;
  border-radius: 16px;
  border: none;
  background: var(--ios-fill-secondary);
  position: relative;
  cursor: pointer;
  transition: background 0.35s var(--ios-spring);
  margin-top: 2px;
}

.ios-toggle.active {
  background: var(--ios-green);
}

.toggle-knob {
  position: absolute;
  top: 2px;
  left: 2px;
  width: 27px;
  height: 27px;
  border-radius: 50%;
  background: #fff;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.12), 0 0 1px rgba(0, 0, 0, 0.08);
  transition: transform 0.35s var(--ios-spring);
}

.ios-toggle.active .toggle-knob {
  transform: translateX(20px);
}

.person-delete {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  border: none;
  background: var(--ios-fill-secondary);
  color: var(--ios-red);
  font-size: 13px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  margin-top: 20px;
  transition: all 0.25s var(--ios-spring);
}

.person-delete:hover {
  background: var(--ios-red);
  color: #fff;
}

.person-delete:active {
  transform: scale(0.88);
}

.friend-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.friend-chip {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 8px 14px;
  border-radius: 20px;
  background: var(--ios-fill);
  border: 1px solid var(--ios-card-border);
  cursor: pointer;
  transition: all 0.3s var(--ios-spring);
  user-select: none;
}

.friend-chip.active {
  background: rgba(0, 122, 255, 0.1);
  border-color: rgba(0, 122, 255, 0.3);
}

.friend-checkbox {
  display: none;
}

.friend-chip-text {
  font-size: 14px;
  font-weight: 500;
  color: var(--ios-text);
}

.friend-chip.active .friend-chip-text {
  color: var(--ios-blue);
}

.action-bar {
  display: flex;
  gap: 12px;
  justify-content: center;
  margin: 24px 0;
}

.action-bar .ios-btn-large:disabled {
  opacity: 0.4;
  cursor: not-allowed;
  transform: none;
}

.ios-alert {
  padding: 14px 16px;
  border-radius: var(--ios-radius-sm);
  margin-bottom: 16px;
  display: flex;
  align-items: flex-start;
  gap: 10px;
  backdrop-filter: blur(20px) saturate(180%);
  -webkit-backdrop-filter: blur(20px) saturate(180%);
}

.ios-alert-error {
  background: rgba(255, 59, 48, 0.08);
  border: 1px solid rgba(255, 59, 48, 0.15);
}

.ios-alert-warning {
  background: rgba(255, 149, 0, 0.08);
  border: 1px solid rgba(255, 149, 0, 0.15);
}

.alert-content {
  display: flex;
  align-items: flex-start;
  gap: 10px;
  flex: 1;
}

.alert-icon {
  font-size: 18px;
  flex-shrink: 0;
}

.alert-text {
  font-size: 14px;
  font-weight: 500;
  color: var(--ios-text);
  line-height: 1.5;
}

.alert-detail {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.alert-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}

.alert-tag {
  display: inline-block;
  padding: 4px 10px;
  border-radius: 6px;
  background: rgba(255, 149, 0, 0.12);
  font-size: 13px;
  font-weight: 500;
  color: var(--ios-orange);
}

.alert-close {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  border: none;
  background: var(--ios-fill-secondary);
  color: var(--ios-text-secondary);
  font-size: 12px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  transition: all 0.2s var(--ios-ease);
}

.alert-close:active {
  transform: scale(0.9);
}

.stat-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
}

.stat-item {
  text-align: center;
  padding: 14px 8px;
  border-radius: var(--ios-radius-sm);
  background: var(--ios-fill);
  border: 1px solid var(--ios-card-border);
}

.stat-highlight {
  background: rgba(0, 122, 255, 0.06);
  border-color: rgba(0, 122, 255, 0.12);
}

.stat-value {
  font-size: 28px;
  font-weight: 700;
  color: var(--ios-text);
  letter-spacing: -0.5px;
}

.stat-unit {
  font-size: 14px;
  font-weight: 400;
  color: var(--ios-text-secondary);
  margin-left: 2px;
}

.stat-highlight .stat-value {
  color: var(--ios-blue);
}

.stat-green {
  color: var(--ios-green) !important;
}

.stat-label {
  font-size: 12px;
  font-weight: 500;
  color: var(--ios-text-secondary);
  margin-top: 4px;
}

.chain-flow {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-wrap: wrap;
  gap: 8px;
  padding: 8px 0;
}

.chain-node {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  transition: transform 0.35s var(--ios-spring);
}

.chain-node:hover {
  transform: translateY(-2px);
}

.chain-avatar {
  width: 44px;
  height: 44px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 18px;
  font-weight: 700;
  color: #fff;
  transition: transform 0.35s var(--ios-spring);
}

.chain-avatar.head {
  background: linear-gradient(135deg, #FF6B6B, #FF3B30);
}

.chain-avatar.mid {
  background: linear-gradient(135deg, #5AC8FA, #007AFF);
}

.chain-avatar.tail {
  background: linear-gradient(135deg, #63E6BE, #34C759);
}

.chain-node:hover .chain-avatar {
  transform: scale(1.1);
}

.chain-name {
  font-size: 13px;
  font-weight: 600;
  color: var(--ios-text);
}

.chain-elf {
  font-size: 11px;
  color: var(--ios-text-secondary);
}

.chain-arrow {
  color: var(--ios-text-tertiary);
  display: flex;
  align-items: center;
  padding-top: 4px;
}

.chain-arrow svg {
  width: 20px;
  height: 20px;
}

.result-card {
  overflow: hidden;
}

.result-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.result-identity {
  display: flex;
  align-items: center;
  gap: 12px;
}

.result-avatar {
  width: 44px;
  height: 44px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 18px;
  font-weight: 700;
  color: #fff;
  flex-shrink: 0;
}

.result-avatar.head { background: linear-gradient(135deg, #FF6B6B, #FF3B30); }
.result-avatar.mid { background: linear-gradient(135deg, #5AC8FA, #007AFF); }
.result-avatar.tail { background: linear-gradient(135deg, #63E6BE, #34C759); }

.result-name {
  font-size: 18px;
  font-weight: 700;
  color: var(--ios-text);
}

.result-elf-badge {
  font-size: 13px;
  color: var(--ios-text-secondary);
  margin-top: 2px;
}

.role-badge {
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 13px;
  font-weight: 600;
}

.role-badge.源头 {
  background: rgba(255, 59, 48, 0.1);
  color: var(--ios-red);
}

.role-badge.中间人 {
  background: rgba(0, 122, 255, 0.1);
  color: var(--ios-blue);
}

.role-badge.车尾 {
  background: rgba(52, 199, 89, 0.1);
  color: var(--ios-green);
}

.ios-divider {
  height: 1px;
  background: var(--ios-separator);
  margin: 14px 0;
}

.cost-list {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.cost-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 12px;
  border-radius: var(--ios-radius-xs);
  background: var(--ios-fill);
}

.cost-row.info-tag {
  background: rgba(0, 122, 255, 0.05);
}

.cost-row.subtotal {
  background: rgba(255, 149, 0, 0.05);
  border: 1px solid rgba(255, 149, 0, 0.12);
  margin-top: 4px;
}

.cost-label {
  font-size: 14px;
  color: var(--ios-text);
}

.cost-value {
  font-size: 15px;
  font-weight: 600;
  color: var(--ios-red);
}

.cost-value.info {
  font-size: 13px;
  font-weight: 400;
  color: var(--ios-blue);
}

.net-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 14px 16px;
  border-radius: var(--ios-radius-sm);
  background: linear-gradient(135deg, var(--ios-blue), var(--ios-purple));
  color: #fff;
  margin-top: 4px;
}

.net-label {
  font-size: 16px;
  font-weight: 600;
}

.net-value {
  font-size: 24px;
  font-weight: 800;
  letter-spacing: -0.5px;
}

.transfer-section {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.transfer-title,
.friend-status-title {
  font-size: 14px;
  font-weight: 600;
  color: var(--ios-text-secondary);
  margin-bottom: 4px;
}

.transfer-none {
  font-size: 14px;
  color: var(--ios-text-tertiary);
  padding: 8px 0;
}

.transfer-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 12px;
  border-radius: var(--ios-radius-xs);
  background: var(--ios-fill);
}

.transfer-item.out {
  border-left: 3px solid var(--ios-orange);
}

.transfer-item.in {
  border-left: 3px solid var(--ios-green);
}

.transfer-icon {
  font-size: 18px;
}

.transfer-info {
  flex: 1;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.transfer-action {
  font-size: 14px;
  color: var(--ios-text);
  font-weight: 500;
}

.transfer-amount {
  font-size: 16px;
  font-weight: 700;
  color: var(--ios-text);
}

.friend-status-list {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.friend-status-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  border-radius: var(--ios-radius-xs);
  font-size: 14px;
  color: var(--ios-text);
  background: var(--ios-fill);
}

.friend-status-item.ok {
  background: rgba(52, 199, 89, 0.05);
}

.friend-status-item.need {
  background: rgba(255, 149, 0, 0.05);
}

.friend-status-icon {
  font-size: 16px;
}

.friend-status-label {
  margin-left: auto;
  font-size: 13px;
  font-weight: 500;
}

.friend-status-item.ok .friend-status-label {
  color: var(--ios-green);
}

.friend-status-item.need .friend-status-label {
  color: var(--ios-orange);
}

.ios-list-enter-active {
  transition: all 0.45s var(--ios-spring);
}
.ios-list-leave-active {
  transition: all 0.3s var(--ios-ease);
}
.ios-list-enter-from {
  opacity: 0;
  transform: translateY(-16px) scale(0.95);
}
.ios-list-leave-to {
  opacity: 0;
  transform: translateX(40px) scale(0.95);
}

.ios-alert-enter-active {
  transition: all 0.45s var(--ios-spring);
}
.ios-alert-leave-active {
  transition: all 0.25s var(--ios-ease);
}
.ios-alert-enter-from,
.ios-alert-leave-to {
  opacity: 0;
  transform: translateY(-10px) scale(0.95);
}

.ios-card-anim-enter-active {
  transition: all 0.55s var(--ios-spring);
}
.ios-card-anim-leave-active {
  transition: all 0.3s var(--ios-ease);
}
.ios-card-anim-enter-from {
  opacity: 0;
  transform: translateY(24px) scale(0.96);
}
.ios-card-anim-leave-to {
  opacity: 0;
  transform: translateY(-12px) scale(0.98);
}

.export-bar {
  display: flex;
  justify-content: center;
  margin-bottom: 16px;
}

.export-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 12px 24px;
  border-radius: var(--ios-radius);
  font-size: 16px;
  font-weight: 600;
}

.export-spinner {
  display: inline-block;
  width: 18px;
  height: 18px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-top-color: #fff;
  border-radius: 50%;
  animation: spin 0.6s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.export-icon-wrap {
  font-size: 18px;
}

.export-container {
  border-radius: var(--ios-radius);
  overflow: hidden;
}

.export-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 20px;
  background: linear-gradient(135deg, var(--ios-blue), var(--ios-purple));
  color: #fff;
}

.export-title {
  font-size: 18px;
  font-weight: 700;
  letter-spacing: -0.3px;
}

.export-tier-badge {
  padding: 4px 12px;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.2);
  font-size: 13px;
  font-weight: 600;
}

@media (max-width: 640px) {
  .app-content {
    padding: 12px 12px 40px;
  }

  .ios-card {
    padding: 16px;
    border-radius: 14px;
  }

  .person-fields {
    flex-direction: column;
  }

  .name-field,
  .elf-field {
    width: 100%;
    min-width: 0;
  }

  .segment-btn {
    padding: 12px 8px;
  }

  .stat-grid {
    grid-template-columns: repeat(2, 1fr);
    gap: 8px;
  }

  .stat-value {
    font-size: 22px;
  }

  .net-value {
    font-size: 20px;
  }

  .chain-flow {
    gap: 4px;
  }

  .chain-avatar {
    width: 36px;
    height: 36px;
    font-size: 15px;
  }
}
</style>
