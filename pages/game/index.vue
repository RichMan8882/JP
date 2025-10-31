<script lang="ts" setup>
// import
import '@/assets/sass/TempPage.min.css'
import anime from '@/assets/js/anime.es.js'
const { t } = useI18n()
const siteStore = useSiteStore()
const PlayerStore = usePlayerStore()
const orderStore = useOrderStore()
const { queryKlines, queryRounds, queryOrder, bet } = useGameStore()
const { getListenkey } = PlayerStore
const { locale } = useI18n()
const lang = locale.value
const { queryInstruction, queryNews } = useSiteStore()
const router = useRouter()
const timeoutId = ref(null)
const hourDegrees = ref(0)
const minuteDegrees = ref(0)
const isMenuVisible = ref(false)

const secondDegrees = ref(0)
// data
const coinBoxChecked = ref(false)
const record = ref({
  title: t('當前委託'),
  type: 'currentOrder',
  search: true
})
const search = ref({
  symbol: '',
  roundNo: '',
  dateRangeStart: '',
  dateRangeEnd: '',
  limit: 60,
  page: 1,
  pageSize: 100
})
const dataList = ref([
  {
    title: t('出荷数'),
    unit: t(''),
    index: '0'
  },
  {
    title: t('返品・交換数'),
    unit: t(''),
    index: '1'
  }
  // {
  //   title: t('待放置數量'),
  //   unit: t('劑'),
  //   index: '2'
  // },
  // {
  //   title: t('待放置廠商'),
  //   unit: t(''),
  //   index: '3'
  // }
])
const showToolPopup = ref(false)
const symbol = ref('')
const symbolData: any = ref(null)
const availableCurrency: any = ref([])
const defaultTime = new Date(2000, 1, 1, 12, 0, 0)
const socket: any = ref(null)
const socketConnected = ref(false)
const betData = ref({
  playerId: PlayerStore.playerInfo.id,
  option: [],
  amount: '',
  roundNo: ''
})
const recordList = ref([])
const historyRecordList = ref([])
// 圖表數據值
const nowTimestamp: any = ref(null)
const socketNewPrice: any = ref(null)
const isFirstGet = ref(true)
const createChartData = ref(null)
const socketCurrentRoundCountdown = ref(0)
const news = ref(null)
const rule = ref(null)
const disableBet = ref(false)
const currentSelectSymbolPrice = ref(0)
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
const selectSymbol = (value: any) => {
  symbol.value = value.symbol
  symbolData.value = value.symbolData
  closeWebSocket()
  document.getElementById('coinBox').checked = false
}

// method
const goPopup = async (title: string) => {
  console.log('recordList', recordList)
  record.value.title = title
  showToolPopup.value = true
  search.value.symbol = symbol.value
  // 確認order數據

  if (title === t('當前委託')) {
    orderList.value = socketOrderList.value.sort((a, b) => {
      return new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime()
    })
    record.value.type = 'currentOrder'
    record.value.search = true
    return
  }
  if (title === t('歷史委託')) {
    orderList.value = socketOrderList.value.sort((a, b) => {
      return new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime()
    })
    record.value.type = 'historyOrder'
    record.value.search = true
    return
  }
  if (title === t('歷史盤口')) {
    record.value.title = ' 歷史總覽'
    record.value.type = 'historyRecord'
    const response = await queryRounds({
      symbol: symbol.value,
      limit: 60,
      page: 1,
      pageSize: 100
    })
    console.log('queryRounds', response.data.result)
    historyRecordList.value = response.data.result.filter((item: any) => {
      return Number(item.roundNo) < Number(betData.value.roundNo)
    })
    console.log('betData.value.roundNo', betData.value.roundNo)
    console.log('queryRounds', historyRecordList.value)
    record.value.search = true
    return
  }
  if (title === t('網站公告')) {
    record.value.type = 'announcement'
    record.value.search = false
    return
  }
  if (title === t('規則說明')) {
    record.value.type = 'rule'
    record.value.search = false
  }
}
const closePopup = () => {
  showToolPopup.value = false
}
const connectRecordList = computed(() => {
  const title = record.value.type
  switch (title) {
    case 'currentOrder':
      orderList.value = socketOrderList.value
        .filter((item: any) => item.status === 0)
        .sort((a, b) => {
          return (
            new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime()
          )
        })
      return orderList.value
    case 'historyOrder':
      orderList.value = socketOrderList.value
        // .filter((item: any) => item.status === 1)
        .sort((a, b) => {
          return (
            new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime()
          )
        })
      return orderList.value
  }
})
const addBetGameType = (type: any) => {
  console.log(type)

  betData.value.option = [type + '']
  // if (betData.value.option.includes(type)) {
  //   betData.value.option = betData.value.option.filter((item) => item !== type)
  // } else {
  //   betData.value.option.push(type)
  // }
  // console.log(betData.value.option)
}

const checkBetData = () => {
  if (!disableBet.value) {
    disableBet.value = true
    console.log(checkBetData.value)

    try {
      betData.value.productId = symbolData.value.id
      console.log(betData.value)
      const pairData = symbolData.value
      console.log('pairData', pairData)
      // banned action
      if (pairData.banned) {
        ElNotification({
          message: pairData.notice,
          type: 'error',
          showClose: false
        })
        return
      }
      // banned player
      const bannedPlayer = pairData.bannedPlayers.find(
        (item: any) => item.playerId === PlayerStore.playerInfo?.id
      )
      if (bannedPlayer) {
        if (bannedPlayer.banned) {
          ElNotification({
            message: bannedPlayer.reason,
            type: 'error',
            showClose: false
          })
          return
        }
      }
      if (betData.value.amount === '') {
        // ElNotification({
        //   message: `${t('請輸入下單金額')}`,
        //   type: 'error',
        //   showClose: false
        // })
        // return
        betData.value.amount = 300
      }
      if (betData.value.option.length === 0) {
        ElNotification({
          message: `${t('請選擇參數設置模式')}`,
          type: 'error',
          showClose: false
        })
        return
      }
      if (
        parseFloat(betData.value.amount) < symbolData.value.config.minBetAmount
      ) {
        ElNotification({
          message: ` ${t('最低交易金額為')} ¥ ${symbolData.value.config.minBetAmount}`,
          type: 'error',
          showClose: false
        })
        return
      }
      if (
        parseFloat(betData.value.amount) > symbolData.value.config.maxBetAmount
      ) {
        ElNotification({
          message: `${t('最高交易金額為')} ¥  ${symbolData.value.config.maxBetAmount}`,
          type: 'error',
          showClose: false
        })
        return
      }
      setTimeout(
        async () => {
          betData.value.amount = betData.value.amount.toString()
          const response = await bet(betData.value)
          console.log('bet response', response)
          if (response.success) {
            ElMessageBox.alert(
              `
               <p style="margin:0 0 8px 0"> ${t('系統編號')}: ${response.data.roundNo} </p>
               <p style="margin:0 0 8px 0"> ${t('設置選項')}: ${gameOptionNameList(
                response.data.option
              )} </p>
               <p style="margin:0 0 8px 0"> ${t('時間')}: ${formatDate(
                response.data.openAt
              )} </p>
             `,
              `${t('設置成功')}`,
              {
                confirmButtonText: `${t('確認')}`,
                center: true,
                dangerouslyUseHTMLString: true,
                customClass: 'messageStyle'
              }
            )
          } else {
            // ElNotification({＄3778405
            //   title: response.message,
            //   type: 'error',
            //   showClose: false
            // })
          }
          clearBetData()
          await PlayerStore.fetchInfo()
        },
        bannedPlayer ? (bannedPlayer.delay + 1) * 1000 : 1000
      )
    } catch (error) {
      console.error(error)
    } finally {
      setTimeout(() => {
        disableBet.value = false
      }, 3000)
    }
  }
}
const gameOptionOdd = (type: number) => {
  switch (type) {
    case 0:
      return symbolData.value
        ? symbolData.value.config.option[0].odds * 100 + '%'
        : 0.1 * 100 + '%'
    case 1:
      return symbolData.value
        ? symbolData.value.config.option[1].odds * 100 + '%'
        : 0.1 * 100 + '%'
    case 2:
      return symbolData.value
        ? symbolData.value.config.option[2].odds * 100 + '%'
        : 0.1 * 100 + '%'
    case 3:
      return symbolData.value
        ? symbolData.value.config.option[3].odds * 100 + '%'
        : 0.1 * 100 + '%'
    case 4:
      return symbolData.value
        ? symbolData.value.config.option[4].odds * 100 + '%'
        : 0.1 * 100 + '%'
    case 5:
      return symbolData.value
        ? symbolData.value.config.option[5].odds * 100 + '%'
        : 0.1 * 100 + '%'
    case 6:
      return symbolData.value
        ? symbolData.value.config.option[6].odds * 100 + '%'
        : 0.1 * 100 + '%'
    default:
      return ''
  }
}

const showOption = (type: number) => {
  switch (type) {
    case 0:
      return symbolData.value ? symbolData.value.config.option[0].isOpen : false
    case 1:
      return symbolData.value ? symbolData.value.config.option[1].isOpen : false
    case 2:
      return symbolData.value ? symbolData.value.config.option[2].isOpen : false
    case 3:
      return symbolData.value ? symbolData.value.config.option[3].isOpen : false
    case 4:
      return symbolData.value ? symbolData.value.config.option[4].isOpen : false
    case 5:
      return symbolData.value ? symbolData.value.config.option[5].isOpen : false
    case 6:
      return symbolData.value ? symbolData.value.config.option[6].isOpen : false
    default:
      return true
  }
}

const clearBetData = () => {
  betData.value = {
    playerId: PlayerStore.playerInfo.id,
    productId: '',
    option: [],
    amount: '',
    roundNo: ''
  }
  // document.getElementById('game-1').checked = false
  // document.getElementById('game-2').checked = false
  // document.getElementById('game-3').checked = false
  // document.getElementById('game-4').checked = false
  // document.getElementById('game-5').checked = false
  // document.getElementById('game-6').checked = false
  // document.getElementById('game-7').checked = false
}

const initMilliseconds = (timestamp: any) => {
  const newDate = new Date(timestamp)
  newDate.setMilliseconds(0)
  return newDate.getTime()
}

const randomVolume = () => {
  return parseFloat((Math.random() * 100 + 1).toFixed(2))
}

const formatDate = (timestamp: string) => {
  const date = new Date(timestamp)
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  const hours = String(date.getHours()).padStart(2, '0')
  const minutes = String(date.getMinutes()).padStart(2, '0')
  const seconds = String(date.getSeconds()).padStart(2, '0')

  return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`
}

const gameOptionNameList = (list: Array) => {
  const result = []
  list.forEach((item) => {
    result.push(gameOptionName(item))
  })
  return result.join(', ')
}
const gameOptionName = (type: Number) => {
  switch (type) {
    case 0:
      return `${t('高')}`
    case 1:
      return `${t('低')}`
    case 2:
      return `${t('單')}`
    case 3:
      return `${t('雙')}`
    case 4:
      return `${t('出荷')}`
    case 5:
      return `${t('返品・交換')}`
    case 6:
      return `${t('反指標')}`
    default:
      return ''
  }
}
const gameResultName = (type: Number) => {
  // 0 : 高、1 : 低、2 : 單、3 : 雙、4 : 漲、5 : 跌 6 : 合
  switch (type) {
    case 0:
      return `${t('高')}`
    case 1:
      return `${t('反指標')}`
    case 2:
      return `${t('單')}`
    case 3:
      return `${t('雙')}`
    case 4:
      return `${t('漲')}`
    case 5:
      return `${t('跌')}`
    case 6:
      return `${t('和')}`
    default:
      return `${t('未知')}`
  }
}
const getChartData = async (timestamp: any) => {
  // 取得遠端資料
  // 取 3 分鐘的資料
  const now = new Date(nowTimestamp.value)
  now.setSeconds(0, 0)
  now.setMinutes(now.getMinutes() - 3)
  const startTime = now.getTime()
  const endTime = timestamp
  // const data = {
  //   symbol: symbol.value,
  //   interval: '1',
  //   startTime,
  //   endTime
  // }
  // console.log(symbolData.value)
  const data = {
    productId: symbolData.value.id
  }
  const klineData = await queryKlines(data)
  if (klineData.success) {
    isFirstGet.value = false // 第一次獲取開關關閉
    const { result } = klineData.data
    // console.log('klines', result)
    const newKLines = Object.entries(result).map(([timestamp, value]) => {
      const volume = randomVolume()
      const turnover = Number(value.close) * volume
      return {
        timestamp: Number(timestamp) * 1000,
        open: Number(value.open),
        close: Number(value.close),
        high: Number(value.high),
        low: Number(value.low),
        volume,
        turnover
      }
    })
    createChartData.value = { newKLines, time: timestamp }
  }
}

const searchOrder = async () => {
  const startDate = new Date(search.value.dateRangeStart)
  search.value.dateRangeStart = startDate.getTime()
  const endDate = new Date(search.value.dateRangeEnd)
  search.value.dateRangeEnd = endDate.getTime()
  console.log('searchOrder', search.value)
  if (!search.value.dateRangeStart) {
    delete search.value.dateRangeStart
  }
  if (!search.value.dateRangeEnd) {
    delete search.value.dateRangeEnd
  }
  if (record.value.type !== 'historyRecord') {
    const response = await queryOrder(search.value)
    if (response.success) {
      orderList.value = response.data.result
    } else {
      ElNotification({
        message: response.message,
        type: 'error'
      })
    }
  } else {
    // 搜尋盤口
    const response = await queryRounds(search.value)
    if (response.success) {
      historyRecordList.value = response.data.result
    } else {
      ElNotification({
        message: response.message,
        type: 'error'
      })
    }
  }
  search.value.dateRangeStart = ''
  search.value.dateRangeEnd = ''
  search.value.roundNo = ''
}

const pickerOptions = {
  daterange: {
    maxTime: '', // 最大日期
    minTime: '', // 最小日期
    max: 30 // 限制范围 30天
  }
}
const disabledDate = (time) => {
  const threeMonthsAgo = new Date()
  threeMonthsAgo.setMonth(threeMonthsAgo.getMonth() - 3)
  return time.getTime() < threeMonthsAgo.getTime()
}

const productList: any = ref([])

const orderList: any = ref([])
const socketOrderList: any = ref([])
const ogOrderList: any = ref(null)
// ctyptoData
const reconnected = ref(true)
const closeWebSocket = async () => {
  if (socket.value) {
    socket.value.close()
  }
}

const sendMessage = (msg: any) => {
  // send message to server
  socket.value.send(JSON.stringify(msg))
}

// 連接 socket
const startConnectWebSocket = async () => {
  const getListenkeyRes = await getListenkey()
  console.log('getListenkeyRes', getListenkeyRes)
  const runtimeConfig = useRuntimeConfig()
  const { SOCKETBASE } = runtimeConfig.public
  socket.value = new WebSocket(
    `${SOCKETBASE}/${getListenkeyRes.data.listenkey}`
  )
  socket.value.onopen = (event) => {
    // console.log('Connected to socket', event)
    socketConnected.value = true
    sendMessage({
      op: 'subscribe',
      channel: ['kline', 'order', 'product']
    })
  }
  socket.value.onmessage = async (e) => {
    const message = JSON.parse(e.data)
    // console.log('收到來自 socket 的訊息', message);
    const { event, data } = message
    switch (event) {
      case 'PRODUCT_UPDATE': {
        // console.log('PRODUCT_UPDATE', data)
        productList.value = data.result
        if (symbolData.value === null) {
          symbol.value = productList.value[0].symbol
          symbolData.value = productList.value[0]
        }
        break
      }
      case 'ORDER_UPDATE': {
        // console.log('ORDER_UPDATE', data)
        socketOrderList.value = data.result
      }
      case 'KLINE_UPDATE': {
        // console.log('KLINE_UPDATE', data)
        if (productList.value.length > 0) {
          productList.value.forEach((item: any) => {
            const findSystemPrice = data.result.find(
              (socketData: any) => socketData.symbol === item.symbol
            )
            // console.log('findSystemPrice', findSystemPrice)

            const currency = availableCurrency.value.find(
              (currency: any) => currency.symbol === item.symbol
            )
            if (currency) {
              currency.price.push({
                open: parseFloat(
                  currency.price[currency.price.length - 1].close
                ),
                close: findSystemPrice.price
                  ? parseFloat(findSystemPrice.price.close)
                  : 0
              })
            } else {
              availableCurrency.value.push({
                symbol: item.symbol,
                price: [
                  {
                    open: findSystemPrice.price
                      ? parseFloat(findSystemPrice.price.open)
                      : 0,
                    close: findSystemPrice.price
                      ? parseFloat(findSystemPrice.price.close)
                      : 0
                  }
                ],
                symbolData: item
              })
            }
          })
        }

        // 圖表列表數據
        const { timestamp: correctTimestamp } = message
        const lastTimestamp = Math.floor(nowTimestamp.value / 1000)
        if (lastTimestamp !== Math.floor(correctTimestamp / 1000)) {
          const timestamp = initMilliseconds(correctTimestamp)
          nowTimestamp.value = timestamp
          const symbolData = data.result.find(
            (item) => item.symbol === symbol.value
          )
          // console.log('symbol.value', symbol.value)
          // console.log('symbolData', symbolData)
          // 已選擇數據整理
          if (symbolData) {
            const { price } = symbolData
            // 數據整理
            let newPrice = Object.entries(price).reduce((acc, [key, value]) => {
              acc[key] = parseFloat(value)
              return acc
            }, {})
            const volume = randomVolume()
            const turnover = newPrice.close * volume
            newPrice = {
              timestamp,
              ...newPrice,
              turnover,
              volume
            }
            socketNewPrice.value = newPrice

            // 第一次取得所有數據 & 產生空資料
            if (isFirstGet.value) getChartData(timestamp)

            // currentSelectSymbolPrice
            currentSelectSymbolPrice.value = price.close
          }
        }
        break
      }
      case 'SERVER_TIME': {
        const {
          currentRoundId,
          currentRoundCountdown,
          allowBetRoundId,
          allowBetRoundCountdown
        } = data
        betData.value.roundNo = allowBetRoundId
        socketCurrentRoundCountdown.value = allowBetRoundCountdown
        // console.log(data);
        if (currentRoundCountdown === 59) {
          await PlayerStore.fetchInfo()
          // const queryOrderRes = await orderStore.queryOrder({
          //   pair: symbol.value
          // })
          // recordList.value = queryOrderRes.data.result
        }
        break
      }
      default:
        break
    }
  }
  socket.value.onclose = async () => {
    console.log('Disconnected from socket')
    isFirstGet.value = true
    socketConnected.value = false
    if (reconnected) {
      await startConnectWebSocket()
      console.log('reconnected to socket')
    }
  }

  socket.value.onerror = (error) => {
    socketConnected.value = false
    console.error('WebSocket error:', error)
  }
}
// 更新時鐘的函數
const updateClock = () => {
  const now = new Date()
  const hours = now.getHours()
  const minutes = now.getMinutes()
  const seconds = now.getSeconds()

  hourDegrees.value = (hours % 12) * 30 + minutes * 0.5
  minuteDegrees.value = minutes * 6
  secondDegrees.value = seconds * 6
}

// 初次呼叫更新時鐘
updateClock()

// 定義一些狀態變數和函數
const currentTime = ref('')
const countdown60 = ref(60)
const countdown70 = ref(70)
const inputValue = ref(50)
const counting = ref({
  ers: 33.71,
  sers: 3.3,
  ber: 400,
  es: 70
})

const updateCountingNumber = (value, min, max, fixed) => {
  counting.value[value] = parseFloat(
    (Math.random() * (max - min) + min).toFixed(fixed)
  )
}

// 更新當前時間的函數
const updateTime = () => {
  const now = new Date()
  const hours = String(now.getHours()).padStart(2, '0')
  const minutes = String(now.getMinutes()).padStart(2, '0')
  const seconds = String(now.getSeconds()).padStart(2, '0')
  currentTime.value = `${hours}:${minutes}:${seconds}`
}

// 更新 60 秒倒數計時的函數
const updateCountdown60 = () => {
  if (countdown60.value > 0) {
    countdown60.value--
  } else {
    countdown60.value = 60
  }
}

// 更新 70 秒倒數計時的函數
const updateCountdown70 = () => {
  if (countdown70.value > 0) {
    countdown70.value--
  } else {
    countdown70.value = 70
  }
}

const ersTimer = ref('')
const sersTimer = ref('')
const berTimer = ref('')
const esTimer = ref('')

// 組件掛載時設置計時器並更新時間和倒數計時
const randomTimer = ref(null)

await onMounted(async () => {
  handleDataInView()
  randomTimer.value = setInterval(() => {
    triggerDataRandom()
  }, 5000)
  await startConnectWebSocket()

  const pair = symbol.value
  const type = 'game'
  const path = 'game'
  /** await Promise then  */
  // 開始加載數據（此處沒有使用 await，因此主線程不會被阻塞）
  const queryInstructionPromise = queryInstruction(lang, type)
  const queryNewsPromise = queryNews(lang, path)
  // 使用 .then() 方法處理異步任務結果
  queryInstructionPromise
    .then((queryInstructionRes) => {
      // 處理響應結果
      rule.value = queryInstructionRes.data
    })
    .catch((error) => {
      console.error('Error loading instruction data:', error)
    })

  queryNewsPromise
    .then((queryNewsRes) => {
      // 處理響應結果
      news.value = queryNewsRes.data
    })
    .catch((error) => {
      console.error('Error loading news data:', error)
    })
  /** await Promise then  */

  const resetTimer = () => {
    // if (timeoutId) {
    //   clearTimeout(timeoutId.value)
    // }
    // timeoutId.value = setTimeout(
    //   () => {
    //     navigateTo('/user')
    //   },
    //   siteStore.siteData?.gameCenterOutInterval
    //     ? siteStore.siteData?.gameCenterOutInterval * 60 * 1000
    //     : 5 * 60 * 1000
    // ) // 5 分钟
  }
  window.onload = resetTimer
  window.onmousemove = resetTimer
  window.onmousedown = resetTimer
  window.ontouchstart = resetTimer
  window.onclick = resetTimer
  window.onscroll = resetTimer
  window.onkeypress = resetTimer

  updateTime()

  ersTimer.value = setInterval(
    () => updateCountingNumber('ers', 30, 50, 2),
    1000
  )
  sersTimer.value = setInterval(
    () => updateCountingNumber('sers', 3, 8, 1),
    1000
  )
  berTimer.value = setInterval(
    () => updateCountingNumber('ber', 400, 600, 1),
    1000
  )
  esTimer.value = setInterval(() => updateCountingNumber('es', 70, 90, 1), 1000)
})

// 組件卸載時清除計時器
onUnmounted(() => { })

onBeforeUnmount(() => {
  clearInterval(ersTimer)
  clearInterval(sersTimer)
  clearInterval(berTimer)
  clearInterval(esTimer)
  reconnected.value = false
  closeWebSocket()
})

// 處理加法按鈕事件的函數
const addToInput = (value) => {
  betData.value.amount = value
}

const cryptoSelect = ref(false)
const openCryptoSelect = () => {
  cryptoSelect.value = !cryptoSelect.value
}
const cryptoName = (crypto) => {
  switch (crypto.symbol) {
    case 'BTCUSDT':
      return '太陽能'
    case 'WFEURUSD':
      return '風能'
    case 'WFGBPUSD':
      return '地熱能'
    case 'WFUSDTWD':
      return '海洋能'
    default:
      return crypto.label || crypto.name || crypto.symbol
  }
}
// action methods
const symbolChange = (symbol: string) => {
  switch (symbol) {
    case 'BTCUSDT':
      return '太陽能'
    case 'WFEURUSD':
      return '風能'
    case 'WFGBPUSD':
      return '地熱能'
    case 'WFUSDTWD':
      return '海洋能'
    default:
      return ''
  }
}
const betFormatNumber = (num) => {
  if (num >= 1000000) {
    return (num / 1000000).toFixed(0) + 'M'
  } else if (num >= 1000) {
    return (num / 1000).toFixed(0) + 'k'
  } else {
    return num.toString()
  }
}
const betAmountsList = computed(() => {
  if (symbolData.value) {
    return symbolData.value.config.betAmount
  } else {
    return []
  }
})
const formatHour = (timestamp: string) => {
  const date = new Date(timestamp)
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  const hours = String(date.getHours()).padStart(2, '0')
  const minutes = String(date.getMinutes()).padStart(2, '0')
  const seconds = String(date.getSeconds()).padStart(2, '0')

  return `${hours}:${minutes}:${seconds}`
}
const dataNumber = ref({
  0: 0,
  1: 0,
  2: 0,
  3: 0
})

const handleDataInView = () => {
  anime({
    easing: 'easeInOutSine',
    targets: dataNumber.value,
    0: 67218,
    1: 21168,
    2: 149006,
    3: 28,
    round: 1,
    duration: 2000
  })
}

function getRandomInRange() {
  return Math.random() * 0.2 + 0.9
}
const triggerDataRandom = () => {
  anime({
    easing: 'easeInOutSine',
    targets: dataNumber.value,
    0: 67218 * getRandomInRange(),
    1: 21168 * getRandomInRange(),
    2: 149006 * getRandomInRange(),
    3: 28 * getRandomInRange(),
    round: 1,
    duration: 2000
  })
}
</script>

<template>
  <div class="page">
    <div class="page-video">
      <video autoplay muted loop class="video-element" webkit-playsinline="true" playsinline="true"
        x5-video-player-type="h5" x5-video-player-fullscreen="false">
        <source src="https://upload.comethike.com/uploads/1759578336584.mov" />
      </video>
    </div>
    <div class="user-t white">
      <div class="login-no">
        <div class="goBack" @click="navigateTo('/user')">
          {{ t('返回首頁') }}
        </div>
        <span class="log-user">
          <span style="margin-right: 5px">
            <i class="fa-solid fa-user"></i>
          </span>
          Hello,<span style="display: inline-block; margin: 0 5px">{{ PlayerStore?.playerInfo?.username }}
          </span>
          <span style="display: inline-block">
            ¥ {{ new Intl.NumberFormat('zh-TW').format(playerWalletBalance) }}
          </span>
        </span>
      </div>
    </div>
    <div class="sys">
      <div class="container">
        <div class="row text-center">
          <div class="col-md-12 col-sm-6 col-xs-6">
            <div class="heading-count">
              <h2>{{ $lang('工單') }}</h2>
              <p>
                <span style="font-size: 10px">注文番号</span> <br />{{
                  betData.roundNo || ''
                }}
              </p>
              <div class="card-body-time" :style="socketCurrentRoundCountdown < 11 ? 'color:red' : ''">
                {{ $lang('剩餘時間:') }}
                <span>{{ socketCurrentRoundCountdown || '0' }}s</span>
              </div>
            </div>
          </div>
          <div class="btns" style="padding: 40px 0 20px">
            <div v-for="(item, index) in dataList" :key="`data-${index}`" class="time-entry">
              <span id="a1a_value">&nbsp;{{ dataNumber[item.index] }}&nbsp;{{ item.unit }}</span>{{ item.title }}
            </div>
            <div class="round button radio-group" :class="{
              active: betData.option[0] == 4
            }" @click="addBetGameType(4)">
              {{ $lang('出荷') }}
            </div>

            <div class="round button radio-group" :class="{ active: betData.option[0] == 5 }"
              @click="addBetGameType(5)">
              {{ $lang('返品/交換') }}
            </div>
            <div class="button-group" @click="checkBetData">
              <button id="confirmButton" class="btn btn-common">
                {{ $lang('確定') }}
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<style lang="scss">
.messageStyle {
  border-radius: 15px !important;

  .el-message-box__header {
    .el-message-box__title {
      color: #f58194 !important;
      font-size: 1.5rem !important;
      font-weight: 600;
    }
  }

  .el-message-box__content {
    color: #000;

    .el-message-box__container {
      .el-message-box__message span {
        color: #0a00ff !important;
      }
    }
  }

  .el-message-box__btns {
    height: 80px;
    text-align: center;

    .el-button {
      height: 45px;
      width: 160px;
      font-size: 20px;
      background: #f58194;
      color: #fff;
      background-color: #f58194;
      border: 1px solid #f58194;
    }
  }
}
</style>
<style lang="sass" scoped>
.page

  // animation: backgroundSwitch 15s infinite
  // background-position: center
  // background-size: cover

  position: relative
  height: 100vh
  overflow: auto
  .page-video
    position: absolute
    z-index: -1
    object-fit: cover
    top: 0
    left: 0
    width: 100%
    height: 100%
    video
      width: 100%
      height: 100%
      object-fit: cover
  .sbps
    position: absolute
    top: 0
    bottom: 0
    left: 0
    right: 0
</style>
<style lang="sass" scoped>

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



.white
    color: #FFF
.black
  color: #0C1239
/* user

.user-t
  background: #ff6c7a
  padding: 12px 0
  font-size: 16px
  margin-bottom: 26px
  @media (max-width: 786px)
    position: sticky
    top:0
    z-index: 15
.log-size
  /* font-size: 16px;

.login-off, .login-no
  padding: 0 12px
  display: flex
  justify-content: center
  .goBack
    color: #ff6c7a
    background-color: rgba(255, 255, 255, 0.767)
    padding-right: 7px
    padding-left: 7px
    margin-right: 10px
    border-radius: 3px
    font-weight: bold
    cursor: pointer
    white-space: nowrap
    display: flex
    align-items: center
    justify-content: center


.login-off
  font-size: 14px
.a-aim
  transition: all 0.3s ease 0s
  &:hover
    color: #EF2853

.sys
  color: #fff
  position: absolute
  margin: 0 auto
  padding-block-start: 150px
  text-align: center
  width: 100%
  overflow-y: auto
  @media (max-width: 1200px)
    padding-block-start: 0px
  .container
    margin: 0 auto
    padding-right: 15px
    padding-left: 15px
    max-width: 1140px
    width: 100%
    height: 100%
    @media (max-width: 1200px)
      .heading-count
        padding-top: 10px
        padding-bottom: 10px
        h2
          font-size: 36px
    .heading-count
      background-color: rgba(255, 255, 255, 0.411)
      padding-top: 20px
      padding-bottom: 20px
      border-radius: 30px
      h2
        font-size: 42px
        color: #ffffff
        line-height: 30px
        margin-bottom: 15px
        text-shadow: 2px 2px 0 #000, -1px -1px 0 #000, 1px -1px 0 #000, -1px 1px 0 #000, 1px 1px 0 #000
    .heading-count p
      color: #1d5cd5
      font-size: 18px
      font-weight: bold
      text-align: center
      margin: 30px 0 10px
    .card-body-time
      color: #212529
      line-height: 36px
      // font-weight: bold
  /* 定义流动动画 */
  @keyframes gradientFlow
    0%
      background-position: 0 0
    100%
      background-position: 100% 100%

  .btns
    display: grid
    grid-template-columns: repeat(2, 1fr)
    justify-content: center
    gap: 15px
    .time-entry
      color: #fff
      overflow: hidden
      font-size: 18px
      display: inline-block
      padding: 20px
      text-align: center
      font-weight: 800
      color: #fff
      font-size: 18px
      line-height: 22px
      border-radius: 20px
      border: 4px solid  #ff6c7a
      transition: all .3s
      text-shadow: 1px 1px 0 #000, -1px -1px 0 #000, 1px -1px 0 #000, -1px 1px 0 #000, 1px 1px 0 #000
      background: transparent
      &::before
        content: ""
        position: absolute
        top: 0
        left: 0
        width: 100%
        height: 100%
        background: repeating-linear-gradient( -45deg, #ff6c7a, #ff6c7a 5px,  #ff8691 0, #ff8691 8px )
        // animation: gradientFlow 30s linear infinite
        background-size: 200% 200%
        z-index: -1
      span
        font-size: 28px
        font-family: 'Montserrat', sans-serif
        line-height: 30px
        font-weight: 700
        display: block
        color: #fff
        margin-bottom: 10px
    .radio-group
      margin-top: 20px
      padding: 20px 20px
      border-radius: 10px
      font-size: 22px
      border: 2px solid  #fff
      overflow: hidden
      &::before
        content: ""
        position: absolute
        top: 0
        left: 0
        width: 100%
        height: 100%
        background: repeating-linear-gradient( -45deg, #ff6c7a, #ff6c7a 5px,  #ff8691 0, #ff8691 8px )
        background-size: 200% 200%
        z-index: -1
      &:hover
        &::before
          background: repeating-linear-gradient( -45deg, #ff6c7a, #ff6c7a 5px,  #ff8691 0, #ff8691 8px )
          background-size: 200% 200%
          animation: gradientFlow 30s linear infinite
      &.active
        background: repeating-linear-gradient( -45deg, #d64a5a, #d64a5a 5px,  #d96070 0, #d96070 8px)
        background-size: 200% 200%
        animation: gradientFlow 30s linear infinite
    .button-group
      overflow: hidden
      margin: 40px 20px 0
      position: relative
      padding: 10px 20px
      border-radius: 10px
      font-size: 22px
      grid-column: 3 / 1
      border: 2px solid  #fff
      transition: all .3s
      background: repeating-linear-gradient( -45deg, #ff6c7a, #ff6c7a 5px,  #ff8691 0, #ff8691 8px )
      &:hover
        margin: 40px 0 0
</style>
<style lang="sass" scoped>
.game
  background: #efefef
  width: 100dvw
  @media screen and (min-width: 768px)
    display: flex

  &-container
    display: flex
    flex-direction: column
    height: calc(100dvh - 50px)
    overflow-y: auto
    overflow-x: hidden
    @media screen and (min-width: 768px)
      flex: 1
      height: calc(100dvh - 64px)

  &-list
    display: none
    @media screen and (min-width: 768px)
      display: block
      width: 85px
      background: #fff
      li
        background-image: linear-gradient(to bottom, #ffffff 50%, #E9E9E9 100%)
        font-size: 14px
        font-weight: bold
        color: #2e3076
        padding: 1.2rem 0
        display: flex
        flex-direction: column
        align-items: center
        justify-content: center
        width: 100%
        border-top: 1px solid #d8ddd8
        border-bottom: 1px solid #d8ddd8
        cursor: pointer
        transition: all ease .2s
        span
          font-size: 26px
        &:hover
          opacity: .8
          filter: brightness(0.8)

  &-header
    position: sticky
    top: 0
    left: 0
    width: 100%
    padding-left: 10px
    padding-right: 10px
    height: 50px
    background-image: linear-gradient(to left, #3d53c9 0%, #3d5bc9 100%)
    display: flex
    justify-content: space-between
    align-items: center
    color: #fff

    &-info,
    &-main
      display: flex
      align-items: center
      font-size: 15px

    &-info
      &-item
        margin-right: 15px
        display: flex
        align-items: center
        span
          display: block
          margin-right: 5px
        @media screen and (min-width: 768px)
          margin: 0
          &:not(:last-child)
            position: relative
            &::before
              content: ""
              display: block
              position: absolute
              top: 50%
              right: 0
              width: 2px
              height: 90%
              background: #d7d7d7
              transform: translateY(-50%)
      &-item:last-child
        display: none
        @media screen and (min-width: 768px)
          display: flex

    &-ham
      font-size: 26px
      @media screen and (min-width: 768px)
        display: none

    &-logo
      width: 50px
      margin-right: 15px
      img
        width: 100%

    @media screen and (min-width: 768px)
      height: 64px
      &-logo
        width: 87px
      &-info
        &-item
          padding: 0 32px
</style>

<style lang="sass" scoped>
.game
  &-hero
    display: flex
    justify-content: center
    align-items: center
    flex: 1
    background-image: linear-gradient(to right,rgba(255, 255, 255, 0.2),rgba(255, 255, 255, 0.2)),url('@/assets/image/game/fy_game_background.jpg')
    background-size: cover
    background-position: center
    padding: 15px

    .card
      max-width: 400px
      width: 100%
      border-radius: 5px
      box-shadow: 0 2px 5px rgba(0, 0, 0, .5)
      background-color: #fff

      &-header
        display: flex
        align-items: center
        justify-content: space-between
        padding: 10px
        border-bottom: 1px solid #eee
        font-size: 15px
        span:last-child
          font-weight: bold

      &-body
        padding: 10px
        display: flex
        justify-content: space-between
        align-items: center

        &-info
          display: flex
          align-items: center
          img
            width: 60px
            height: 60px
            border-radius: 50%
            margin-right: 10px
          span
            font-size: 14px

        &-time
          padding: 10px
</style>

<style lang="sass" scoped>
.game-data
  display: flex
  flex-wrap: wrap
  @media screen and (min-width: 768px)
    flex-wrap: nowrap
  &-item
    padding: .25rem
    width: calc(50% - 1rem)
    margin: .5rem
    &:nth-child(1),
    &:nth-child(2)
      &::before
        top: 100%
        right: 5%
        width: 90%
        height: 2px
        transform: none
        content: ""
        display: block
        position: absolute
        background: #d7d7d7
        @media screen and (min-width: 768px)
          display: none

    @media screen and (min-width: 768px)
      padding: 4px
      margin: 8px

      &:not(:last-child)
        &::before
          content: ""
          display: block
          position: absolute
          top: 50%
          right: 0
          width: 2px
          height: 90%
          background: #d7d7d7
          transform: translateY(-50%)
    &-header
      display: flex
      align-items: center
      font-weight: bold
      margin-bottom: 8px
      font-size: 14px
      &-icon
        width: 42px
        height: 42px
        display: flex
        justify-content: center
        align-items: center
        font-size: 20px
        border-radius: 5px
        margin-right: .5rem
        background: #d7d7d7
      @media screen and (min-width: 768px)
        font-size: 18px
    &-body
      font-weight: bold
      font-size: 14px
      text-align: center
      span
        font-size: 32px
      @media screen and (min-width: 768px)
        font-size: 20px
        span
          font-size: 42px
</style>

<style lang="sass" scoped>
.menu
  position: fixed
  left: 0
  top: 0
  width: 100%
  height: 100%
  z-index: 99
  background-color: rgba(0, 0, 0, 0.5)
  transform: translateX(-100%)
  transition: transform 0.3s

  @media screen and (min-width: 768px)
    display: none

  &.active
    transform: translateX(0)

  &-container
    background-color: #fff
    width: 270px
    height: 100%

  &-header
    position: relative
    display: flex
    flex-flow: row wrap
    align-items: center
    justify-content: flex-start
    width: 100%
    padding: 20px 15px 15px
    position: relative
    z-index: 0
    color: #fff
    background-image: linear-gradient(to left, #95c93d 0%, #95c93d 100%)

    &::before
      content: ""
      position: absolute
      left: 0
      top: 0
      width: 100%
      height: 100%
      background-image: url('@/assets/image/member/mobile-bg.png')
      background-size: cover
      background-position: center
      opacity: .43
      z-index: 1
      pointer-events: none

    &-time
      border-bottom: 1px solid #fff
      padding-bottom: 15px
      color: #fff
      width: 100%
      font-size: 15px
      font-weight: bold

    &-info
      padding: 15px 0
      color: #fff
      width: 100%
      font-size: 15px
      font-weight: bold
      display: flex
      justify-content: space-between

    &-close
      position: absolute
      top: 8px
      right: 15px
      font-size: 28px
      z-index: 99

  &-list
    &-item
      font-size: 16px
      justify-content: flex-start
      align-items: center
      padding: 1rem 0
      padding-left: 15px
      font-weight: bold
      color: #2e3076
      display: flex
      border-top: 1px solid #d8ddd8
      border-bottom: 1px solid #d8ddd8
      cursor: pointer
      background-image: linear-gradient(to bottom, #ffffff 50%, #E9E9E9 100%)

      span
        display: block
        margin-right: 12px
        font-size: 24px
</style>

<style scoped>
[hidden] {
  display: none !important;
}

.popup {
  position: fixed;
  z-index: 999;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgb(0, 0, 0);
  background-color: rgba(0, 0, 0, 0.4);
  transition: opacity 0.15s linear;
  padding: 0 40px;
}

.dataBody {
  padding: 10px 0;
}

.gameOptionSpan {
  padding: 0 5px;
}

.inputDate>>>.el-input__prefix {
  display: none;
}

.inputDate input {
  padding: 5px;
  margin: 0 1px;
}

.inputDate svg {
  margin: 0 5px 0 0;
  color: #fff;
}

.bulItem input[type='checkbox'] {
  display: none;
}

.bulItem .content {
  display: none;
}

.bulItem input[type='checkbox']:checked~.content {
  display: block;
}

.gameResultName {
  display: flex;
}

.gameResultNameItem {
  padding: 0 5px;
  font-size: 16px;
}

@media screen and (max-width: 768px) {
  .popup {
    padding: 0 10px;
    height: 100%;
  }
}

input {
  margin: 0;
}

.searchRow svg {
  color: #363c4e;
}
</style>

<style>
.el-input__wrapper {
  background-color: #00000000;
  border: 1pxsolid #00000000;
  box-shadow: none;
}
</style>
