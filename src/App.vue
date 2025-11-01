<template>
  <div :class="['app', isNight ? 'night' : 'day']">
    <header class="hero" role="banner">
      <div class="sky" aria-hidden="true">
        <div class="gradient"></div>
        <div class="moon">
          <div class="crater c1"></div>
          <div class="crater c2"></div>
          <div class="crater c3"></div>
        </div>
        <div class="stars"></div>
        <div class="cloud c1"></div>
        <div class="cloud c2"></div>
      </div>

      <div class="hero-content">
        <h1 class="title" aria-label="풍성한 한가위, 마음도 둥글게">
          풍성한 한가위, 마음도 둥글게
        </h1>
        <p class="subtitle">
          달빛 아래 가족과 이웃이 모여 정을 나누는 날, 추석.
          고운 마음 서로 건네며 넉넉함이 차오르는 시간을 만들어 보세요.
        </p>

        <div class="toolbar">
          <button class="btn" @click="toggleTheme" :aria-pressed="isNight">
            {{ isNight ? '낮 모드로 보기' : '달 보기(밤 모드)' }}
          </button>
        </div>

        <form class="wish-form" @submit.prevent="addWish" aria-label="달에게 소원 보내기">
          <input
              class="wish-input"
              v-model="wishInput"
              :placeholder="isNight ? '보름달에 빌 소원을 적어보세요(최대 40자)' : '소원은 달이 떴을 때 더 잘 전해져요! (밤 모드 추천)'"
              maxlength="40"
              aria-label="소원 입력"
          />
          <button class="btn accent" type="submit" :disabled="!wishInput.trim().length">
            소원 빌기
          </button>
        </form>
      </div>

      <div class="wish-layer" aria-hidden="true">
        <div
            v-for="w in wishes"
            :key="w.id"
            class="wish"
            :style="wishStyle(w)"
            role="img"
            :aria-label="'소원: ' + w.text"
        >
          <span class="lantern">🏮</span>
          <span class="wish-text">{{ w.text }}</span>
        </div>
      </div>
    </header>

    <main class="content" role="main">
      <section class="section about" aria-labelledby="about-title">
        <h2 id="about-title" class="section-title">한가위 이야기</h2>
        <p class="body">
          한가위(추석)는 음력 8월 15일, 보름달이 가장 밝게 차오르는 날입니다. 예부터
          가을 추수를 마무리하며 조상님께 감사의 마음을 전하고, 이웃과 음식을 나누며
          풍년을 기원했습니다. 가족들은 멀리 떨어져 있어도 이 날만큼은 한 자리에 모여
          도란도란 이야기를 나누고, 달빛 아래에서 전통 놀이를 즐기며 정을 쌓았습니다.
          보름달처럼 둥글고 넉넉한 마음을 다시 떠올리는 날이지요.
        </p>
      </section>

      <section class="section foods" aria-labelledby="foods-title">
        <h2 id="foods-title" class="section-title">한가위 전통 음식</h2>
        <div class="grid">
          <article v-for="f in foods" :key="f.id" class="card">
            <div class="card-emoji" aria-hidden="true">{{ f.emoji }}</div>
            <h3 class="card-title">{{ f.name }}</h3>
            <p class="card-desc">{{ f.description }}</p>
          </article>
        </div>
      </section>

      <section class="section games" aria-labelledby="games-title">
        <h2 id="games-title" class="section-title">한가위 전통 놀이</h2>
        <div class="grid">
          <article v-for="g in games" :key="g.id" class="card">
            <div class="card-emoji" aria-hidden="true">{{ g.emoji }}</div>
            <h3 class="card-title">{{ g.name }}</h3>
            <p class="card-desc">{{ g.description }}</p>
          </article>
        </div>

        <div class="interactive">
          <div class="panel">
            <h3 class="panel-title">달빛 강강술래</h3>
            <p class="panel-desc">
              손에 손을 잡고 보름달 아래 원을 그리며 춤추던 남도의 전통 놀이.
              버튼을 눌러 원형 무대를 빙글빙글 돌려보세요.
            </p>
            <div class="sulle-stage" :class="{ dancing: isDancing }" aria-hidden="true">
              <div class="dancer" v-for="n in 10" :key="n" :style="dancerStyle(n)"></div>
            </div>
            <button class="btn" @click="isDancing = !isDancing" :aria-pressed="isDancing">
              {{ isDancing ? '멈추기' : '시작하기' }}
            </button>
          </div>

          <div class="panel">
            <h3 class="panel-title">미니 윷 던지기</h3>
            <p class="panel-desc">
              윷가락 네 개의 앞뒤로 결과가 결정됩니다. 도·개·걸·윷·모 중 무엇이 나올까요?
            </p>
            <div class="sticks" :class="{ rolling: isRolling }" role="img" aria-label="윷가락">
              <div
                  v-for="(s, i) in sticks"
                  :key="i"
                  class="stick"
                  :class="s ? 'front' : 'back'"
                  :aria-label="s ? '앞면' : '뒷면'"
              >
                <div class="grain"></div>
              </div>
            </div>
            <button class="btn accent" @click="tossYut" :disabled="isRolling">
              {{ isRolling ? '던지는 중...' : '윷 던지기' }}
            </button>
            <div class="yut-result" aria-live="polite" :aria-busy="isRolling">
              {{ yutResult ? `${yutResult} — ${yutDesc}` : '버튼을 눌러 결과를 확인해 보세요!' }}
            </div>
            <div class="history" v-if="yutHistory.length">
              최근 결과:
              <span v-for="(r, idx) in yutHistory" :key="idx" class="chip">{{ r }}</span>
            </div>
          </div>
        </div>
      </section>
    </main>

    <footer class="footer" role="contentinfo">
      <p>즐거운 추석 보내세요. 보름달처럼 마음이 환해지는 밤 되시길! ✨</p>
      <small>© {{ currentYear }} 한가위 소개 • Vue 3 + TypeScript + Vite</small>
    </footer>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'

type Food = {
  id: number
  name: string
  description: string
  emoji: string
}

type Game = {
  id: number
  name: string
  description: string
  emoji: string
}

type Wish = {
  id: number
  text: string
  left: number // percent 5~95
  duration: number // seconds
  delay: number // seconds
}

const isNight = ref(true)
const isDancing = ref(false)
const currentYear = new Date().getFullYear()

// Main copy is in template

// Foods
const foods = ref<Food[]>([
  {
    id: 1,
    name: '송편',
    emoji: '🥟',
    description:
        '갓 찐 송편 속에 깨·콩·밤을 넣어 보름달처럼 빚어 먹는 한가위의 상징 같은 음식.',
  },
  {
    id: 2,
    name: '한과',
    emoji: '🍘',
    description:
        '꿀과 쌀, 콩으로 빚어 바삭하고 달콤한 전통 과자. 차와 함께 곁들이기 좋아요.',
  },
  {
    id: 3,
    name: '전',
    emoji: '🍳',
    description:
        '부추전·동그랑땡 등 각종 재료를 기름에 지진 전. 풍성한 상차림의 단골 손님.',
  },
  {
    id: 4,
    name: '식혜',
    emoji: '🫖',
    description:
        '엿기름과 밥으로 만든 전통 음료. 달콤하고 시원하게 입가심하기 좋습니다.',
  },
])

// Games
const games = ref<Game[]>([
  {
    id: 1,
    name: '강강술래',
    emoji: '🌀',
    description:
        '달빛 아래 손을 맞잡고 원을 그리며 빙글빙글 도는 남도의 전통 춤·놀이.',
  },
  {
    id: 2,
    name: '윷놀이',
    emoji: '🪵',
    description:
        '윷가락 네 개를 던져 말을 움직이는 보드 게임. 도·개·걸·윷·모로 승부를 겨룹니다.',
  },
  {
    id: 3,
    name: '줄다리기',
    emoji: '🪢',
    description:
        '서로 힘을 합쳐 줄을 당기며 겨루는 놀이. 마을의 단합과 풍년을 기원했어요.',
  },
  {
    id: 4,
    name: '씨름',
    emoji: '🤼',
    description:
        '모래판에서 힘과 재치로 맞붙는 민속 스포츠. 서로의 기량을 겨루며 흥을 돋웠습니다.',
  },
])

// Theme auto based on time
onMounted(() => {
  const h = new Date().getHours()
  isNight.value = h >= 18 || h < 6
})

// Theme toggle
function toggleTheme() {
  isNight.value = !isNight.value
}

// "강강술래" dancer positions
function dancerStyle(n: number): Record<string, string> {
  const total = 10
  const angle = ((n - 1) / total) * Math.PI * 2
  const radius = 64
  const x = Math.cos(angle) * radius
  const y = Math.sin(angle) * radius
  return {
    transform: `translate(${x}px, ${y}px)`,
  }
}

// 윷놀이
const sticks = ref<number[]>([0, 0, 0, 0]) // 0: back, 1: front
const isRolling = ref(false)
const yutResult = ref<string | null>(null)
const yutDesc = ref('')
const yutHistory = ref<string[]>([])

function tossYut() {
  if (isRolling.value) return
  isRolling.value = true
  yutResult.value = null

  // Simulate quick flip animation updates
  const flips: number[] = Array.from({ length: 4 }, () =>
      Math.random() < 0.5 ? 0 : 1
  )
  sticks.value = flips

  // Compute result with real distribution by counting fronts
  const count = flips.reduce((a, b) => a + b, 0)
  let result = ''
  let desc = ''
  switch (count) {
    case 0:
      result = '모'
      desc = '다섯 칸 전진 + 한 번 더!'
      break
    case 1:
      result = '도'
      desc = '한 칸 전진'
      break
    case 2:
      result = '개'
      desc = '두 칸 전진'
      break
    case 3:
      result = '걸'
      desc = '세 칸 전진'
      break
    case 4:
      result = '윷'
      desc = '네 칸 전진 + 한 번 더!'
      break
  }

  // Slight delay to feel the roll
  setTimeout(() => {
    yutResult.value = result
    yutDesc.value = desc
    yutHistory.value = [result, ...yutHistory.value].slice(0, 6)
    isRolling.value = false
  }, 650)
}

// Wishes
const wishes = ref<Wish[]>([])
const wishInput = ref('')

function wishStyle(w: Wish): Record<string, string> {
  return {
    left: `${w.left}%`,
    animationDuration: `${w.duration}s`,
    animationDelay: `${w.delay}s`,
  }
}

function addWish() {
  const text = wishInput.value.trim()
  if (!text) return
  const id = Date.now() + Math.round(Math.random() * 1000)
  const w: Wish = {
    id,
    text,
    left: 8 + Math.random() * 84, // 8% ~ 92%
    duration: 10 + Math.random() * 8, // 10~18s
    delay: Math.random() * 3, // 0~3s
  }
  wishes.value.push(w)
  wishInput.value = ''

  // Auto remove after animation
  const ttl = (w.duration + w.delay) * 1000 + 100
  setTimeout(() => {
    wishes.value = wishes.value.filter((x) => x.id !== w.id)
  }, ttl)
}
</script>

<style scoped>
/* Base */
.app {
  --bg-from: #0b1026;
  --bg-to: #1a2240;
  --text: #eaeefc;
  --muted: #b7c0e7;
  --card: rgba(255, 255, 255, 0.06);
  --card-border: rgba(255, 255, 255, 0.12);
  --accent: #ffcf33;
  --accent-ink: #1a1200;
  --chip: rgba(255, 255, 255, 0.16);

  color: var(--text);
  background: linear-gradient(180deg, var(--bg-from), var(--bg-to));
  min-height: 100svh;
  font-family: system-ui, -apple-system, Segoe UI, Roboto, Noto Sans KR, Apple SD Gothic Neo,
  Malgun Gothic, sans-serif;
  line-height: 1.6;
}

.app.day {
  --bg-from: #fdf4df;
  --bg-to: #f3e1c3;
  --text: #2c2c2c;
  --muted: #5e5141;
  --card: rgba(255, 255, 255, 0.82);
  --card-border: rgba(0, 0, 0, 0.06);
  --accent: #da7f1f;
  --accent-ink: #fff7ea;
  --chip: rgba(0, 0, 0, 0.08);
}

/* Hero / Sky */
.hero {
  position: relative;
  overflow: hidden;
  padding: 72px 20px 140px;
}

.sky {
  position: absolute;
  inset: 0;
  pointer-events: none;
}

.gradient {
  position: absolute;
  inset: 0;
  background: radial-gradient(1200px 500px at 80% -10%, rgba(255, 255, 255, 0.15), transparent),
  linear-gradient(180deg, var(--bg-from), var(--bg-to));
}

.moon {
  position: absolute;
  top: 40px;
  right: 40px;
  width: clamp(120px, 22vw, 220px);
  aspect-ratio: 1/1;
  border-radius: 50%;
  background: radial-gradient(circle at 30% 30%, #fff8cc, #ffd24d 55%, #eab742 75%, #0000 76%),
  #ffd24d;
  box-shadow: 0 0 80px 20px rgba(255, 213, 77, 0.35), 0 0 0 2px rgba(255, 255, 255, 0.06) inset;
  filter: drop-shadow(0 10px 30px rgba(255, 210, 80, 0.2));
  transition: transform 0.6s ease, opacity 0.6s ease;
}

.app.day .moon {
  opacity: 0.5;
  transform: translateY(-10px) scale(0.9);
}

.crater {
  position: absolute;
  background: radial-gradient(circle at 35% 35%, rgba(0, 0, 0, 0.18), rgba(0, 0, 0, 0.28) 60%, transparent 61%);
  border-radius: 50%;
  opacity: 0.25;
}
.crater.c1 {
  width: 18%;
  height: 18%;
  left: 20%;
  top: 28%;
}
.crater.c2 {
  width: 14%;
  height: 14%;
  right: 22%;
  top: 40%;
}
.crater.c3 {
  width: 10%;
  height: 10%;
  left: 45%;
  top: 60%;
}

.stars {
  position: absolute;
  inset: 0;
  background-image: radial-gradient(2px 2px at 20% 30%, rgba(255, 255, 255, 0.9), transparent),
  radial-gradient(1px 1px at 45% 10%, rgba(255, 255, 255, 0.7), transparent),
  radial-gradient(1.5px 1.5px at 70% 40%, rgba(255, 255, 255, 0.8), transparent),
  radial-gradient(1.2px 1.2px at 85% 60%, rgba(255, 255, 255, 0.6), transparent),
  radial-gradient(1px 1px at 30% 80%, rgba(255, 255, 255, 0.7), transparent);
  opacity: 0.7;
  transition: opacity 0.5s ease;
}

.app.day .stars {
  opacity: 0.05;
}

/* Clouds (visible in day) */
.cloud {
  position: absolute;
  background: #ffffff;
  filter: blur(12px);
  opacity: 0.25;
  border-radius: 60px;
}
.cloud.c1 {
  left: 10%;
  top: 20%;
  width: 240px;
  height: 80px;
}
.cloud.c2 {
  right: 15%;
  top: 35%;
  width: 200px;
  height: 70px;
}
.app.night .cloud {
  opacity: 0.07;
}

/* Hero content */
.hero-content {
  position: relative;
  z-index: 1;
  max-width: 980px;
  margin: 0 auto;
  text-align: center;
}

.title {
  font-size: clamp(28px, 6vw, 64px);
  font-weight: 800;
  letter-spacing: -0.02em;
  margin: 0 0 12px;
  text-shadow: 0 2px 20px rgba(0, 0, 0, 0.2);
}

.subtitle {
  color: var(--muted);
  font-size: clamp(14px, 2.4vw, 18px);
  margin: 0 auto 18px;
  max-width: 720px;
}

.toolbar {
  display: flex;
  justify-content: center;
  gap: 12px;
  margin: 6px 0 12px;
}

.btn {
  appearance: none;
  cursor: pointer;
  border: 1px solid var(--card-border);
  background: var(--card);
  color: var(--text);
  padding: 10px 14px;
  border-radius: 12px;
  font-weight: 600;
  transition: transform 0.06s ease, background 0.2s ease, border 0.2s ease;
  backdrop-filter: blur(6px);
}
.btn:hover {
  transform: translateY(-1px);
  border-color: rgba(255, 255, 255, 0.22);
}
.btn:active {
  transform: translateY(0);
}
.btn.accent {
  background: var(--accent);
  color: var(--accent-ink);
  border-color: color-mix(in oklab, var(--accent) 60%, #0000);
}

/* Wish input */
.wish-form {
  margin: 12px auto 0;
  display: flex;
  gap: 8px;
  justify-content: center;
  max-width: 720px;
}
.wish-input {
  flex: 1;
  min-width: 200px;
  max-width: 480px;
  background: var(--card);
  color: var(--text);
  border: 1px solid var(--card-border);
  border-radius: 12px;
  padding: 12px 14px;
  outline: none;
  transition: border 0.2s ease, background 0.2s ease;
}
.wish-input:focus {
  border-color: rgba(255, 255, 255, 0.3);
}

/* Floating wishes */
.wish-layer {
  position: absolute;
  inset: 0;
  overflow: hidden;
  pointer-events: none;
}
.wish {
  position: absolute;
  bottom: -10%;
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  background: rgba(255, 255, 255, 0.12);
  border: 1px solid rgba(255, 255, 255, 0.22);
  border-radius: 999px;
  color: #fff;
  font-weight: 600;
  filter: drop-shadow(0 8px 20px rgba(0, 0, 0, 0.25));
  animation-name: floatUp;
  animation-timing-function: ease-in;
  animation-iteration-count: 1;
}
.wish .lantern {
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.25));
}
.wish .wish-text {
  white-space: nowrap;
}

@keyframes floatUp {
  0% {
    transform: translateY(0) scale(0.95);
    opacity: 0;
  }
  10% {
    opacity: 1;
  }
  100% {
    transform: translateY(-120vh) scale(1.05);
    opacity: 0.6;
  }
}

/* Content sections */
.content {
  position: relative;
  z-index: 1;
  max-width: 1080px;
  margin: -80px auto 0;
  padding: 0 20px 60px;
}

.section {
  margin: 28px 0 32px;
}

.section-title {
  font-size: clamp(20px, 3.6vw, 28px);
  font-weight: 800;
  margin: 0 0 14px;
}

.body {
  margin: 0;
  color: var(--text);
  opacity: 0.95;
  background: var(--card);
  border: 1px solid var(--card-border);
  padding: 16px 16px;
  border-radius: 14px;
}

/* Cards grid */
.grid {
  display: grid;
  grid-template-columns: repeat(1, minmax(0, 1fr));
  gap: 12px;
}
@media (min-width: 560px) {
  .grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}
@media (min-width: 900px) {
  .grid {
    grid-template-columns: repeat(4, minmax(0, 1fr));
  }
}

.card {
  background: var(--card);
  border: 1px solid var(--card-border);
  border-radius: 16px;
  padding: 14px;
  transition: transform 0.1s ease, border 0.2s ease, background 0.2s ease;
  min-height: 152px;
  display: flex;
  flex-direction: column;
}
.card:hover {
  transform: translateY(-2px);
  border-color: rgba(255, 255, 255, 0.22);
}
.card-emoji {
  font-size: 28px;
  line-height: 1;
  margin-bottom: 8px;
}
.card-title {
  font-size: 18px;
  font-weight: 800;
  margin: 0 0 6px;
}
.card-desc {
  margin: 0;
  color: var(--muted);
  font-size: 14px;
}

/* Interactive area */
.interactive {
  display: grid;
  grid-template-columns: 1fr;
  gap: 16px;
  margin-top: 10px;
}
@media (min-width: 860px) {
  .interactive {
    grid-template-columns: 1fr 1fr;
  }
}

.panel {
  background: var(--card);
  border: 1px solid var(--card-border);
  border-radius: 16px;
  padding: 16px;
}
.panel-title {
  font-weight: 800;
  margin: 0 0 6px;
  font-size: 18px;
}
.panel-desc {
  margin: 0 0 12px;
  color: var(--muted);
  font-size: 14px;
}

/* 강강술래 무대 */
.sulle-stage {
  position: relative;
  width: 180px;
  height: 180px;
  margin: 0 auto 12px;
  border-radius: 50%;
  background: radial-gradient(circle at 50% 50%, rgba(255, 255, 255, 0.12), rgba(255, 255, 255, 0.06));
  border: 1px dashed var(--card-border);
  display: grid;
  place-items: center;
  isolation: isolate;
}
.sulle-stage::after {
  content: '';
  position: absolute;
  inset: 12px;
  border-radius: 50%;
  border: 1px dashed var(--card-border);
}
.sulle-stage.dancing {
  animation: rotate 8s linear infinite;
}
@keyframes rotate {
  to {
    transform: rotate(360deg);
  }
}
.dancer {
  position: absolute;
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: linear-gradient(180deg, #ffd84a, #ffb300);
  box-shadow: 0 2px 8px rgba(255, 213, 77, 0.55);
}

/* 윷가락 */
.sticks {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
  margin-bottom: 10px;
}
.stick {
  height: 54px;
  border-radius: 10px;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.4s ease;
  border: 1px solid var(--card-border);
  overflow: hidden;
}
.stick.front {
  background: linear-gradient(180deg, #e3c8a3, #c6a57a);
}
.stick.back {
  background: linear-gradient(180deg, #9b7b56, #83643e);
}
.stick .grain {
  position: absolute;
  inset: 0;
  background-image: radial-gradient(1px 10px at 50% 20%, rgba(0, 0, 0, 0.08), transparent),
  radial-gradient(2px 1px at 60% 60%, rgba(0, 0, 0, 0.08), transparent);
  opacity: 0.5;
}
.sticks.rolling .stick {
  animation: wiggle 0.6s ease;
}
@keyframes wiggle {
  0% {
    transform: translateY(0) rotate(0deg);
  }
  25% {
    transform: translateY(-6px) rotate(8deg);
  }
  50% {
    transform: translateY(0) rotate(-8deg);
  }
  100% {
    transform: translateY(-2px) rotate(4deg);
  }
}
.yut-result {
  font-weight: 700;
  margin-top: 6px;
}
.history {
  margin-top: 6px;
  color: var(--muted);
  font-size: 14px;
}
.chip {
  display: inline-block;
  padding: 4px 8px;
  margin-left: 6px;
  border-radius: 999px;
  background: var(--chip);
}

/* Footer */
.footer {
  padding: 24px 20px 60px;
  text-align: center;
  color: var(--muted);
}
.footer p {
  margin: 0 0 8px;
}

/* Reduce motion */
@media (prefers-reduced-motion: reduce) {
  .sulle-stage.dancing,
  .wish {
    animation: none !important;
  }
  .sticks.rolling .stick {
    animation: none;
  }
}
</style>