<script setup>
/* ===================================================================
   一日食光 · 首页组件
   从根目录的静态版（index.html + app.js + style.css）迁移而来，
   迁移时把「查 DOM + innerHTML 拼字符串」换成了 Vue 的响应式写法：
   1) 轮播图：current 记录当前第几屏，自动播放用 setInterval，
      在 onMounted 里启动、onBeforeUnmount 里清理
   2) 口味切换：activeCategory 是一个 ref，菜品列表用 computed 过滤
   =================================================================== */
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'
import heroBanner from '../assets/hero-banner.jpg'

/* -------------------- 数据：口味分类 -------------------- */
const categories = ['减脂', '清淡', '快手早餐', '香辣', '汤品', '甜品']

/* -------------------- 数据：菜品 --------------------
   字段说明：
   name   菜名
   cats   属于哪几个分类（可以挂多个）
   tags   卡片上显示的小标签（2 个正好）
   badge  右上角的时间 / 场景角标
   kcal   热量
   time   烹饪时长
   emoji  占位块里的图标
   image  图片地址，留空就显示占位块；填了就显示真图
   ---------------------------------------------------- */
const dishes = [
    {
        name: '香煎鸡胸杂粮碗', cats: ['减脂'],
        tags: ['减脂', '高蛋白'], badge: '10分钟', kcal: 380, time: '约30分钟',
        emoji: '🍗', image: ''
    },
    {
        name: '番茄鸡蛋面', cats: ['减脂', '快手早餐'],
        tags: ['减脂', '高蛋白'], badge: '早餐', kcal: 380, time: '约30分钟',
        emoji: '🍜', image: ''
    },
    {
        name: '冬瓜虾仁汤', cats: ['减脂', '汤品'],
        tags: ['减脂', '高蛋白'], badge: '早餐', kcal: 86, time: '约20分钟',
        emoji: '🍤', image: ''
    },
    {
        name: '紫薯燕麦粥', cats: ['减脂', '快手早餐'],
        tags: ['减脂', '高纤维'], badge: '早餐', kcal: 230, time: '约20分钟',
        emoji: '🍠', image: ''
    },
    {
        name: '清蒸鲈鱼', cats: ['清淡'],
        tags: ['高蛋白', '低脂'], badge: '晚餐', kcal: 210, time: '约25分钟',
        emoji: '🐟', image: ''
    },
    {
        name: '玉米排骨汤', cats: ['清淡', '汤品'],
        tags: ['暖胃', '家常'], badge: '60分钟', kcal: 320, time: '约60分钟',
        emoji: '🌽', image: ''
    },
    {
        name: '蒜蓉西兰花', cats: ['清淡', '快手早餐'],
        tags: ['减脂', '低卡'], badge: '15分钟', kcal: 120, time: '约15分钟',
        emoji: '🥦', image: ''
    },
    {
        name: '番茄牛腩汤', cats: ['清淡', '汤品'],
        tags: ['高蛋白', '浓香'], badge: '90分钟', kcal: 410, time: '约90分钟',
        emoji: '🍲', image: ''
    },
    {
        name: '桂花酒酿圆子', cats: ['汤品', '甜品'],
        tags: ['暖甜', '秋冬'], badge: '甜品', kcal: 220, time: '约25分钟',
        emoji: '🍡', image: ''
    },
    {
        name: '麻辣香锅', cats: ['香辣'],
        tags: ['重口', '下饭'], badge: '25分钟', kcal: 660, time: '约25分钟',
        emoji: '🌶️', image: ''
    },
    {
        name: '酸辣土豆丝', cats: ['香辣', '快手早餐'],
        tags: ['爽脆', '下饭'], badge: '10分钟', kcal: 180, time: '约10分钟',
        emoji: '🥔', image: ''
    },
    {
        name: '小炒黄牛肉', cats: ['香辣'],
        tags: ['高蛋白', '下饭'], badge: '20分钟', kcal: 520, time: '约20分钟',
        emoji: '🥩', image: ''
    },
    {
        name: '麻婆豆腐', cats: ['香辣'],
        tags: ['麻香', '下饭'], badge: '15分钟', kcal: 320, time: '约15分钟',
        emoji: '🍛', image: ''
    },
    {
        name: '红豆双皮奶', cats: ['甜品'],
        tags: ['奶香', '微甜'], badge: '甜品', kcal: 260, time: '约40分钟',
        emoji: '🍮', image: ''
    },
    {
        name: '抹茶红豆卷', cats: ['甜品'],
        tags: ['抹茶', '甜品'], badge: '甜品', kcal: 300, time: '约50分钟',
        emoji: '🍥', image: ''
    },
    {
        name: '焦糖布丁', cats: ['甜品'],
        tags: ['奶香', '微甜'], badge: '甜品', kcal: 240, time: '约35分钟',
        emoji: '🍮', image: ''
    }
]

/* -------------------- 口味切换 -------------------- */
const activeCategory = ref(categories[0])

/* 当前分类下的菜品：数据一变，视图自动更新，不需要再手动 renderDishes() */
const filteredDishes = computed(() =>
    dishes.filter(d => d.cats.includes(activeCategory.value))
)

/* 标签配色：轻食类 → 绿，蛋白/家常类 → 黄，其它 → 红 */
const tagClass = (text) => {
    if (['减脂', '低卡', '低脂', '高纤维'].includes(text)) return 'tag-green'
    if (['高蛋白', '家常', '暖胃', '微甜', '奶香'].includes(text)) return 'tag-yellow'
    return 'tag-red'
}

/* -------------------- 轮播图 -------------------- */
const slideCount = 2
const current = ref(0)

let timer = null

function goTo(index) {
    current.value = (index + slideCount) % slideCount
}
function next() {
    goTo(current.value + 1)
}
function prev() {
    goTo(current.value - 1)
}

function startAuto() {
    stopAuto()
    timer = setInterval(next, 5000) // 5 秒自动切换轮播图
}
function stopAuto() {
    if (timer) clearInterval(timer)
    timer = null
}

/* 组件挂载后再开定时器，卸载时一定要清掉，否则会内存泄漏 */
onMounted(startAuto)
onBeforeUnmount(stopAuto)
</script>

<template>
    <!-- ================= 顶部导航 ================= -->
    <header class="topbar">
        <div class="topbar-inner">
            <a class="logo" href="#">
                <span class="logo-mark"></span>
                一日食光
            </a>

            <nav class="nav">
                <a class="is-active" href="#">首页</a>
                <a href="#">智能推荐</a>
                <a href="#">全部餐食</a>
            </nav>

            <div class="auth">
                <button class="btn btn-ghost" type="button">登录</button>
                <button class="btn btn-ghost btn-strong" type="button">注册</button>
            </div>
        </div>
    </header>

    <!-- ================= 轮播图 ================= -->
    <section class="hero" id="hero" @mouseenter="stopAuto" @mouseleave="startAuto">
        <div class="slides">

            <!-- 第 1 屏：左边文案 + 右边餐盘图 -->
            <article class="slide slide--peach" :class="{ 'is-active': current === 0 }">
                <div class="slide-inner">
                    <div class="hero-text">
                        <h1>今天吃什么？</h1>
                        <p>根据口味、忌口和时间，找到适合你的那一餐</p>
                        <div class="hero-actions">
                            <a class="btn btn-dark" href="#taste">去推荐</a>
                            <a class="btn btn-light" href="#taste">浏览全部餐食</a>
                        </div>
                    </div>

                    <div class="hero-media">
                        <!-- 首屏这张想要换成自己的图，把下面这段 .ph--hero 换成
                             <img :src="你的图片" alt="今日推荐餐盘" /> 即可（建议 1:1 方图） -->
                        <div class="ph ph--hero">
                            <span class="ph-emoji">🍱</span>
                            <span class="ph-text">餐盘大图占位</span>
                        </div>
                    </div>
                </div>
            </article>

            <!-- 第 2 屏：同一套布局，换成清爽的绿色主题 -->
            <article class="slide slide--mint" :class="{ 'is-active': current === 1 }">
                <div class="slide-inner">
                    <div class="hero-text">
                        <h1>一日三餐，清爽一点</h1>
                        <p>高蛋白、少油少盐，三十分钟就能端上桌</p>
                        <div class="hero-actions">
                            <a class="btn btn-green" href="#taste">看看本周菜单</a>
                            <a class="btn btn-light" href="#taste">按口味筛选</a>
                        </div>
                    </div>

                    <div class="hero-media">
                        <!-- 原图是 16:9、左侧自带标题文字，所以这里只取右边那块餐盘：
                             图片按高度铺满、右对齐，左边多出来的部分被裁掉 -->
                        <img :src="heroBanner" alt="高蛋白轻食餐盘" />
                    </div>
                </div>
            </article>

        </div>

        <button class="car-btn car-prev" type="button" aria-label="上一张" @click="prev">‹</button>
        <button class="car-btn car-next" type="button" aria-label="下一张" @click="next">›</button>

        <!-- 小圆点由 slides 数量生成，点一下就跳到对应那一屏 -->
        <div class="car-dots">
            <button v-for="i in slideCount" :key="i" class="car-dot" :class="{ 'is-active': current === i - 1 }"
                type="button" :aria-label="`第 ${i} 张`" @click="goTo(i - 1)"></button>
        </div>
    </section>

    <!-- ================= 口味切换 + 菜品列表 ================= -->
    <main class="panel" id="taste">
        <div class="tabs" id="tabs">
            <button v-for="name in categories" :key="name" class="tab"
                :class="{ 'is-active': name === activeCategory }" type="button" @click="activeCategory = name">
                {{ name }}
            </button>
        </div>

        <div class="dish-grid" id="dish-grid">
            <article v-for="d in filteredDishes" :key="d.name" class="dish">
                <div class="dish-thumb">
                    <img v-if="d.image" :src="d.image" :alt="d.name" />
                    <div v-else class="ph ph--dish">
                        <span class="ph-emoji">{{ d.emoji }}</span>
                    </div>
                </div>
                <div class="dish-body">
                    <div class="dish-tags">
                        <span v-for="t in d.tags" :key="t" class="tag" :class="tagClass(t)">{{ t }}</span>
                        <span class="badge">{{ d.badge }}</span>
                    </div>
                    <h3>{{ d.name }}</h3>
                    <div class="dish-meta">
                        <span>热量 {{ d.kcal }}kcal</span>
                        <span>{{ d.time }}</span>
                    </div>
                </div>
            </article>
        </div>

        <p v-if="!filteredDishes.length" class="empty">这个分类下还没有菜品哦～</p>
    </main>

    <!-- ================= 底部文案 ================= -->
    <footer class="tip">每日推荐 · 健康搭配 · 简单上手</footer>
</template>

<style scoped>
/* ===================================================================
   一日食光 · 组件样式
   结构：topbar → hero(轮播) → panel(tabs + 菜品) → tip
   只写组件内部元素；body / html 这类全局规则在 src/style.css 里
   =================================================================== */

/* ---------- 通用按钮 ---------- */
.btn {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 12px 26px;
    border: 0;
    border-radius: 999px;
    font-family: inherit;
    font-size: 15px;
    line-height: 1;
    color: #241F1A;
    text-decoration: none;
    cursor: pointer;
    transition: transform .15s ease, background .2s ease, box-shadow .2s ease;
}

.btn:hover {
    transform: translateY(-1px);
}

.btn-dark {
    background: #1E1B18;
    color: #fff;
}

.btn-dark:hover {
    background: #000;
}

.btn-light {
    background: #fff;
    box-shadow: 0 2px 12px rgba(90, 66, 40, .07);
}

.btn-light:hover {
    box-shadow: 0 6px 18px rgba(90, 66, 40, .12);
}

.btn-ghost {
    background: #fff;
    padding: 10px 22px;
    font-size: 14px;
}

.btn-ghost:hover {
    background: #F6EFE4;
}

/* =================== 顶部导航 =================== */
.topbar {
    background: #F8F3EB;
}

.topbar-inner {
    display: flex;
    align-items: center;
    gap: 40px;
    max-width: 1280px;
    height: 66px;
    margin: 0 auto;
    padding: 0 40px;
}

.logo {
    display: flex;
    align-items: center;
    gap: 10px;
    font-size: 20px;
    font-weight: 700;
    letter-spacing: .5px;
    color: #231F1B;
    text-decoration: none;
}

/* 参考图里那个小圆环图标，纯 CSS 画的 */
.logo-mark {
    position: relative;
    width: 22px;
    height: 22px;
    border: 2px solid #DCCEBB;
    border-radius: 50%;
}

.logo-mark::after {
    content: "";
    position: absolute;
    inset: 4px;
    border-radius: 50%;
    background: #F1E1C9;
}

.nav {
    display: flex;
    gap: 38px;
    margin: 0 auto;
}

.nav a {
    padding: 6px 0;
    font-size: 16px;
    color: #2C2A27;
    text-decoration: none;
    border-bottom: 2px solid transparent;
}

.nav a:hover {
    color: #000;
}

.nav a.is-active {
    font-weight: 600;
    border-bottom-color: #1E1B18;
}

.auth {
    display: flex;
    gap: 10px;
}

.btn-strong {
    font-weight: 600;
}

/* =================== 轮播图 =================== */
.hero {
    position: relative;
    height: clamp(310px, 22.5vw, 400px);
    overflow: hidden;
}

.slide {
    position: absolute;
    inset: 0;
    opacity: 0;
    transition: opacity .55s ease;
    pointer-events: none;
}

.slide.is-active {
    opacity: 1;
    pointer-events: auto;
}

.slide--peach {
    background: linear-gradient(100deg, #FCEBD8 0%, #F9E3CB 45%, #F6DBBF 100%);
}

.slide-inner {
    display: flex;
    align-items: center;
    gap: 30px;
    height: 100%;
    max-width: 1280px;
    margin: 0 auto;
    padding: 0 40px;
}

.hero-text {
    max-width: 560px;
}

.hero-text h1 {
    margin: 0;
    font-size: clamp(38px, 3.9vw, 62px);
    font-weight: 800;
    line-height: 1.08;
    letter-spacing: -1px;
}

.hero-text p {
    margin: 18px 0 0;
    font-size: clamp(14px, 1.1vw, 17px);
    color: #7B6C5C;
}

.hero-actions {
    display: flex;
    gap: 14px;
    margin-top: 28px;
}

.hero-media {
    display: flex;
    align-items: center;
    height: 100%;
    margin-left: auto;
}

/* 首屏大图：统一做成圆角卡片，和左边文案对齐 */
.hero-media img {
    display: block;
    width: 310px;
    height: 310px;
    object-fit: cover;
    border-radius: 22px;
}

/* 第 2 屏：绿色主题（图片只取餐盘那一块） */
.slide--mint {
    background: linear-gradient(100deg, #EEF5E8 0%, #E3EFD9 50%, #D9EACE 100%);
}

.slide--mint .hero-text h1 {
    color: #2C5733;
}

.slide--mint .hero-text p {
    color: #62745A;
}

.btn-green {
    background: #2F5D3A;
    color: #fff;
}

.btn-green:hover {
    background: #26492F;
}

/* 图片 16:9，卡片偏竖，于是按高度铺满、往右对齐，
   左边那一截刚好被裁掉 —— 遗留在图里的标题文字就不会露出来 */
.slide--mint .hero-media img {
    object-position: 100% 50%;
}

/* 整屏铺满的图片（如果某一屏想直接用整张图，套这个类） */
.slide-cover {
    width: 100%;
    height: 100%;
    object-fit: cover;
    object-position: center 45%;
    display: block;
}

/* 左右切换按钮 */
.car-btn {
    position: absolute;
    top: 50%;
    z-index: 5;
    display: grid;
    place-items: center;
    width: 40px;
    height: 40px;
    padding-bottom: 3px;
    border: 0;
    border-radius: 50%;
    background: rgba(255, 255, 255, .82);
    color: #3A342E;
    font-size: 24px;
    line-height: 1;
    cursor: pointer;
    transform: translateY(-50%);
    transition: background .2s ease;
}

.car-btn:hover {
    background: #fff;
}

.car-prev {
    left: 20px;
}

.car-next {
    right: 20px;
}

/* 底部小圆点 */
.car-dots {
    position: absolute;
    bottom: 52px;
    left: 50%;
    z-index: 12;
    display: flex;
    gap: 8px;
    transform: translateX(-50%);
}

.car-dot {
    width: 8px;
    height: 8px;
    padding: 0;
    border: 0;
    border-radius: 999px;
    background: rgba(36, 31, 26, .22);
    cursor: pointer;
    transition: width .25s ease, background .25s ease;
}

.car-dot.is-active {
    width: 22px;
    background: #1E1B18;
}

/* =================== 菜品面板 =================== */
.panel {
    position: relative;
    z-index: 10;
    max-width: 1280px;
    margin: -26px auto 0;
    padding: 24px 34px 34px;
    background: #FBF7F1;
    border-radius: 26px;
    box-shadow: 0 18px 40px rgba(120, 95, 70, .08);
}

.tabs {
    display: flex;
    flex-wrap: wrap;
    gap: 6px;
    padding-bottom: 18px;
}

.tab {
    padding: 9px 20px;
    border: 0;
    border-radius: 999px;
    background: transparent;
    font-family: inherit;
    font-size: 15px;
    color: #4A423A;
    cursor: pointer;
    transition: background .2s ease, color .2s ease;
}

.tab:hover {
    background: #F5EBDC;
}

.tab.is-active {
    background: #F3E4CB;
    color: #241F1A;
    font-weight: 600;
}

/* 4 列卡片 */
.dish-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 18px;
}

.dish {
    padding: 10px;
    background: #fff;
    border-radius: 16px;
    box-shadow: 0 6px 18px rgba(120, 95, 70, .06);
    animation: fade-up .35s ease both;
}

@keyframes fade-up {
    from {
        opacity: 0;
        transform: translateY(8px);
    }

    to {
        opacity: 1;
        transform: none;
    }
}

.dish-thumb {
    aspect-ratio: 287 / 182;
    overflow: hidden;
    border-radius: 12px;
    background: #F3EBDF;
}

.dish-thumb img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    display: block;
}

.dish-body {
    padding: 12px 6px 6px;
}

.dish-tags {
    display: flex;
    align-items: center;
    gap: 6px;
    margin-bottom: 9px;
}

.tag {
    padding: 3px 9px;
    border-radius: 6px;
    font-size: 12.5px;
    line-height: 1.35;
}

.tag-green {
    background: #E3EFD8;
    color: #4B7A2F;
}

.tag-yellow {
    background: #EEEFC6;
    color: #6B742A;
}

.tag-red {
    background: #FBE2DC;
    color: #A8452F;
}

.badge {
    margin-left: auto;
    padding: 3px 9px;
    border-radius: 6px;
    background: #F9E6CC;
    color: #A8712B;
    font-size: 12.5px;
}

.dish-body h3 {
    margin: 0 0 10px;
    font-size: 17px;
    font-weight: 600;
}

.dish-meta {
    display: flex;
    justify-content: space-between;
    font-size: 13.5px;
    color: #8C847A;
}

.empty {
    padding: 40px 0 20px;
    text-align: center;
    color: #9A9187;
}

/* =================== 底部文案 =================== */
.tip {
    max-width: 1212px;
    margin: 22px auto 40px;
    padding: 26px 20px;
    background: #FBF7F1;
    border-radius: 20px;
    text-align: center;
    font-size: 17px;
    letter-spacing: .5px;
    color: #4A423A;
}

/* =================== 图片占位块 ===================
   只是为了先占住位置，正式使用时会换成 <img>，可以整段删掉
   ================================================= */
.ph {
    display: grid;
    place-items: center;
    gap: 6px;
    color: #8A7A66;
    background:
        repeating-linear-gradient(45deg, #EFE5D8 0 10px, #F5EDE3 10px 20px);
    border: 2px dashed #D8C9B4;
    border-radius: 22px;
}

.ph--hero {
    width: 310px;
    height: 310px;
    margin-right: 40px;
}

.ph--dish {
    width: 100%;
    height: 100%;
    border: 0;
    border-radius: 0;
    background: linear-gradient(140deg, #F7EEE2, #EFE1CD);
}

.ph-emoji {
    font-size: 54px;
    line-height: 1;
}

.ph--dish .ph-emoji {
    font-size: 44px;
    opacity: .75;
}

.ph-text {
    font-size: 13px;
}

/* =================== 响应式 =================== */
@media (max-width: 1440px) {

    /* 窄一点的屏幕上，给左右切换按钮留出位置，别压到标题 */
    .slide-inner {
        padding: 0 84px;
    }
}

@media (max-width: 1080px) {
    .dish-grid {
        grid-template-columns: repeat(2, 1fr);
    }

    .ph--hero {
        width: 300px;
        height: 270px;
        margin-right: 0;
    }

    .hero-media img {
        width: 300px;
        height: 270px;
    }
}

@media (max-width: 820px) {
    .topbar-inner {
        gap: 20px;
        padding: 0 20px;
    }

    .nav {
        display: none;
    }

    .auth {
        margin-left: auto;
    }

    .hero {
        height: 320px;
    }

    .slide-inner {
        padding: 0 24px;
    }

    .hero-media {
        display: none;
    }

    .panel {
        margin: -20px 14px 0;
        padding: 18px 16px 24px;
        border-radius: 20px;
    }

    .dish-grid {
        grid-template-columns: 1fr 1fr;
        gap: 12px;
    }

    .tip {
        margin: 18px 14px 30px;
        font-size: 15px;
    }
}

@media (max-width: 520px) {
    .dish-grid {
        grid-template-columns: 1fr;
    }

    .car-btn {
        display: none;
    }
}
</style>
