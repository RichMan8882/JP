<script lang="ts" setup>
import { Swiper, SwiperSlide } from 'swiper/vue'
import { Navigation, Pagination, Scrollbar, A11y } from 'swiper/modules'
import 'swiper/css'
import 'swiper/css/navigation'
import 'swiper/css/pagination'
import 'swiper/css/scrollbar'
const { isLogin } = useAuthStore()
import cls1 from '@/assets/image/index/cls1.svg'
import cls2 from '@/assets/image/index/cls2.svg'
import cls3 from '@/assets/image/index/cls3.svg'
import cls4 from '@/assets/image/index/cls4.svg'
import cls5 from '@/assets/image/index/cls5.svg'
import cls6 from '@/assets/image/index/cls6.svg'
const PlayerStore = usePlayerStore()
const siteStore = useSiteStore()
const { signout } = useAuthStore()
const router = useRouter()
const homeRouter = ref('/user')
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
    homeRouter.value = '/user'
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

await useAsyncData(async () => {
  const path = 'user'
  const queryNewsRes = await queryNews(lang, path)
  news.value = queryNewsRes.data.result
  console.log('queryNewsRes.data', queryNewsRes.data)
})

const activityLis = ref([
  {
    title: t('着物モデルさん大募集'),
    src: 'https://upload.comethike.com/uploads/1759832603922.png',
    link: 'https://www.youtube.com/watch?v=dQw4w9WgXcQ',
    location:
      '撮影はモデルさんのお住まいの地域（所在地）で行っていただきますので、遠方への移動は不要です。',
    name: '募集人数5名',
    salary: '日円：25000円(含稅)',
    information: [
      '【種類】撮影テーマ  ポーズ例',
      '【振袖】華やか・青春・成人式の喜び  巾着を持って優雅に歩く、袖を広げて見せる、振り返りながら袖に手を添える',
      '【訪問着】上品・社交的・エレガント  玄関で軽くノックする、手土産を持って微笑む、正座してお茶を差し出す',
      '【浴衣】夏祭り・青春・花火  団扇やかき氷を手に、花火を見上げる、夏の小道を歩きながら振り返る',
      '【留袖】端正・成熟した女性の美しさ  礼をする、家族と並んで立つ、膝に手を添えて端正に座る',
      '【花嫁衣裳】伝統婚礼・幸福感  花を持ってうつむき微笑む、新郎と並んで座る、赤い和傘の下で見つめ合う',
      '【モダン着物】伝統×現代・和洋ミックス  和風の街並みでコーヒーを持つ、街角でおしゃれに立つ、振り返りながら可愛く微笑む'
    ]
  },
  {
    title: t('自分だけの美しい瞬間を残してみませんか？'),
    src: 'https://upload.comethike.com/uploads/1766408747881.jpg',
    link: '',
    location:
      '',
    name: '',
    salary: '',
    information: [
      'ただいま、漢服・着物・ウェディングドレスの撮影モデルを募集しています。',
      '一度の撮影で、全く異なる三つの文化・スタイルの美しさを体験できます🌸💍',
      '• 優雅で華麗な 漢服',
      '• 落ち着きと柔らかさを感じる 着物',
      '• 夢のように華やかでロマンチックな ウェディングドレス',
      'どの雰囲気がお好きな方でも、',
      '「もっと綺麗になりたい」「新しい自分を試してみたい」',
      'そんな気持ちがあれば大歓迎です💖',
      '撮影中は、カメラマンがポーズを丁寧にサポートしますので、',
      '未経験の方でも安心して参加いただけます✨',
      '「素敵な一枚を残したい」という気持ちさえあれば十分です。',
      'カメラを通して、あなたの一番輝く姿を一緒に記録しましょう。',
      '素敵な作品を一緒に作れる日を、心より楽しみにしています！'
    ]
  },
  {
    title: t('🎄🤍 クリスマスの小さなプレゼント 🤍🎄'),
    src: 'https://upload.comethike.com/uploads/1766409232824.jpg',
    link: '',
    location: '',
    name: '',
    salary: '',
    information: [
      'この子どもマフラー、',
      'この投稿にコメントしてくれたママさんへお渡しします 🧣✨',
      '✔️ コメントするだけ',
      '✔️ 数量制限なし',
      'お子さんの年齢や、',
      '「可愛いと思った理由」だけでもOKです🤍',
      '気軽にコメントしてください🫶'
    ]
  }
])

const clsLis = ref([
  {
    title: t('在宅で働く'),
    name: t('自分に合ったペースで！'),
    src: cls1,
    link: '',
    location: ''
  },
  {
    title: t('さまざまな無料講座で'),
    name: t('自分をさらに磨ける'),
    src: cls2,
    link: '',
    location: ''
  },
  {
    title: t('子どもとの時間を増やす'),
    name: t('少なめ'),
    src: cls3,
    link: '',
    location: ''
  },
  {
    title: t('時間に縛られず'),
    name: t('あなたのタイミングで働く'),
    src: cls4,
    link: '',
    location: ''
  },
  {
    title: t('経歴'),
    name: t('資格などスキルを活かす'),
    src: cls5,
    link: '',
    location: ''
  },
  {
    title: t('専業主婦'),
    name: t('専業主婦の方もOK ブランクOK'),
    src: cls6,
    link: '',
    location: ''
  }
])
</script>
<template>
  <div class="dashboard" :style="isLogin() ? '' : 'padding-top: 63px !important'">
    <headerTop />
    <div class="dashboard-video">
      <video autoplay muted loop poster="@/assets/image/index/shutterstock.jpg" class="video-element"
        webkit-playsinline="true" playsinline="true" x5-video-player-type="h5" x5-video-player-fullscreen="false">
        <source src="https://upload.comethike.com/uploads/1759564877962.mov" type="video/mp4" />
      </video>
    </div>
    <div v-if="false" style="margin: 0 clamp(15px, 3.15vw, 60px); padding-bottom: 10px">
      <div class="lis-cont">
        <ul>
          <li>
            <div class="li-box" @click="handleNavigateTo('game')">
              <svg xmlns="http://www.w3.org/2000/svg" width="45" height="45" viewBox="0 0 45 45" class="svg-color">
                <g id="レイヤー_2" data-name="レイヤー 2" transform="translate(0.491 -0.297)">
                  <!-- <g id="Rectangle" transform="translate(0 0)">
                    <rect id="長方形_2537" data-name="長方形 2537" width="45" height="45" transform="translate(-0.491 0.297)"
                      fill="none" />
                  </g> -->
                  <g id="icon_data" transform="translate(5.109 8.824)">
                    <path id="パス_1510" data-name="パス 1510" d="M10,26.631V9H35.982V21.063"
                      transform="translate(-5.36 -9)" fill="none" stroke-linecap="round" stroke-linejoin="round"
                      stroke-width="2" />
                    <path id="パス_1511" data-name="パス 1511" d="M35.622,28l4.64,7.423H5L9.64,28Z"
                      transform="translate(-5 -10.369)" fill="none" stroke-linecap="round" stroke-linejoin="round"
                      stroke-width="2" />
                  </g>
                </g>
              </svg>
              <div class="text">訂單中心</div>
              <div class="ico">
                <i class="fa-solid fa-angle-right" style="width: 20px; height: 20px"></i>
              </div>
            </div>
          </li>
          <li>
            <div class="li-box" @click="handleNavigateTo('/user/info')">
              <svg xmlns="http://www.w3.org/2000/svg" width="45" height="45" viewBox="0 0 45 45" class="svg-color">
                <g id="レイヤー_2" data-name="レイヤー 2" transform="translate(0.491 -0.297)">
                  <!-- <g id="Rectangle" transform="translate(0 0)">
                    <rect id="長方形_2537" data-name="長方形 2537" width="45" height="45" transform="translate(-0.491 0.297)"
                      fill="none" />
                  </g> -->
                  <g id="icon_data" transform="translate(5.109 8.824)">
                    <path id="パス_1510" data-name="パス 1510" d="M10,26.631V9H35.982V21.063"
                      transform="translate(-5.36 -9)" fill="none" stroke-linecap="round" stroke-linejoin="round"
                      stroke-width="2" />
                    <path id="パス_1511" data-name="パス 1511" d="M35.622,28l4.64,7.423H5L9.64,28Z"
                      transform="translate(-5 -10.369)" fill="none" stroke-linecap="round" stroke-linejoin="round"
                      stroke-width="2" />
                  </g>
                </g>
              </svg>
              <div class="text">職員資料</div>
              <div class="ico">
                <i class="fa-solid fa-angle-right" style="width: 20px; height: 20px"></i>
              </div>
            </div>
          </li>
          <li>
            <div class="li-box" @click="handleNavigateTo('/user/bank')">
              <svg xmlns="http://www.w3.org/2000/svg" width="45" height="45" viewBox="0 0 45 45" class="svg-color">
                <g id="レイヤー_2" data-name="レイヤー 2" transform="translate(0.491 -0.297)">
                  <!-- <g id="Rectangle" transform="translate(0 0)">
                    <rect id="長方形_2537" data-name="長方形 2537" width="45" height="45" transform="translate(-0.491 0.297)"
                      fill="none" />
                  </g> -->
                  <g id="icon_data" transform="translate(5.109 8.824)">
                    <path id="パス_1510" data-name="パス 1510" d="M10,26.631V9H35.982V21.063"
                      transform="translate(-5.36 -9)" fill="none" stroke-linecap="round" stroke-linejoin="round"
                      stroke-width="2" />
                    <path id="パス_1511" data-name="パス 1511" d="M35.622,28l4.64,7.423H5L9.64,28Z"
                      transform="translate(-5 -10.369)" fill="none" stroke-linecap="round" stroke-linejoin="round"
                      stroke-width="2" />
                  </g>
                </g>
              </svg>
              <div class="text">{{ t('薪資專區') }}</div>
              <div class="ico">
                <i class="fa-solid fa-angle-right" style="width: 20px; height: 20px"></i>
              </div>
            </div>
          </li>
          <li>
            <div class="li-box" @click="handleNavigateTo('/user/record')">
              <svg xmlns="http://www.w3.org/2000/svg" width="45" height="45" viewBox="0 0 45 45" class="svg-color">
                <g id="レイヤー_2" data-name="レイヤー 2" transform="translate(0.491 -0.297)">
                  <!-- <g id="Rectangle" transform="translate(0 0)">
                    <rect id="長方形_2537" data-name="長方形 2537" width="45" height="45" transform="translate(-0.491 0.297)"
                      fill="none" />
                  </g> -->
                  <g id="icon_data" transform="translate(5.109 8.824)">
                    <path id="パス_1510" data-name="パス 1510" d="M10,26.631V9H35.982V21.063"
                      transform="translate(-5.36 -9)" fill="none" stroke-linecap="round" stroke-linejoin="round"
                      stroke-width="2" />
                    <path id="パス_1511" data-name="パス 1511" d="M35.622,28l4.64,7.423H5L9.64,28Z"
                      transform="translate(-5 -10.369)" fill="none" stroke-linecap="round" stroke-linejoin="round"
                      stroke-width="2" />
                  </g>
                </g>
              </svg>
              <div class="text">財務中心</div>
              <div class="ico">
                <i class="fa-solid fa-angle-right" style="width: 20px; height: 20px"></i>
              </div>
            </div>
          </li>
        </ul>
      </div>
    </div>
    <div class="dashboard-banner">
      <div class="bannerBox">
        <div class="bnr-img">
          <div class="bnr-t">
            <h2>
              <span style="color: #ff6c7a"> ここでは、 </span>
              <br />すべての女性が自分らしい輝きを咲かせられます。
            </h2>
            <h3 style="text-align: right">
              ただの仕事探しのプラットフォームではなく、<br /><span style="color: #ff6c7a; font-weight: 500">あなたの夢が叶う</span>
            </h3>
          </div>
          <img src="https://upload.comethike.com/uploads/1759567370550.jpg" alt="" />
        </div>
        <div class="bnr-txt">
          <p>
            私たちは、女性のための多様なマッチングプラットフォームです。このサイトのコンセプトは、「女性が自分自身の成長と変化を通して、より良い自分へと進化し、理想や夢を実現できる場所になること」です。
          </p>
          <p>
            ここは、女性のためだけに作られた特別な空間で、仕事の機会を提供するだけでなく、自分を磨くためのさまざまなイベントや講座もご用意しています。
          </p>
          <p>自分らしく輝き、夢を叶える第一歩をここから踏み出しましょう。</p>
        </div>
      </div>
    </div>
    <div class="dashboard-lis">
      <div class="dashboard-lis-banner">
        <img src="@/assets/image/index/banner_pc.webp" alt="" />
      </div>
      <div class="dashboard-lis-title">
        <div class="dashboard-lis-title-txt">
          <svg xmlns="http://www.w3.org/2000/svg" width="5" height="39.5" viewBox="0 0 5 39.5">
            <g id="グループ_1103" data-name="グループ 1103" transform="translate(-473.5 -1804.5)">
              <rect id="長方形_2437" data-name="長方形 2437" width="20" height="5"
                transform="translate(473.5 1844) rotate(-90)" fill="#ff6c7a" />
              <rect id="長方形_2438" data-name="長方形 2438" width="20" height="5"
                transform="translate(473.5 1824.5) rotate(-90)" fill="#fbb" />
            </g>
          </svg>
          <h2 class="dashboard-lis-title-title">{{ $lang('當前活動') }}</h2>
        </div>
        <p class="dashboard-lis-title-txt">
          {{ $lang('すべての挑戦が、自分を咲かせる旅になる。') }}
        </p>
      </div>
      <div class="dashboard-lis-list">
        <ul class="dashboard-lis-list-ul">
          <li v-for="(item, index) in activityLis" :key="index">
            <div class="li-box">
              <figure class="img">
                <img :src="item.src" alt="" />
              </figure>
              <h5 class="txt">
                <p>
                  {{ item.title }}
                </p>
              </h5>
              <div class="text">
                <h4 v-if="item.name != ''">
                  <svg class="svg-color" xmlns="http://www.w3.org/2000/svg" width="23" height="23" viewBox="0 0 23 23">
                    <g id="グループ_1646" data-name="グループ 1646" transform="translate(-213 -3241)">
                      <circle id="楕円形_36" data-name="楕円形 36" cx="11.5" cy="11.5" r="11.5"
                        transform="translate(213 3241)" fill="#3f3a39" />
                      <g id="ノートパソコンのアイコン素材4" transform="translate(217 3183.844)">
                        <path id="パス_1430" data-name="パス 1430"
                          d="M35.864,70.928h10.7a.739.739,0,0,0,.739-.739V63.895a.739.739,0,0,0-.739-.739h-10.7a.739.739,0,0,0-.739.739v6.294A.739.739,0,0,0,35.864,70.928Zm.391-6.779h9.914v5.612H36.255Z"
                          transform="translate(-34.157 0)" fill="#fff" />
                        <path id="パス_1431" data-name="パス 1431"
                          d="M14,374.162l-1.318-1.17a.509.509,0,0,0-.335-.118H1.763a.509.509,0,0,0-.335.118l-1.318,1.17a.3.3,0,0,0-.109.226v.408a.2.2,0,0,0,.222.172H13.887a.2.2,0,0,0,.222-.172v-.408A.3.3,0,0,0,14,374.162Zm-8.457.138.382-.516H8.342l.381.516Z"
                          transform="translate(0 -301.183)" fill="#fff" />
                      </g>
                    </g>
                  </svg>
                  <span>
                    {{ item.name }}
                  </span>
                </h4>
                <h4 v-if="item.location != ''">
                  <svg class="svg-color" xmlns="http://www.w3.org/2000/svg" width="23" height="23" viewBox="0 0 23 23">
                    <g id="グループ_1647" data-name="グループ 1647" transform="translate(-213 -3275)">
                      <circle id="楕円形_35" data-name="楕円形 35" cx="11.5" cy="11.5" r="11.5"
                        transform="translate(213 3275)" fill="#3f3a39" />
                      <g id="ビルのアイコン" transform="translate(124 3279.791)">
                        <path id="パス_1428" data-name="パス 1428"
                          d="M96,0V13.723h3.629V11.579h1.814v2.144h3.629V0Zm2.495,8.72H97.134V7h1.361Zm0-3H97.134V4h1.361Zm0-3H97.134V1h1.361Zm2.721,6H99.855V7h1.361Zm0-3H99.855V4h1.361Zm0-3H99.855V1h1.361Zm2.721,6h-1.361V7h1.361Zm0-3h-1.361V4h1.361Zm0-3h-1.361V1h1.361Z"
                          fill="#fff" />
                      </g>
                    </g>
                  </svg>
                  <span>
                    {{ item.location }}
                  </span>
                </h4>
                <h4 v-if="item.salary != ''">
                  <svg class="svg-color" xmlns="http://www.w3.org/2000/svg" width="23" height="23" viewBox="0 0 23 23">
                    <g id="グループ_1648" data-name="グループ 1648" transform="translate(-213 -3309)">
                      <circle id="楕円形_37" data-name="楕円形 37" cx="11.5" cy="11.5" r="11.5"
                        transform="translate(213 3309)" fill="#3f3a39" />
                      <g id="円マークアイコン" transform="translate(170.547 3314)">
                        <path id="パス_1427" data-name="パス 1427"
                          d="M59.212,0H56.319L53.832,4.545,51.345,0H48.453l2.918,5.332H48.882v2.02H52.3V8.669H48.882v2.02H52.3v2.581h3.059V10.689h3.421V8.669H55.362V7.353h3.421V5.332H56.294Z"
                          transform="translate(0)" fill="#fff" />
                      </g>
                    </g>
                  </svg>
                  <span>
                    {{ item.salary }}
                  </span>
                </h4>
              </div>
              <div class="p-box">
                <div class="box">
                  <p v-for="text in item.information" class="p-txt">
                    {{ text }}
                  </p>
                </div>
              </div>
            </div>
          </li>
        </ul>
      </div>
    </div>
    <div class="dashboard-lis" style="padding-top: 0px">
      <div class="dashboard-lis-title">
        <div class="dashboard-lis-title-txt">
          <svg xmlns="http://www.w3.org/2000/svg" width="5" height="39.5" viewBox="0 0 5 39.5">
            <g id="グループ_1103" data-name="グループ 1103" transform="translate(-473.5 -1804.5)">
              <rect id="長方形_2437" data-name="長方形 2437" width="20" height="5"
                transform="translate(473.5 1844) rotate(-90)" fill="#ff6c7a" />
              <rect id="長方形_2438" data-name="長方形 2438" width="20" height="5"
                transform="translate(473.5 1824.5) rotate(-90)" fill="#fbb" />
            </g>
          </svg>
          <h2 class="dashboard-lis-title-title">
            {{ $lang('每一次挑戰都是一場綻放的旅程') }}
          </h2>
        </div>
      </div>
      <div class="dashboard-lis-list">
        <ul class="dashboard-lis-list-ul2">
          <li v-for="(item, index) in clsLis" :key="index" @click="() => { if (!isLogin()) navigateTo('/login') }">
            <div class="li-box2">
              <a>
                <img class="lazyload" :src="item.src" alt="" />
                <div>
                  <p>{{ item.title }}</p>
                  <h3>{{ item.name }}</h3>
                </div>
              </a>
              <span></span>
            </div>
          </li>
        </ul>
      </div>
    </div>
    <div class="ul-container">
      <section v-if="false" class="ul-ad">
        <div class="ul-inner-container">
          <div class="ul-ad-content">
            <div class="ul-ad-txt">
              <span class="ul-ad-sub-title">{{
                $lang('積極的家庭工作者')
              }}</span>
              <h2 class="ul-section-title" style="text-shadow: 2px 3px 5px #4d4d4d">
                {{ $lang('我們衷心感謝您一直以來的惠顧。') }}
              </h2>
              <div class="ul-ad-categories">
                <span class="category"><span><i class="fa-solid fa-check"></i></span>{{ $lang('日本') }}</span>
                <span class="category"><span><i class="fa-solid fa-check"></i></span>{{ $lang('韓国') }}</span>
                <span class="category"><span><i class="fa-solid fa-check"></i></span>{{ $lang('台灣') }}</span>
                <span class="category"><span><i class="fa-solid fa-check"></i></span>{{ $lang('全球') }}</span>
              </div>
            </div>
            <!-- <div class="ul-ad-img">
              <img src="@/assets/image/index/ad01-img.png" alt="Ad Image">
            </div> -->
            <a class="ul-btn" style="font-weight: bold">{{ $lang('了解詳情') }}
              <svg t="1739515035390" class="icon" viewBox="0 0 1024 1024" version="1.1"
                xmlns="http://www.w3.org/2000/svg" p-id="1851" xmlns:xlink="http://www.w3.org/1999/xlink" width="20"
                height="20">
                <path class="color" d="M416 192v64h306.016L200.96 776.992 247.04 823.04 768 301.984V608h64V192H416z"
                  p-id="1852"></path>
              </svg>
            </a>
          </div>
        </div>
      </section>
    </div>
    <indexFooter />
  </div>
</template>

<style scoped lang="sass">

.video-element
  width: 100%
  height: 100%
  object-fit: cover
  /* 隐藏 Webkit 原生控件（关键！） */
  &::-webkit-media-controls
    display: none !important

  &::-webkit-media-controls-enclosure
    display: none !important

  /* 强制内联播放（冗余属性确保兼容） */
  -webkit-playsinline: true
  playsinline: true



//所有控件
video::-webkit-media-controls-enclosure
    display: none
.dashboard
  padding-top: 123px
  @media (max-width: 768px)
    padding-top: 70px
.dashboard-video
  position: relative
  width: 100%
  video
    width: 100%
.dashboard-banner
  padding:  60px 50px
  @media (max-width: 768px)
    padding:  20px 20px
  .bannerBox
    border-radius: 18px
    max-width: 100%
    width: 1320px
    margin: 0 auto
    overflow: hidden
    transition: 0.4s ease
    position: relative
    padding: 15px
    background-color: #ffdddd
    display: grid
    grid-template-columns: repeat(2, 1fr)
    @media (max-width: 992px)
      grid-template-columns: repeat(1, 1fr)
    .bnr-txt
      padding: 20px
      @media (max-width: 992px)
        padding: 20px 0 0
      p
        color: #3f3a39
        font-size: 18px
        font-weight: 700
        margin-bottom: 20px
    .bnr-img
      position: relative
      overflow: hidden
      border-radius: 12px
      .bnr-t
        position: absolute
        top: 0
        left: 0
        z-index: 1
        padding: 20px 40px 0 20px
        text-shadow: 2px 2px 5px #ff6c7a
        color: #fff
        font-size: 22px
        font-weight: 700
        @media (max-width: 992px)
          font-size: 16px !important
          padding: 20px 20px 0 20px
          h2
            font-size: 14px !important
        h2
          font-size: 18px
      img
        width: 100%
        height: auto
        object-fit: cover
        transition: 0.4s ease
      &:hover
        img
          transform: scale(1.05)
.dashboard-lis
  background-color: #f2f2f2
  padding: 50px 20px 50px
  .dashboard-lis-banner
    max-width: 1080px
    width: 100%
    margin: 0 auto 20px
    overflow: hidden
    border-radius: 10px
    &:hover
      img
        transform: scale(1.05)
    img
      width: 100%
      transition: 0.4s ease
  .dashboard-lis-title
    margin-bottom: clamp(40px, 2.05vw, 30px)
    display: flex
    align-items: center
    justify-content: center
    flex-direction: column
    .dashboard-lis-title-txt
      display: flex
      align-items: center
      gap: clamp(10px, 1.05vw, 20px)
      .dashboard-lis-title-title
        font-size: 30px
        color: #3f3a39
        font-weight: 700
  .dashboard-lis-list
    max-width: 1080px
    width: 100%
    margin: 0 auto
    .dashboard-lis-list-ul
      display: grid
      grid-template-columns: repeat(4, 1fr)
      gap: clamp(20px, 1.05vw, 30px)
      @media (max-width: 1024px)
        grid-template-columns: repeat(3, 1fr)
      @media (max-width: 768px)
        grid-template-columns: repeat(1, 1fr)
      li
        background-color: #fff
        border-radius: 10px
        overflow: hidden
        box-shadow: 2px 2px 10px rgba(0, 0, 0, .1)
      .li-box
        width: 100%
        .img
          width: 100%
          height: 130px
          overflow: hidden
          text-align: center
          @media (max-width: 922px)
            height: 196px
          img
            width: 100%
            vertical-align: middle
            object-fit: cover
        .txt
          padding: 10px 8px
          background-color: #ffb9bf
          p
            font-size: 15px
            color: #514848
            font-weight: 600
            overflow: hidden
            text-overflow: ellipsis
            display: -webkit-box
            -webkit-line-clamp: 2
            overflow:hidden
            /*! autoprefixer: off */
            -webkit-box-orient: vertical
        .text
          padding: 20px 20px 0
          h4
            display: flex
            align-items: flex-start
            gap: clamp(10px, 1.05vw, 20px)
            margin-bottom: 10px
            font-size: 14px
            .svg-color
              width: 20px
              min-width: 20px
              height: 20px
        .p-box
          padding: 5px 20px 40px
          .box
            padding: 10px 5px
            border: 2px solid rgba(0, 0, 0, .1)
          .p-txt
            font-size: 14px
            line-height: 30px
            overflow: hidden
            text-overflow: ellipsis
            display: -webkit-box
            -webkit-line-clamp: 1
            overflow: hidden
            /*! autoprefixer: off */
            -webkit-box-orient: vertical
    .dashboard-lis-list-ul2
      display: grid
      grid-template-columns: repeat(3, 1fr)
      gap: clamp(20px, 1.05vw, 30px)
      @media (max-width: 992px)
        grid-template-columns: repeat(2, 1fr)
      @media (max-width: 768px)
        grid-template-columns: repeat(1, 1fr)
      .li-box2
        overflow: hidden
        position: relative
        padding: 20px 20px 60px
        border-radius: 10px
        background: #fff8ea
        border: 2px solid #fff
        font-weight: 700
        h3
          font-size: 18px
        &:hover
          opacity: .8
        a
          width: 100%
          display: flex
          align-items: center
          justify-content: center
          flex-direction: column
          text-align: center
          gap: 5px
        span
          display: inline-block
          position: absolute
          bottom: 0
          right: 0
          width: 0
          height: 0
          border-color: transparent transparent #ff6c7a
          border-style: solid
          border-width: 0 0 62px 102px
          &::after
            display: block
            position: absolute
            content: ""
            width: 34px
            height: 10px
            bottom: -46px
            right: 17px
            border-bottom: 2px solid #fff
            border-right: 2px solid #fff
            -webkit-transform: skew(45deg)
            transform: skew(45deg)
.ul-container
  margin: 0 clamp(15px, 3.15vw, 60px)
.lis-cont
  margin: clamp(40px, 4.2vw, 80px) clamp(0px, 2.88vw, 150px)
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
    border: 2px solid #ff6c7ab9
    font-size: 17px
    border-radius: 10px
    overflow: hidden
    padding: clamp(6px, 0.47vw, 9px)
    display: flex
    align-items: center
    gap: clamp(12px, 1.05vw, 20px)
    position: relative
    z-index: 1
    .svg-color
      stroke: #ff6c7ab9
    &::before
      content: ""
      position: absolute
      inset: 0
      opacity: 0
      background: linear-gradient(90deg,#ffddddc2, #ff6c7ab9)
      transition: 0.4s ease
      z-index: -1
    &:hover
      color: #FFF
      .svg-color
        stroke: #fff
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
      color: #ff6c7a
      font-size: clamp(13px, 0.84vw, 16px)
    &:hover .ico
      background-color: #FFF
.ul-ad
  box-sizing: border-box
  margin: clamp(40px, 4.2vw, 80px) 0
  border-radius: clamp(15px, 2.63vw, 50px)
  background: #ff6c7a80
  padding-top: clamp(37px, 3vw, 57px)
  position: relative
  overflow: hidden
  z-index: 1
  &::before
    content: ""
    position: absolute
    inset: 0
    background: url(@/assets/image/index/home-bm.svg) no-repeat center bottom
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
  .ul-ad-img
    img
      filter: saturate(.5)
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
    color: #ff6c7a
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
</style>
