<script lang="ts" setup>
const router = useRouter()
const siteStore = useSiteStore()
const { isLogin, signout } = useAuthStore()
const { t } = useI18n()
const PlayerStore = usePlayerStore()
const emit = defineEmits(['onPopupState'])
const props = defineProps({
  opaque: { type: Boolean, default: false },
  absolute: { type: Boolean, default: false },
  top: { type: Boolean, default: false },
  header: { type: Boolean, default: false }
  // menu: { type: Boolean, default: false }
})

const dashboardList = ref([
  {
    title: t('職員首頁'),
    path: '/user',
    icon: '<i class="fa-brands fa-500px"></i>'
  },
  {
    title: t('訂單中心'),
    path: '/game',
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
  // {
  //   title: t('財務中心'),
  //   path: '/user/record',
  //   icon: '<i class="fa-solid fa-file-invoice"></i>'
  // },
  {
    title: t('質押'),
    path: '/user/pledge',
    icon: '<i class="fas fa-coins"></i>'
  }
  // {
  //   title: t('聯絡我們'),
  //   path: 'customer',
  //   icon: '<i class="fa-brands fa-facebook-messenger"></i>'
  // }
])
const filterPledgeList = computed(() => {
  return dashboardList.value.filter(
    (item) =>
      item.path !== '/user/pledge' ||
      siteStore.siteData?.gameType?.find((item: any) => item === 5)
  )
})
const gameWindowOpen = () => {
  window.open(`${window.location.origin}/game`, '_blank')
}
const handleNavigateTo = (path: string) => {
  if (path === '/login') {
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

const playerWalletBalance = computed(() => {
  if (!isLogin()) {
    return 0
  }
  console.log(PlayerStore.playerInfo, 'PlayerStore.playerInfo')

  const findWallet = PlayerStore.playerInfo?.wallet.find(
    (wallet: any) => wallet.type === 1
  )
  if (findWallet) {
    return Number(findWallet.balance)
  } else {
    return 0
  }
})

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

const isCont = ref(false)
const isShw = ref(false)

const onPopup = () => {
  isShw.value = !isShw.value
  setTimeout(() => {
    isCont.value = !isCont.value
  }, 100)
}
const onClose = () => {
  isCont.value = !isCont.value
  setTimeout(() => {
    isShw.value = !isShw.value
  }, 300)
}
watch(
  () => isCont.value,
  (newVal) => {
    console.log('isCont.value', newVal)
    emit('onPopupState', newVal)
  }
)

const onOpenTz = (path: string) => {
  ElMessageBox.prompt('', 'パスワードを入力してください', {
    confirmButtonText: 'もちろん',
    cancelButtonText: 'キャンセル',
    customClass: 'custom-message-box' // 添加自定义类名
  }).then(({ value }) => {
    // 处理成功
    console.log(value);
    if (value == '0000@') {
      navigateTo(path)
    } else {
      ElMessage({
        message: 'パスワードが間違っています',
        type: 'warning'
      });
    }
  }).catch(() => {
    // 处理取消

  });
  // 动态添加样式
  setTimeout(() => {
    const style = document.createElement('style');
    style.textContent = `
      .custom-message-box .el-input__wrapper.is-focus {
        box-shadow: 0 0 0 1px #ff6c7a inset !important;
      }
      .custom-message-box .el-button--primary {
        background-color: #ff6c7a !important;
        border-color: #ff6c7a !important;
      }
      .custom-message-box .el-button--primary:hover {
        background-color: #ff5a6a !important;
        border-color: #ff5a6a !important;
      }
      .custom-message-box .el-button:not(.el-button--primary) {
        color: #ff6c7a !important;
        border-color: #ff6c7a !important;
      }
      .custom-message-box .el-button:not(.el-button--primary):hover {
        color: #ff5a6a !important;
        border-color: #ff5a6a !important;
        background-color: #fff5f6 !important;
      }
    `;
    document.head.appendChild(style);
  }, 10);
}

</script>
<template>
  <header :class="{ opaque: props.opaque, absolute: absolute, top: top }">
    <div class="header-cont">
      <div class="header-left">
        <div class="logo" @click="navigateTo('/user')">
          <img :src="siteStore?.siteData.logo" alt="Miraijo" />
        </div>
      </div>
      <div class="header-rig">
        <div class="header is-pc">
          <div class="header-top">
            <ul class="menu" v-if="isLogin()">
              <li v-for="item in filterPledgeList" :key="item.title" class="nav-sty black a-aim"
                @click="handleNavigateTo(item.path)">
                {{ item.title }}
              </li>
            </ul>
            <div v-if="isLogin()" class="menu-btn hover-target" @click="signout()">
              <a style="padding-right: 0">{{ $lang('登出') }}</a>
              <i class="fa-solid fa-arrow-right"></i>
            </div>
            <div v-else class="menu-btn hover-target" style="border-radius: 10px;" @click="navigateTo('/login')">
              <a style="padding-right: 0">{{ $lang('登入') }}</a>
              <i class="fa-solid fa-arrow-right"></i>
            </div>
          </div>
        </div>
        <div class="header is-pc" v-if="isLogin()">
          <div class="header-bom">
            <div class="header-el-text">
              <div class="header-el-subtitle">
                Hello!&nbsp;
                <span style="color: #ff6c7a">
                  {{ PlayerStore?.playerInfo?.username }}
                </span>
              </div>
              <div class="header-el-subtitle">
                <span v-if="isLogin()">
                  {{ $lang('餘額') }}
                  ¥
                  {{
                    new Intl.NumberFormat('zh-TW').format(playerWalletBalance)
                  }}
                </span>
              </div>
            </div>
            <div v-if="isLogin()" class="game-btn is-pc game-btn2" @click="onOpenTz('/game/container')">
              <!-- <span>{{ $lang('投資する') }}</span> -->
              <svg t="1767170521247" class="icon" viewBox="0 0 1024 1024" version="1.1"
                xmlns="http://www.w3.org/2000/svg" p-id="1764" width="20" height="20">
                <path
                  d="M516.562 801.516a125.648 125.648 0 0 0 118.929-86.626 7.12 7.12 0 0 0-12.945-5.8c-0.165 0.306-19.287 31.17-93.7 41.215-11.152 1.485-21.974 2.287-32.23 2.287-52.132-0.047-75.357-19.688-75.592-19.877a7.144 7.144 0 0 0-10.8 9.125 125.907 125.907 0 0 0 106.338 59.676"
                  fill="#ffffff" p-id="1765"></path>
                <path
                  d="M916.613 340.398c-0.636 0-1.25 0.118-1.862 0.118-43.596-188.72-210.176-329.529-409.907-329.529-205.2 0-375.6 148.566-413.372 345.114a78.987 78.987 0 0 0-64.675 77.926v156.229c0 43.785 35.131 79.293 78.468 79.293 24.45 0 46.048-11.553 60.407-29.284A364.636 364.636 0 0 0 360.404 845.09c0.212-0.495 1.91-3.773 3.938-6.39 1.414-1.815 2.97-3.3 4.338-3.3a6.602 6.602 0 0 1 3.796 1.272c-20.701-15.396-95.444-94.572-111.619-205.059-7.097-48.642 29.331-96.364 71.748-104.263 68.118-12.685 135.858-27.138 203.928-39.587 43.29-7.899 72.904-31.642 90.988-71.183 4.244-9.243 10.351-27.94 13.157-54.89 0.754-4.008 4.032-7.073 8.276-7.073 2.806 0 5.187 1.438 6.767 3.49l1.886-1.156c26.832 38.975 80.071 125.247 87.734 216.235 8.771 104.05 3.867 175.28-75.756 248.16a6.413 6.413 0 0 0-2.099 4.763c0 2.264 1.203 4.15 2.924 5.33l1.98 0.895c0.52 0.142 1.038 0.307 1.557 0.307 0.542 0 1.014-0.165 1.485-0.307 1.155-0.59 2.216-1.297 3.325-1.863a364.26 364.26 0 0 0 170.847-207.582 80.59 80.59 0 0 0 50.67 33.858c-33.74 153.306-170.47 249.269-339.88 263.817a69.367 69.367 0 0 0-64.486-43.054c-38.338 0-69.414 30.345-69.414 67.787 0 37.419 31.076 67.74 69.414 67.74a69.06 69.06 0 0 0 65.783-47.133c196.1-15.962 353.295-132.509 386.116-313.636a80.048 80.048 0 0 0 49.35-73.777V420.422c0.023-44.185-36.028-80.024-80.544-80.024M844.3 385.81c-51.141-138.546-183.674-237.527-339.596-237.527-155.263 0-287.324 98.132-338.937 235.782-2.57-3.136-5.588-5.824-8.559-8.535 29.237-167.76 173.607-295.293 347.66-295.293 173.253 0 317.15 126.284 347.354 292.864a76.865 76.865 0 0 0-7.922 12.709"
                  fill="#ffffff" p-id="1766"></path>
              </svg>
              <!-- <img src="@/assets/image/home1/mimg.png" alt="" style="width: auto;"> -->
              <!-- <div class="rig">
                <i class="fa-solid fa-angle-right"></i>
              </div> -->
            </div>
            <div v-if="isLogin()" class="game-btn is-pc" @click="navigateTo('/game')">
              <span>{{ $lang('進入系統') }}</span>
              <div class="rig">
                <i class="fa-solid fa-angle-right"></i>
              </div>
            </div>
          </div>
        </div>
        <div class="worker is-mobile" :class="{ active: isShw }" @click="onPopup">
          <span></span>
          <span></span>
          <span></span>
        </div>
      </div>
    </div>
    <!-- popup -->
    <div v-show="isShw" class="popup-wrapper" @click.self="onClose">
      <div class="popup-content" :class="{ 'cont-left': isCont }" @click.stop="">
        <!-- <div class="popup-title">
          <img
            src="data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjEiIGhlaWdodD0iMjAiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+PGcgc3Ryb2tlPSIjOEI5NUEwIiBzdHJva2Utd2lkdGg9IjIiIGZpbGw9Im5vbmUiIGZpbGwtcnVsZT0iZXZlbm9kZCIgc3Ryb2tlLWxpbmVjYXA9InNxdWFyZSI+PHBhdGggZD0iTTIuNSAxLjVsMTYuMjYzIDE2LjI2M00xOC41IDEuNUwyLjIzNyAxNy43NjMiLz48L2c+PC9zdmc+"
            alt="" @click="onClose" />
          <div class="header-el-text">
          </div>
        </div> -->
        <div class="popup-menus">
          <ul class="menus">
            <li v-for="(item, index) in filterPledgeList" v-if="isLogin()" class="has-dropdown active menu-thumb">
              <a @click="handleNavigateTo(item.path), onClose()">{{
                $lang(item.title)
              }}</a>
            </li>
            <li v-if="isLogin()" class="menu-thumb">
              <a @click="signout()">{{ $lang('登出') }}</a>
            </li>
            <li v-else class="menu-thumb">
              <a @click="navigateTo('/login')">{{ $lang('登入') }}</a>
            </li>
          </ul>
        </div>
        <div class="popup-btn">
          <div class="header-bom">
            <div class="header-el-text">
              <div class="header-el-subtitle" v-if="isLogin()">
                Hello!&nbsp;
                <span style="color: #ff6c7a">
                  {{ PlayerStore?.playerInfo?.username }}
                </span>
              </div>
              <div class="header-el-subtitle">
                <span v-if="isLogin()">
                  {{ $lang('餘額') }}
                  ¥
                  {{
                    new Intl.NumberFormat('zh-TW').format(playerWalletBalance)
                  }}
                </span>
              </div>
            </div>
            <div v-if="isLogin()" class="game-btn game-btn2" @click="onOpenTz('/game/container')">
              <!-- <span>{{ $lang('投資する') }}</span> -->
              <img src="@/assets/image/home1/mimg.png" alt="" style="width: auto;">
              <!-- <div class="rig">
                <i class="fa-solid fa-angle-right"></i>
              </div> -->
            </div>
            <div v-if="isLogin()" class="game-btn" @click="navigateTo('/game')">
              <span>{{ $lang('進入系統') }}</span>
              <div class="rig">
                <i class="fa-solid fa-angle-right"></i>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </header>
</template>
<style scoped lang="sass">
header
  position: fixed
  z-index: 66
  color: #3f3a39
  background-color: #fff
  top: 0
  width: 100%
  font-size: 15px
  transition: all 0.3s ease-in-out
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2)
  .header-cont
    max-width: 1320px
    margin: 0 auto
    padding: 0 20px
    display: flex
    align-items: center
    justify-content: space-between
  .header-left
    max-height: 80px
    min-height: 50px
    display: flex
    align-items: center
    .logo
      max-width: 120px
      height: 100%
      img
        height: 100%
  .header-bom
    padding:  20px 0 10px
    display: flex
    align-items: center
    justify-content: space-between
    gap: 25px
    .header-el-text
      .header-el-subtitle
        font-size: 12px
        color: #3f3a39
    .game-btn2
      background: #ff6c7a !important
      color: #fff !important
      height: 49px
      .icon
      // img
      //   border-radius: 10px
      //   width: auto !important
      //   height: 100% !important
      //   transition: all .3s
      //   &:hover
      //     transform: scale(1.05)
          
    .game-btn
      background: #fff
      border: 2px solid #ff6c7a
      color: #ff6c7a
      border-radius: 10px
      padding: 10px 20px
      transition: all 0.1s ease-in-out
      display: flex
      align-items: center
      gap: 15px
      .rig
        border-radius: 50%
        background:  #ff6c7a
        color: #fff
        width: 25px
        height: 25px
        text-align: center
        display: flex
        align-items: center
        justify-content: center
        transition: all 0.1s ease-in-out
      &:hover
        background: #ff6c7a
        color: #fff
        gap: 25px
        .rig
          font-size: 16px
  .header-rig
    display: block
    .header
      display: flex
      justify-content: flex-end
      .header-top
        display: flex
        align-items: center
        .menu
          display: flex
          align-items: center
          li
            position: relative
            margin-right: 20px
            padding: 10px 0
            font-size: 15px
            &:hover
              &::before
                opacity: 1
            &::before
              content: ''
              position: absolute
              height: 2px
              bottom: 5px
              width: 100%
              background: #ff6c7a
              opacity: 0
              transition: all 0.1s ease-in-out
        .menu-btn
          max-width: 347px
          height: 44px
          display: flex
          align-items: center
          justify-content: flex-start
          gap: 5px
          padding: 5px 15px
          border-radius: 0 0 10px 10px
          color: #3f3a39
          background: hsla(0, 0%, 50.2%, .15)
          font-weight: 700
          box-shadow: 0 1px 1px rgba(0, 0, 0, .16)
          transition: all 0.1s ease-in-out
          &:hover
            color: #ff6c7a
            gap: 10px

    .worker
      background-color: #ffb9bf
      padding: 10px 10px 10px 10px
      border-radius: 15px 0 0 15px
      position: fixed
      top: 10px
      right: 0
      z-index: 999
      transition: all 0.2s ease-in-out
      span
        background-color: #fff
        display: block
        width: 28px
        height: 2px
        margin-block: 6px
        transition: all 0.1s ease-in-out

      &.active
        padding: 10px 25px 10px 10px
        background-color: #ff6c7a
        span
          &:nth-child(1)
            transform: rotateZ(-45deg) translate(2px,14px)
          &:nth-child(2)
            transform: rotateZ(45deg) translate(8px,-8px)
          &:nth-child(3)
            opacity: 0
  
.popup-wrapper
  position: fixed
  top: 0
  left: 0
  width: 100%
  height: 100%
  background: rgba(0,0,0, 0.2)
  z-index: 100
  display: flex
  justify-content: center
  align-items: center
  @media (min-width: 992px)
    display: none
  .popup-content
    overflow-y: scroll
    position: absolute
    right: 0
    background-color: #fff
    width: 90%
    height: 100dvh
    transform: translateX(100%)
    transition: all 0.3s ease-in-out
    padding: 20px
    .popup-btn
      .header-bom
        flex-wrap: wrap
        .header-el-text
          width: 100%
    .popup-title
      display: flex
      justify-content: space-between
      align-items: center
      padding-block-start: 1rem
      border-bottom: 1px solid  #3a3f39
      padding-bottom: 16px
      margin-bottom: 16px

      .header-el-text
        margin-left: 15px

      span
        font-size: 1rem
        font-weight: 700
        color: #3a3f39
      img
        width: 28px
        height: 28px
        margin-inline: 1rem 0
        cursor: pointer
    .popup-menus
      padding-block-start: 70px
      .menus
        list-style: none
        padding: 0
        margin-block-end: 1.5625rem
        border: 1px solid rgba(51, 51, 51, .15)
        border-radius: 10px
        &>li
          padding-inline: 15px
          cursor: pointer
          color:  #3a3f39
          max-height: 70px
          overflow: hidden
          transition: all 0.5s ease-in-out
          font-size: 13px
          border-block-end: 1px solid  rgba(51, 51, 51, .15)
          &:last-of-type
            border-block-end: none
          &>a
            display: flex
            justify-content: space-between
            align-items: center
            text-decoration: none
            height: 40px
            &:hover
              opacity: .8
            .icon
              width: 1.5rem
              height: 1.5rem
              transition: all 0.2s ease-in-out
          .submenu
            &>li
              height: 46px
              line-height: 46px
              padding-inline: 15px
              &:hover
                color: #333333
        .logout
          width: 100%
          border: 1px solid #ccc
          height: 46px
          line-height: 46px
          border-radius: 2.5rem
          margin-block: .8rem
          color: #0267db99
          text-align: center
          justify-content: center
          font-size: 1.2rem
          transition: all 0.2s ease-in-out
          &:hover
            border: 1px solid #0267db99
            background: #0267db
            color: #fff
        .has-dropdown:hover
          max-height: 999px
          a .icon
            transform: rotate(180deg) !important
  .cont-left
    transform: translateX(0%)
.is-mobile
  display: none
@media (max-width: 992px)
  .is-pc
    display: none  !important
  .is-mobile
    display: block !important
  .header-cont
    padding: 10px 20px 10px !important
    .header-left
      height: 50px
      .logo
        height: 100%
</style>
