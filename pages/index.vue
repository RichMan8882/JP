<script lang="ts" setup>
import documentContent from '@/assets/document/index.js'
const documentContentData = computed(() => {
  return documentContent()
})
const router = useRouter()
const { isLogin } = useAuthStore()

const siteStore = useSiteStore()
const { queryChatbox } = siteStore
const referrerCodeCookiee = useCookie('referrerCode', {
  expires: new Date(Date.now() + 1000 * 60 * 60 * 24 * 7)
}) as any
const accessTokenCookie = useCookie('accessToken', {
  expires: new Date(Date.now() + 1000 * 60 * 60 * 24 * 7)
}) as any
const mobileNavOpen = ref(false)
const { t } = useI18n()
const headerNavList = ref([
  {
    title: `${t('首頁')}`,
    router: '/user'
  },
  {
    title: `${t('關於我們')}`,
    router: '/about'
  },
  {
    title: `${t('聯係客服')}`,
    router: 'customer'
  }
])
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
// action
await onMounted(async () => {
  console.log('register page')
  if (router.currentRoute.value.query.referrer) {
    referrerCodeCookiee.value = router.currentRoute.value.query.referrer
  }
  await queryChatbox()
})
const gameWindowOpen = () => {
  if (accessTokenCookie) {
    window.open(`${window.location.origin}/game`, '_blank')
  } else {
    navigateTo('/login')
  }
}
const { locale } = useI18n()
const changeLang = () => {
  locale.value = 'en_US'
  console.log('change lang', locale)
}

const open = ref(false)
const onOpen = () => {
  open.value = true
}
const onClose = () => {
  open.value = false
}

const pageRef = ref()
const isWhite = ref()
const isTop = ref()
const handleScroll = () => {
  console.log(pageRef.value.scrollTop)
  isTop.value = pageRef.value.scrollTop >= 500
  isWhite.value = pageRef.value.scrollTop >= 100
}
onMounted(() => {
  pageRef.value.addEventListener('scroll', handleScroll)
})
const upTo = () => {
  pageRef.value.scrollTop = 0
}
</script>

<template>
  <div ref="pageRef" class="page">
    <client-only>
      <!-- <document :content="documentContentData"></document> -->
      <!-- <index-header /> -->
      <!-- <index-mobile-header /> -->
      <div v-if="!isLogin()" class="user-t white">
        <a class="login-off a-aim white" @click="navigateTo('/login')">
          <span style="margin-right: 5px">
            <i class="fa-solid fa-award" style="color: #0e63ff"></i>
          </span>
          職員登入
        </a>
      </div>
      <nav :class="{ sticky: isWhite }">
        <div class="menu-flex">
          <div class="logo">
            <img src="#" alt="" />
          </div>
          <div class="menu-nav">
            <ul>
              <li v-for="item in headerNavList" :key="item.title">
                <a class="nav-sty black a-aim" @click="handleNavigateTo(item.router)">{{ $lang(item.title) }}</a>
              </li>
              <li>
                <a class="nav-sty black a-aim" @click="handleNavigateTo('/user')"><span style="
                      border: 1px solid;
                      padding: 6px;
                      background-color: rgb(208, 233, 255);
                      border-radius: 5px;
                    ">{{ $lang('職員系統') }}</span></a>
              </li>
            </ul>
            <div class="sid" @click="onOpen">
              <i class="fa-solid fa-bars" style="height: 30px; width: 26px; color: #0e63ff"></i>
            </div>
          </div>
          <a class="white" @click="navigateTo('/user')">
            <div class="menu-btn">
              {{ $lang('職員專區') }}
            </div>
          </a>
        </div>
      </nav>

      <div class="page-body">
        <router-view></router-view>
      </div>
      <!-- <index-footer /> -->
      <footer>
        <!-- <div style="border-top: 1px solid;padding: 30px 0;">
        <div style="margin-left: 24px;margin-right: 24px;">
          <p style="font-size: 14px;text-align: center;line-height: 16px;">
            使用本網站期間，請您瞭解並接受Catalent 公司追蹤您的網站活動，以便能夠為您提供更有針對性的回應或信息，以滿足您的要求，您的個人資料將根據我們的隱私權政策進行保存| 資料隱私權準則聲明。
            <br />
            <br />
            版權所有2024，該網站內容所有權歸Catalent 所有
          </p>
        </div>
      </div> -->
      </footer>
      <!-- top -->
      <button v-if="isTop" class="scroll-top scroll-to-target open" data-target="html" @click="upTo">
        <i class="fas fa-level-up-alt"></i>
      </button>
      <!-- 彈出 -->
      <div class="sidebar" :class="open ? 'active' : ''">
        <div class="ul-sidebar-header">
          <div class="ul-sidebar-header-logo"></div>
          <button class="ul-sidebar-closer" @click="onClose">
            <i class="fa-solid fa-xmark fa-xl" style="color: #6c757d"></i>
          </button>
        </div>
        <div class="ul-sidebar-header-nav-wrapper">
          <ul>
            <li v-for="item in headerNavList" :key="item.title">
              <a class="nav-sty black a-aim" @click="handleNavigateTo(item.router)">{{ $lang(item.title) }}</a>
            </li>
            <li>
              <a class="nav-sty black a-aim" @click="handleNavigateTo('/user')"><span style="
                    border: 1px solid;
                    padding: 6px;
                    background-color: rgb(208, 233, 255);
                    border-radius: 5px;
                  ">{{ $lang('職員系統') }}</span></a>
            </li>
          </ul>
        </div>
      </div>
    </client-only>
  </div>
</template>
<style lang="sass">
body
  // background-color: #08001a

  a,
  button
    cursor: pointer
</style>

<style lang="sass">
.only-pc
  display: none

@media screen and (min-width: 768px)
  .only-mobile
    display: none !important

  .only-pc
    display: flex
</style>

<style scoped lang="sass">
@import '@/assets/sass/index/index.sass'
.sticky
  position: sticky
  top:0
  z-index: 99
  padding-top: 30px
  background-color: #fff
  box-shadow: 0 10px 15px rgba(25, 25, 25, 0.1)
  animation: 1000ms ease-in-out 0s normal none 1 running fadeInDown
@keyframes fadeInDown
  0%
    opacity: 0
    -webkit-transform: translate3d(0,-100%,0)
    transform: translate3d(0,-100%,0)
  to
    opacity: 1
    -webkit-transform: translateZ(0)
    transform: translateZ(0)
// sidebar
.sidebar
  position: fixed
  right: 0
  top: 0
  bottom: 0
  height: 100%
  overflow: auto
  background: #fff
  z-index: 999
  transform: translateX(100%)
  transition: 0.4s ease
  padding: clamp(20px, 1.58vw, 30px) 0
  max-width: 100%
  border-left: 1px solid  #025189
  display: flex
  flex-direction: column
  justify-content: space-between
  width: clamp(320px, 40vw, 100%)
  @media screen and (max-width: 991px)
    justify-content: start
.ul-sidebar-closer
  transition: all 1.4s ease
.active
  transform: translateX(0) !important
  .ul-sidebar-closer
    transform: rotateZ(360deg)
.ul-sidebar-header
  display: flex
  align-items: flex-end
  justify-content: space-between
  border-bottom: 1px solid rgb(0 0 0 / 10%)
  padding-bottom: clamp(20px, 1.58vw, 30px)
  padding-right: 20px
.ul-sidebar-header-nav-wrapper
  ul li
    font-weight: bold
    padding: 10px clamp(20px, 1.58vw, 30px)
    position: relative
    display: block
    border-bottom: 1px solid rgb(0 0 0 / 10%)

.page
  scroll-behavior: smooth
  // background: url('@/assets/image/index/body.jpg') top no-repeat #fff
  position: relative
  width: 100dvw
  height: 100dvh
  overflow-y: auto
  display: flex
  flex-direction: column
  background-color: #fff

.scroll-top
  width: 50px
  height: 50px
  line-height: 50px
  position: fixed
  bottom: 0%
  right: 50px
  font-size: 16px
  border-radius: 6px
  z-index: 99
  color: white
  text-align: center
  cursor: pointer
  background: #0e63ff
  transition: 1s ease
  border: none
  animation: 1000ms ease-in-out forwards fadeInDown2
  &::after
    position: absolute
    z-index: -1
    content: ''
    top: 100%
    left: 5%
    height: 10px
    width: 90%
    opacity: 1
    background: radial-gradient(ellipse at center, rgba(0, 0, 0, 0.25) 0%, rgba(0, 0, 0, 0) 80%)
@keyframes fadeInDown2
  0%
    opacity: 0
    transform: translateZ(0)
  to
    opacity: 1
    transform: translate3d(0,-100%,0)
</style>
