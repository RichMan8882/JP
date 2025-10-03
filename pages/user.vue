<script lang="ts" setup>
import { Swiper, SwiperSlide } from 'swiper/vue'
import { Navigation, Pagination, Scrollbar, A11y } from 'swiper/modules'
import 'swiper/css'
import 'swiper/css/navigation'
import 'swiper/css/pagination'
import 'swiper/css/scrollbar'
const PlayerStore = usePlayerStore()
const siteStore = useSiteStore()
const { signout } = useAuthStore()
const router = useRouter()
const homeRouter = ref('/')
const ticketStore = useTicketStore()
const { queryTicket } = ticketStore
const intervalId = ref(null)
const { queryChatbox, queryNews } = siteStore
const { locale } = useI18n()
const lang = locale.value
const { t } = useI18n()

const documentContent = ref([])
const playerWalletBalance = computed(() => {
  const findWallet = PlayerStore.playerInfo.wallet.find(
    (wallet: any) => wallet.type === 1
  )
  if (findWallet) {
    return Number(findWallet.balance)
  } else {
    return 0
  }
})
const isDashboard = computed(() => {
  return router.currentRoute.value.name === 'user'
})
const dashboardList = ref([
  {
    title: t('職員首頁'),
    path: '/user',
    icon: '<i class="fa-brands fa-500px"></i>'
  },
  {
    title: t('訂單中心'),
    path: 'game',
    icon: '<i class="fa-brands fa-500px"></i>'
  },
  {
    title: t('職員資料'),
    path: '/user/info',
    icon: '<i class="fa-solid fa-gears"></i>'
  },
  {
    title: t('薪資專區'),
    path: '/user/bank',
    icon: '<i class="fa-solid fa-handshake-angle"></i>'
  },
  {
    title: t('財務中心'),
    path: '/user/record',
    icon: '<i class="fa-solid fa-file-invoice"></i>'
  },
  {
    title: t('質押'),
    path: '/user/pledge',
    icon: '<i class="fas fa-coins"></i>'
  },
  {
    title: t('聯絡我們'),
    path: 'customer',
    icon: '<i class="fa-brands fa-facebook-messenger"></i>'
  },
  {
    title: t('登出帳號'),
    path: '/',
    icon: '<i class="fa-solid fa-file-invoice"></i>'
  }
])
await useAsyncData(async () => {
  console.log('user.vue useAsyncData')
  if (siteStore.siteData?.smsVerify === 1) {
    if (!PlayerStore.playerInfo?.smsVerify) {
      await navigateTo('/user/smsVerify')
    }
  }
  const path = 'user'
  await queryNews(lang, path)
})
const checkRouter = () => {
  console.log('router.currentRoute.value.name', router.currentRoute.value.name)
  if (router.currentRoute.value.name === 'user') {
    homeRouter.value = '/'
  } else {
    homeRouter.value = '/user'
  }
}
const displayedActivities = computed(() => {
  return siteStore.news.result
})
await onMounted(async () => {
  // setDocument()
  checkRouter()
  const queryChatboxPromise = queryChatbox()
  queryChatboxPromise
    .then((queryChatboxRes) => {
      // 處理響應結果
      console.log('queryChatboxRes', queryChatboxRes.data)
    })
    .catch((error) => {
      console.error('Error loading queryChatboxRes data:', error)
    })
})
onBeforeUnmount(() => {
  if (intervalId.value) {
    console.log('clearInterval queryTicket', intervalId.value)
    clearInterval(intervalId.value)
    intervalId.value = null
  }
})
const marqueeText = computed(() => {
  let text = ''
  for (let i = 0; i <= 5; i++) {
    if (!displayedActivities.value[i]) {
      break
    }
    text += displayedActivities.value[i].title + ' '
  }
  return text
})
watch(
  () => router.currentRoute.value.name,
  () => {
    checkRouter()
    // setDocument()
    if (siteStore.siteData?.smsVerify === 1) {
      if (!PlayerStore.playerInfo?.smsVerify) {
        return navigateTo('/user/smsVerify')
      }
    }
  }
)
const pledgeSocketStore = usePledgeSocketStore()

await onMounted(async () => {
  // 判斷gameType 是否有 5 pledge type
  if (siteStore.siteData?.gameType?.find((item: any) => item === 5)) {
    await pledgeSocketStore.startConnectWebSocket()
  }
})
onBeforeUnmount(() => {
  // 判斷gameType 是否有 5 pledge type
  if (siteStore.siteData?.gameType?.find((item: any) => item === 5)) {
    pledgeSocketStore.closeWebSocket()
  }
})
const walletName = (type) => {
  switch (type) {
    case 1:
      return siteStore.siteData.mainWalletType === type
        ? t('我的資產(電子錢包)')
        : t('美元資產(電子錢包)')
    case 2:
      return siteStore.siteData.mainWalletType === type
        ? t('我的資產(電子錢包)')
        : t('電力資產(電子錢包)')
    case 3:
      return siteStore.siteData.mainWalletType === type
        ? t('我的資產(電子錢包)')
        : t('商城資產(電子錢包)')
    case 4:
      return siteStore.siteData.mainWalletType === type
        ? t('我的資產(電子錢包)')
        : t('質押資產(電子錢包)')
    default:
      return t('其他資產')
  }
}
const news = ref([])

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
  if (path === '/') {
    signout()
    navigateTo(path)
    return
  }
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
console.log(PlayerStore.playerInfo, 'PlayerStore.playerInfo')

const open = ref(false)
const onOpen = () => {
  open.value = true
}
const onClose = () => {
  open.value = false
}
</script>

<template>
  <div class="content">
    <client-only>
      <div class="user-t white" style="background-color: #025189; padding: 9px; line-height: 24px">
        <div class="login-no">
          <span class="log-user">
            <span style="margin-right: 5px">
              <i class="fa-solid fa-user"></i>
            </span>
            Hello,<span style="
                color: rgb(238, 238, 175);
                display: inline-block;
                margin: 0 5px;
              ">{{ PlayerStore?.playerInfo?.username }}</span>
            $ {{ new Intl.NumberFormat('zh-TW').format(playerWalletBalance) }}
          </span>
        </div>
      </div>
      <nav style="margin: 0 clamp(15px, 3.15vw, 60px)">
        <div class="menu-flex">
          <div class="logo" @click="navigateTo('/')">
            <img :src="siteStore?.siteData.logo" alt="" style="max-height: 50px" />
          </div>
          <div class="menu-nav">
            <ul>
              <li v-for="item in filterPledgeList" :key="item.title" class="nav-sty black a-aim"
                @click="handleNavigateTo(item.path)">
                {{ item.title }}
              </li>
            </ul>
            <div style="margin-left: 60px" @click="onOpen">
              <i class="fa-solid fa-bars" style="height: 17.5px; width: 17.5px"></i>
            </div>
          </div>
        </div>
      </nav>
      <router-view></router-view>
      <div class="sidebar" :class="open ? 'active' : ''">
        <div class="ul-sidebar-header">
          <div class="ul-sidebar-header-logo">
            <a>
              <img :src="siteStore?.siteData.logo" style="max-height: 50px" alt="logo" class="logo" />
              <!-- <img src="@/assets/image/index/logo01.png"> -->
            </a>
          </div>
          <button class="ul-sidebar-closer" @click="onClose">
            <i class="fa-solid fa-xmark fa-2xl"></i>
          </button>
        </div>

        <div class="ul-sidebar-header-nav-wrapper">
          <ul>
            <li v-for="item in filterPledgeList" :key="item.title" class="nav-sty black a-aim"
              @click="handleNavigateTo(item.path)">
              {{ item.title }}
            </li>
          </ul>
        </div>

        <div class="pc-jj">
          <div class="ul-sidebar-about d-none d-lg-block" style="margin-top: 1rem">
            <span class="title">量身訂製解決方案</span>
            <p class="mb-0">
              靈活創新的臨床供應解決方案有助於加速您全球各地的臨床研究
            </p>
          </div>
          <div class="ul-sidebar-about d-none d-lg-block">
            <p style="
                font-size: 20px;
                font-weight: bold;
                color: #016ba9;
                margin-bottom: 1rem;
              ">
              全面、經濟有效的支持您的臨床試驗
            </p>
            <p style="font-size: 16px; font-weight: bold; color: #016ba9">
              迄今為止，我們已經參與了5,000多次臨床試驗，擁有的豐富經驗可以幫助您優化臨床試驗設計,
              並保證無論是簡單還是是最複雜的臨床試驗都能找到經濟有效的方案。我們在國際監管環境、需求模擬和預測以及實際操作方面的經驗可以支援可靠的供應並最大程度減少浪費。
            </p>
            <br />
            <p style="font-size: 16px; color: #000">
              我們在臨床供應管理方面的能力包括：
            </p>
            <p style="color: #000">
              ● 全球供應鏈專案生命週期管理，從製造到最終責任制以及銷毀
            </p>
            <p style="color: #000">● 包括預測在內的供應鏈規劃和最佳化服務</p>
            <p style="color: #000">
              ● 與臨床、申辦者和第三方團隊進行協調與協作並制定和監管臨床供應計劃
            </p>
            <p style="color: #000">
              ● 整合IRT(IVR / IWR) 系統，包括需求、規格和監管的共同開發
            </p>
            <p style="color: #000">
              ● 康泰倫特
              Fusion組合和客戶入口網站可實現保密文件共享、即時運單追蹤、報告和供應鏈視覺化
            </p>
            <p style="color: #000">
              ●
              對庫存水準和過期資料進行管理，監管臨床儲存設施、倉庫和臨床中心的庫存
            </p>
            <p style="color: #000">● 冷鏈規範和受管制物質管理的諮詢服務</p>
          </div>
        </div>
        <div v-if="false" class="pc-jj">
          <div class="ul-sidebar-about d-none d-lg-block">
            <span class="title">職員資訊</span>
            <p class="mb-0">
              提供查閱您的職員資料<br />
              ● 職員帳號<br />● 職員名稱<br />● 職員電話
            </p>
          </div>
          <div class="ul-sidebar-about d-none d-lg-block">
            <span class="title">修改密碼</span>
            <p class="mb-0">
              提供修改您的職員密碼<br />
              ● 更換密碼<br />● 更換金鑰密碼
            </p>
          </div>
        </div>
        <div v-if="false" class="pc-jj">
          <div class="ul-sidebar-about d-none d-lg-block">
            <span class="title">薪資申請</span>
            <p class="mb-0">
              提供職員申請薪資<br />
              ● 薪資申請<br />● 銀行帳號提交<br />● 銀行帳號修改
            </p>
          </div>
          <div class="ul-sidebar-about d-none d-lg-block">
            <span class="title">資訊提交後請耐心等待審核</span>
          </div>
        </div>
        <div v-if="false" class="pc-jj">
          <div class="ul-sidebar-about d-none d-lg-block">
            <span class="title">財務中心</span>
            <p class="mb-0">
              提供查閱您的相關紀錄<br />
              ● 財務紀錄<br />● 工作紀錄
            </p>
          </div>
        </div>
        <!-- <div class="ul-sidebar-footer">
        <span class="ul-sidebar-footer-title">線上客服</span>
        <div class="ul-sidebar-footer-social">
          &nbsp;&nbsp;<a href="weblineservice.php"><i class="flaticon-user"></i></a>
        </div>
      </div> -->
      </div>
    </client-only>
  </div>
</template>

<style scoped lang="sass">
// page
//   overflow: auto !important

.content
  background-color: #fff
  width: 100dvw
  height: 100dvh
  // min-height: 100dvh
  overflow-y: auto
  .white
    color: #FFF

  .black
    color: #0C1239

  /* user

  .user-t
    background: #0A0E21
    padding: 12px 0
    font-size: 16px
    margin-bottom: 26px
  .log-size
    /* font-size: 16px;

  .login-off, .login-no
    padding: 0 12px

  .login-off
    font-size: 14px

  .a-aim
    transition: all 0.3s ease 0s

    &:hover
      color: #EF2853

  /* nav

  .menu-flex
    display: flex
    align-items: center
    // flex-wrap: wrap
    justify-content: space-between
    margin-bottom: 20px
  .logo
    cursor: pointer
    // img
      // width: clamp(115px, 8.93vw, 100px)
      // min-width: 205px
  .menu-nav
    /* flex-grow: .3;
    display: flex
    align-items: center
    ul
      /* margin: 0 0 0 300px;
      display: flex
      flex-direction: row
      // flex-wrap: wrap
      padding: 0 0
      align-items: center
      gap: clamp(20px, 2.1vw, 40px)
      li
        font-size: 16px
        font-weight: 600
        padding: 12px 0
        display: block
        line-height: 1
        cursor: pointer
        white-space: nowrap
        @media screen and (max-width: 991px)
          display: none
  .ul-sidebar-header-nav-wrapper
    display: none
  @media screen and (max-width: 991px)
    .ul-sidebar-header-nav-wrapper
      display: block
      margin-top: 15px
      border-top: 1px solid rgba(0, 0, 0, 0.2)
      border-left: 1px solid rgba(0, 0, 0, 0.2)
      border-right: 1px solid rgba(0, 0, 0, 0.2)
      li
        line-height: 1.2
        padding: 12px 15px 10px
        border-bottom: 1px solid rgba(0, 0, 0, 0.2)
    .pc-jj
      display: none
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
    padding: clamp(20px, 1.58vw, 30px) clamp(20px, 1.58vw, 30px)
    max-width: 100%
    border-left: 1px solid  #025189
    display: flex
    flex-direction: column
    // justify-content: space-between
    width: clamp(320px, 40vw, 100%)
    @media screen and (max-width: 991px)
      justify-content: start
  .active
    transform: translateX(0) !important


  .ul-sidebar-header
    display: flex
    align-items: flex-end
    justify-content: space-between
    border-bottom: 1px solid rgba(0, 0, 0, 0.2)
    padding-bottom: clamp(20px, 1.58vw, 30px)
  // .logo
    // width: clamp(115px, 8.93vw, 170px)

  .ul-sidebar-about .title
    font-weight: 600
    font-size: 24px
    margin-bottom: clamp(7px, 0.53vw, 10px)
  p
    line-height: 1.75
  .ul-sidebar-about p
    margin-bottom:1rem
</style>
