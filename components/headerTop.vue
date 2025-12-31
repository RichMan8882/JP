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

              <img src="@/assets/image/home1/mimg.png" alt="" style="width: auto;">
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
      // background: #ff6c7a !important
      color: #fff !important
      height: 49px
      padding: 0 !important
      border: none !important
      box-shadow: 0 4px 5px rgba(0, 0, 0, 0.2)
      img
        border-radius: 10px
        width: auto !important
        height: 100% !important
        transition: all .3s
        &:hover
          transform: scale(1.05)
          
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
