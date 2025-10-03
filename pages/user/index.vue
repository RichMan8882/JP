<script lang="ts" setup>
const { $swiper } = useNuxtApp()
const router = useRouter()
const siteStore = useSiteStore()
const { locale, t } = useI18n()
const lang = locale.value
const { queryNews } = useSiteStore()
const news = ref([])




// methods
const openChatBox = () => {
  const urlPattern = new RegExp(
    '^(https?:\\/\\/)?' + // protocol
    '((([a-z\\d]([a-z\\d-]*[a-z\\d])*)\\.)+[a-z]{2,}|' + // domain name and extension
    '((\\d{1,3}\\.){3}\\d{1,3}))' + // OR ip (v4) address
    '(\\:\\d+)?(\\/[-a-z\\d%_.~+]*)*' + // port and path
    '(\\?[;&a-z\\d%_.~+=-]*)?' + // query string
    '(\\#[-a-z\\d_]*)?$',
    'i'
  ) // fragment locator
  if (urlPattern.test(siteStore.chatbox)) {
    window.open(siteStore.chatbox, '_blank')
  }
}
const gameWindowOpen = () => {
  window.open(`${window.location.origin}/game`, '_blank')
}
await useAsyncData(async () => {
  const path = 'user'
  const queryNewsRes = await queryNews(lang, path)
  news.value = queryNewsRes.data.result
  console.log('queryNewsRes.data', queryNewsRes.data)
})

const handleNavigateTo = (path: string) => {
  if (path === 'customer') {
    openChatBox()
    return
  }
  if (path === 'game') {
    gameWindowOpen()
    return
  }
  navigateTo(path)
}
const filterPledgeList = computed(() => {
  return dashboardList.value.filter(
    (item) =>
      item.path !== '/user/pledge' ||
      siteStore.siteData?.gameType?.find((item: any) => item === 5)
  )
})


import zhu1 from '@/assets/image/index/zhu1.jpg'
import zhu2 from '@/assets/image/index/zhu2.jpg'
import zhu3 from '@/assets/image/index/zhu3.jpg'
import zhu4 from '@/assets/image/index/zhu4.jpg'

import fu1 from '@/assets/image/index/fu1.jpg'
import fu2 from '@/assets/image/index/fu2.jpg'
import fu3 from '@/assets/image/index/fu3.jpg'
import fu4 from '@/assets/image/index/fu4.jpg'
const imgs1 = ref([
  {
    feature: '綜合臨床供應服務',
    content: ['為I期到III期及以後各種規模和複雜性的臨床研究提供最廣泛的全球開發和臨床供應服務', ''],
    img: zhu1
  },
  {
    feature: '全球對照藥採購',
    content: ['我們的全球藥品網絡', '協助您實現高性價比的商業藥品採購'],
    img: zhu2
  },
  {
    feature: '按需供應模式',
    content: ['將傳統的臨床供應鏈', '轉變為以病患需求為驅動的供應鏈'],
    img: zhu3
  },
  {
    feature: '專案&臨床供應管理',
    content: ['我們有充足的經驗', '來優化您的臨床設計'],
    img: zhu4
  }
])
const imgs2 = ref([
  fu1,
  fu2,
  fu3,
  fu4
])
const swiper = ref()
const swiper2 = ref()
const realIndex = ref()
const activeIndex = ref()
onMounted(() => {
  swiper.value = new $swiper('.swiper-pc', {
    allowTouchMove: true,
    slidesPerView: 1,
    loop: true,
    autoplay: true,
    centeredSlides: false
  })
  swiper2.value = new $swiper('.swiper-pc2', {
    allowTouchMove: true,
    loop: true,
    // slidesPerView: 4,
    slidesPerView: 'auto',
    auto: true,
    paginationClickable: true,
  })
  swiper.value.on('realIndexChange', (swiperInstance) => {
    realIndex.value = swiperInstance.realIndex; // 实际索引（0-based）
    activeIndex.value = swiperInstance.activeIndex; // 活动索引（含克隆幻灯片）

    // console.log('真实索引:', realIndex.value);
    // console.log('活动索引:', activeIndex.value);
    swiper2.value.slideTo(activeIndex.value)
    // 自定义操作示例：
    // document.querySelector('.current-index').textContent = realIndex + 1;
  });
})

const upTo = () => {
  swiper.value.slidePrev();  // 上一张
}
const downTo = () => {
  swiper.value.slideNext();  // 下一张
}


</script>
<template>
  <div class="dashboard">
    <!-- <ul class="dashboard-list">
      <li v-for="item in filterPledgeList" :key="item.title" class="dashboard-list-item shadow"
        @click="handleNavigateTo(item.path)">
        <span v-html="item.icon"></span>
        <h4>{{ item.title }}</h4>
      </li>
    </ul> -->
    <main>
      <div class="swp-vw">
        <div class="v-margin">
          <div class="left-swp">
            <div class="imgs">
              <div class="swiper swiper-pc">
                <div class="swiper-wrapper">
                  <div v-for="(item, index) in imgs1" :key="index" class="swiper-slide">
                    <div style="position: relative;">
                      <img :src="item.img" />
                    </div>
                    <div class="swiper-cont">
                      <h4>{{ item.feature }}</h4>
                      <h2 v-for="(txt, index) in item.content" :key="index">{{ txt }}</h2>
                      <span>進入系統
                        <svg t="1739515035390" class="icon" viewBox="0 0 1024 1024" version="1.1"
                          xmlns="http://www.w3.org/2000/svg" p-id="1851" xmlns:xlink="http://www.w3.org/1999/xlink"
                          width="20" height="20">
                          <path class="color"
                            d="M416 192v64h306.016L200.96 776.992 247.04 823.04 768 301.984V608h64V192H416z"
                            p-id="1852">
                          </path>
                        </svg>
                      </span>
                    </div>
                  </div>
                </div>
              </div>
              <div class="fix-btn">
                <div class="btn" @click="upTo">
                  <i class="fa-solid fa-angle-down fa-flip-vertical"></i>
                </div>
                <div class="btn" @click="downTo">
                  <i class="fa-solid fa-angle-down"></i>
                </div>
              </div>
            </div>
          </div>
          <div class="right-swp">
            <div class="imgs">
              <div class="swiper swiper-pc2">
                <div class="swiper-wrapper">
                  <div v-for="(image, index) in imgs2" :key="index" class="swiper-slide">
                    <img :src="image" />
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>
    <div style="margin: 0 clamp(15px, 3.15vw, 60px);padding-bottom: 10px;">
      <div class="lis-cont">
        <ul>
          <li>
            <div class="li-box" @click="handleNavigateTo('game')">
              <img src="@/assets/image/index/nav01.jpg" alt="">
              <div class="text">
                訂單中心
              </div>
              <div class="ico">
                <i class="fa-solid fa-angle-right" style="width:20px;height: 20px;"></i>
              </div>
            </div>
          </li>
          <li>
            <div class="li-box" @click="handleNavigateTo('/user/info')">
              <img src="@/assets/image/index/nav02.jpg" alt="">
              <div class="text">
                職員資料
              </div>
              <div class="ico">
                <i class="fa-solid fa-angle-right" style="width:20px;height: 20px;"></i>
              </div>
            </div>
          </li>
          <li>
            <div class="li-box" @click="handleNavigateTo('/user/bank')">
              <img src="@/assets/image/index/nav03.jpg" alt="">
              <div class="text">
                薪資專區
              </div>
              <div class="ico">
                <i class="fa-solid fa-angle-right" style="width:20px;height: 20px;"></i>
              </div>
            </div>
          </li>
          <li>
            <div class="li-box" @click="handleNavigateTo('/user/record')">
              <img src="@/assets/image/index/nav04.jpg" alt="">
              <div class="text">
                財務中心
              </div>
              <div class="ico">
                <i class="fa-solid fa-angle-right" style="width:20px;height: 20px;"></i>
              </div>
            </div>
          </li>
        </ul>
      </div>
    </div>
    <div>
      <div class="ul-container">
        <section class="ul-ad">
          <div class="ul-inner-container">
            <div class="ul-ad-content">
              <div class="ul-ad-txt">
                <span class="ul-ad-sub-title">全球臨床供應設施網絡</span>
                <h2 class="ul-section-title" style="text-shadow:2px 3px 5px #4d4d4d;">在美國、歐洲和亞太地區擁有9大臨床設施</h2>
                <div class="ul-ad-categories">
                  <span class="category"><span><i class="fa-solid fa-check"></i></span>北美</span>
                  <span class="category"><span><i class="fa-solid fa-check"></i></span>歐洲</span>
                  <span class="category"><span><i class="fa-solid fa-check"></i></span>亞太</span>
                  <span class="category"><span><i class="fa-solid fa-check"></i></span>全球倉儲</span>
                </div>
              </div>
              <div class="ul-ad-img">
                <img src="@/assets/image/index/ad01-img.png" alt="Ad Image">
              </div>
              <a class="ul-btn" style="font-weight: bold;">了解詳情
                <svg t="1739515035390" class="icon" viewBox="0 0 1024 1024" version="1.1"
                  xmlns="http://www.w3.org/2000/svg" p-id="1851" xmlns:xlink="http://www.w3.org/1999/xlink" width="20"
                  height="20">
                  <path class="color" d="M416 192v64h306.016L200.96 776.992 247.04 823.04 768 301.984V608h64V192H416z"
                    p-id="1852">
                  </path>
                </svg>
              </a>
            </div>
          </div>
        </section>
      </div>
    </div>
  </div>
</template>

<style scoped lang="sass">
.ul-container
  margin: 0 clamp(15px, 3.15vw, 60px)
.swp-vw
  // overflow: hidden
  margin: clamp(20px, 1.58vw, 30px) 0
  width: 100%
  // @media ( max-width: 1320px )
  //     max-width: 1320px
  .v-margin
    display: flex
    gap: clamp(15px, 1.58vw, 30px)
    .imgs
      width:100%
    .swiper-pc
      width: 100%
      height: 100%
    .left-swp
      // min-width: 50%
      width: max-content
      position: relative
      z-index: 11
      background-color: white
      position: relative
      .swiper-cont
        position: absolute
        top: 50%
        transform: translateY(-50%)
        left: clamp(15px, 3.15vw, 60px)
        max-width: 58%
        h4
          font-size: clamp(16px, 1.05vw, 20px)
          letter-spacing: 0.3em
          color: #025189
          margin-bottom: clamp(10px, 1.26vw, 24px)
          display: inline-block
        h2
          font-size: clamp(32px, 3.42vw, 25px)
          font-weight: 600
          line-height: 108%
          letter-spacing: -0.03em
          color: var(--black)
          margin-bottom: clamp(7px, 0.89vw, 17px)
        span
          color: black
          border-radius: 999px
          border: 1px solid black
          box-sizing: border-box
          display: inline-flex
          align-items: center
          justify-content: center
          gap: clamp(12px, 1.05vw, 20px)
          height: clamp(45px, 3.15vw, 60px)
          line-height: clamp(45px, 3.15vw, 60px)
          padding: 0 clamp(20px, 1.58vw, 30px)
          font-weight: 400
          font-size: clamp(13px, 0.84vw, 16px)
          position: relative
          text-transform: uppercase
          transition: all .3s ease
          &:hover
            background-color: #EF2853
            color: #FFF
            border-color: #EF2853
            .icon
              fill:#FFF

      .imgs
        max-width: clamp(310px, 59.12vw, 1125px)
        border-radius: 30px
        overflow: hidden
        position: relative
      .fix-btn
        position: absolute
        right: 0
        top: 50%
        z-index: 9
        transform: translateY(-50%)
        background-color: #fff
        border-top-left-radius: 20px
        border-bottom-left-radius: 20px
        .btn
          display: flex
          align-items: center
          padding: 10px
          font-size: 22px
          color: black
          font-weight: 400
          transition: all .3s ease
          &:hover
            color:#EF2853
      img.carousel-img
          width: 100%
    .right-swp
      // max-width: clamp(270px, 23.65vw, 450px)
      border-radius: 20px
      height: 100%
      .swiper-slide
        border-radius: 20px
        overflow: hidden
        max-width: clamp(270px, 23.65vw, 450px)
        // height: 100%
        margin: 0 10px 0 0
.lis-cont
  margin: clamp(40px, 4.2vw, 80px) clamp(15px, 2.88vw, 150px)
.lis-cont ul
  margin:0 clamp(15px, 2.88vw, 150px)
  padding: 0 clamp(30px,5.4vw, 150px)
  display: flex
  align-items: center
  justify-content: center
  flex-wrap: wrap
.lis-cont ul li
  width:25%
  padding: 0 15px
  // margin-top: 30px
  .li-box
    display: flex
    justify-content: space-between
    cursor: pointer
    border: 1px solid #c2c2c2
    font-size: 17px
    border-radius: 10px
    padding: clamp(6px, 0.47vw, 9px)
    display: flex
    align-items: center
    gap: clamp(12px, 1.05vw, 20px)
    position: relative
    z-index: 1
    &::before
      content: ""
      position: absolute
      inset: 0
      border-radius: 10px
      opacity: 0
      background: linear-gradient(#3466f1c2, #025189b9)
      transition: 0.4s ease
      z-index: -1
    &:hover
      color: #FFF
      &:before
        opacity: 1
    
    img
      max-width: 100%
      border-radius: 50%
      width: clamp(39px, 3.1vw, 59px)
      aspect-ratio: 1 / 1
      object-fit: cover
    .text
      font-weight: bold
    .ico
      width: clamp(39px, 3.1vw, 59px)
      aspect-ratio: 1 / 1
      object-fit: cover
      display: flex
      justify-content: center
      align-items: center
      background-color: #F8E6E2
      border-radius: 50%
      color: #025189
      font-size: clamp(13px, 0.84vw, 16px)
    &:hover .ico
      background-color: #FFF
.ul-ad
  box-sizing: border-box
  margin: clamp(40px, 4.2vw, 80px) 0
  border-radius: clamp(15px, 2.63vw, 50px)
  background: #0d70b680
  padding-top: clamp(37px, 3vw, 57px)
  position: relative
  overflow: hidden
  z-index: 1
  &::before 
    content: ""
    position: absolute
    inset: 0
    background: url(@/assets/image/index/banner-bg.jpg) no-repeat center center
    background-size: cover
    mix-blend-mode: multiply
    opacity: 30%
    z-index: -1
  .ul-inner-container 
    margin: 0 clamp(15px, 10.25vw, 195px)
  .ul-ad-content 
    display: flex
    align-items: flex-start
    align-items: center
    justify-content: space-between
    color: white
  .ul-ad-txt, .ul-ad .ul-btn 
    margin-bottom: clamp(15px, 1.84vw, 35px)
  .ul-btn 
    border-radius: 999px
    border: 1px solid white
    display: inline-flex
    align-items: center
    justify-content: center
    gap: clamp(12px, 1.05vw, 20px)
    height: clamp(45px, 3.15vw, 60px)
    line-height: clamp(45px, 3.15vw, 60px)
    padding: 0 clamp(20px, 1.58vw, 30px)
    font-weight: 400
    font-size: clamp(13px, 0.84vw, 16px)
    position: relative
    text-transform: uppercase
    color: white
    flex-shrink: 0
    transition: all .3s ease
    margin-bottom: clamp(15px, 1.84vw, 35px)
    .color
      fill:#FFF
      transition: all .3s ease
    // .right-arrow

    &:hover 
      background-color: #FFF
      color: #EF2853
      .color
        fill: #EF2853
  .ul-ad-sub-title 
    background-color: white
    border-radius: 999px
    padding: clamp(2px, 0.26vw, 5px) clamp(7px, 0.53vw, 10px)
    margin-bottom: clamp(10px, 1.05vw, 20px)
    font-size: clamp(11px, 0.74vw, 14px)
    text-transform: uppercase
    color: #025189
    display: inline-block
  .ul-section-title 
    color: white
    margin-bottom: clamp(12px, 1.05vw, 20px)
    font-weight: 600
    font-size: clamp(25px, 2.1vw, 40px)
    letter-spacing: -0.03em
    color: var(--black)
  .ul-ad-categories 
    display: flex
    gap: clamp(15px, 2.1vw, 40px)
    flex-wrap: wrap
    .category 
      font-weight: 500
      font-size: clamp(11px, 0.74vw, 14px)
      text-transform: uppercase
      display: inline-flex
      gap: clamp(7px, 0.53vw, 10px)


@media screen and (max-width: 1140px)
  .right-swp
    display: none
  .left-swp
    width: 100% !important
    padding: 0 16px
    .imgs
      max-width: 100% !important
  .lis-cont ul
    margin: 0 0
    padding: 0 0
@media screen and (max-width: 992px)
  .lis-cont ul li
    width: 50%
    margin-bottom: 10px
    
@media screen and (max-width: 768px)
  .lis-cont ul li
    width: 100%
    margin-bottom: 10px
  .ul-ad .ul-ad-content
    display: block !important
  .ul-ad-img img
    width: 100%
  .ul-ad .ul-btn
    width: 100%
  .left-swp
    border-radius: 0
    .imgs
      border-radius: 0 !important
      img
        object-fit: cover
        border-radius: 30px
        object-position: 100%
        aspect-ratio: 400 / 459
  .swiper-cont
    padding: 15px 0 0
    max-width: 100% !important
    width: auto
    position: relative !important
    top: 100% !important
    transform: translateY(0) !important
    left: 0 !important
    height: 250px
  .fix-btn
    right: auto !important
    top: auto !important
    bottom:250px  !important
    left: 50%
    display: flex
    transform: translateX(-50%) !important
    border-bottom-left-radius: 0 !important
    border-top-left-radius: 20px
    border-top-right-radius: 20px
    .btn
      transform: rotateZ(-90deg)
        
</style>
