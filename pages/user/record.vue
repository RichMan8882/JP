<script lang="ts" setup>
const { queryTransaction } = useTransactionStore()
const { queryWithdrawRecord, queryDepositRecords, queryWalletLog } =
  useWalletStore()
const router = useRouter()
const recordList: any = ref([])
const withdrawRecordList: any = ref([])
const recordType = ref('withdraw')
const siteStore = useSiteStore()
const playerStore = usePlayerStore()
console.log(playerStore.playerInfo)

const { t } = useI18n()
const pledgeSocketStore = usePledgeSocketStore()

const selectWalletId: any = ref('')
const selectWallet: any = ref({})

const walletTypeName = (type: any) => {
  switch (type) {
    case 1:
      return siteStore.siteData.mainWalletType === type
        ? t('主錢包')
        : t('交易錢包')
    case 2:
      return siteStore.siteData.mainWalletType === type
        ? t('主錢包')
        : t('電力錢包')
    case 3:
      return siteStore.siteData.mainWalletType === type
        ? t('主錢包')
        : t('商城錢包')
    case 4:
      return siteStore.siteData.mainWalletType === type
        ? t('主錢包')
        : t('質押錢包')
    default:
      return t('其他資產')
  }
}

onMounted(async () => {
  // 下單紀錄 列表 filter type:5 & type:7
  // console.log('walletLog', pledgeSocketStore.walletLogData)
  // const firstWallet = playerStore.playerInfo.wallet.find(
  //   (item: any) => item.type === siteStore.siteData.mainWalletType
  // )
  // selectWalletId.value = firstWallet.id
  // selectWallet.value = firstWallet
  // const queryWalletLogRes = await queryWalletLog({
  //   walletId: selectWallet.value.id,
  //   page: 1,
  //   limit: 1000,
  //   size: 1000
  // })
  // recordList.value = queryWalletLogRes.data.result

  const queryWithdrawRecordRes = await queryWithdrawRecord({})
  withdrawRecordList.value = queryWithdrawRecordRes.data.result
})

await useAsyncData(async () => {
  // const queryTransactionRes = await queryTransaction({})
  // recordList.value = queryTransactionRes.data
  const queryWithdrawRecordRes = await queryWithdrawRecord({})
  withdrawRecordList.value = queryWithdrawRecordRes.data.result
})

watch(
  () => selectWalletId.value,
  async (newValue) => {
    const walletData = playerStore.playerInfo.wallet.find(
      (item: any) => item.id === newValue
    )
    selectWallet.value = walletData
    console.log(' selectWalletId', selectWallet.value)
    const queryWalletLogRes = await queryWalletLog({
      walletId: selectWallet.value.id,
      page: 1,
      limit: 1000,
      size: 1000
    })
    recordList.value = queryWalletLogRes.data.result
  },
  { deep: true }
)

const selectWalletData = (data: any) => {
  selectWallet.value = data
  console.log('selectWallet.value', selectWallet.value)
}

const changeRecordType = async (type: string) => {
  recordType.value = type
  console.log('000')
  console.log(type)

  switch (type) {
    case 'transaction':
      console.log(111)

      const firstWallet = playerStore.playerInfo.wallet.find(
        (item: any) => item.type === siteStore.siteData.mainWalletType
      )
      selectWalletId.value = firstWallet.id
      selectWallet.value = firstWallet
      const queryWalletLogRes = await queryWalletLog({
        walletId: selectWallet.value.id
      })
      console.log(222)

      recordList.value = queryWalletLogRes.data.result
      console.log(recordList.value, 'recordList.value ')

      return
    case 'deposit':
      const queryDepositRecordsRes = await queryDepositRecords()
      recordList.value = queryDepositRecordsRes.data.result
      return
    case 'withdraw':
      const queryWithdrawRecordRes = await queryWithdrawRecord({})
      console.log(queryWithdrawRecordRes.data)
      withdrawRecordList.value = queryWithdrawRecordRes.data.result
  }
}

const showSectionType = (type: string) => {
  const bank = siteStore.siteData.depositOptions.bank.find((item) => {
    return item === type
  })
  if (!bank) {
    return true
  } else {
    return false
  }
}

const transactionRecordType = (type: any) => {
  const transactionTypeList = [
    {
      value: 0,
      label: t('儲值')
    },
    {
      value: 1,
      label: t('提領')
    },
    {
      value: 2,
      label: t('轉帳')
    },
    {
      value: 3,
      label: t('贈送')
    },
    {
      value: 4,
      label: t('活動')
    },
    {
      value: 5,
      label: t('結算')
    },
    {
      value: 6,
      label: t('系統')
    },
    {
      value: 7,
      label: t('下注')
    }
  ]
  const data = siteStore.siteData.walletLogType.find(
    (item) => item.type === type
  )
  if (selectWallet.value.type !== 1 && type === 7) {
    return '質押'
  } else {
    return data ? data.label : t('未知')
  }
}
const applyStatus = (status: any) => {
  const statusList = [
    {
      value: 0,
      label: '未審核'
    },
    {
      value: 1,
      label: '已完成'
    },
    {
      value: 2,
      label: '已駁回'
    }
  ]
  const data = statusList.find((item) => item.value === status)
  return data ? data.label : t('未知')
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
const hiddenAccountNo = (value) => {
  if (value) {
    return value.replace(/^.{4}/, '****')
  } else {
    return ''
  }
}
const formatMemo = (value: any) => {
  // const str = value.memo
  //   .replace('會員投注', '會員購買')
  //   .replace('投注-', '購買-')
  //   .replace('下注', '購買')
  //   .replace('輸贏', '盈虧')
  //   .replace('投注本金', '購買本金')
  if (value.includes('[漲跌-上期]')) {
    const pattern = /(\d{12}).*?「([^」]+)」/
    let str = value.match(pattern)

    // str = '訂單編號：' + str[1] + '\r\n' + str[2].replace('高', '購置').replace('低', '放置')
    str =
      '[訂單結果] ' +
      '訂單編號：' +
      str[1] +
      ' ' +
      '\r\n' +
      str[2].replace('比特幣/美元-漲', '結果：出荷').replace('比特幣/美元-跌', '結果：返品・交換')
    return str
  } else if (value.includes('[投注成功]')) {
    let str = value
      .replace('投注成功', '訂單成功')
      .replace('期別', '訂單編號')
      .replace('高', '出荷')
      .replace('低', '返品・交換')
      .replace('漲', '出荷')
      .replace('跌', '返品・交換')

    const pattern =
      /\[訂單成功\]\s+.*?\s+投注-(.*?)\s+訂單編號-(\d+)\s+選項-(.*?)\s+金額-\d+/
    str = str.match(pattern)
    str =
      '[訂單成功] ' +
      '訂單編號：' +
      str[2] +
      '\r\n' +
      str[3].replace('漲', '出荷').replace('跌', '返品・交換')
    return str
  } else {
    return value
  }
}
</script>

<template>
  <div class="pages">
    <headerTop />
    <div class="ul-cart-container" :class="recordType == 'transaction' ? 'ck-container' : ''">
      <div class="tag">
        <h2>
          <i class="fa-solid fa-chart-column"></i>
          {{ $lang('資金紀錄') }} -
          {{
            t(recordType == 'transaction'
              ? '訂單記錄'
              : recordType == 'withdraw'
                ? '提領紀錄'
                : '')
          }}
        </h2>
      </div>
      <div class="changeType">
        <button type="button" class="buttonWhite" :class="recordType === 'transaction' ? 'active' : ''"
          @click="changeRecordType('transaction')">
          {{ $lang('訂單記錄') }}
        </button>
        <button type="button" class="buttonWhite" :class="recordType === 'withdraw' ? 'active' : ''"
          @click="changeRecordType('withdraw')">
          {{ $lang('提領紀錄') }}
        </button>
      </div>
      <div class="cart-top">
        <div v-if="recordType === 'transaction'" class="table-responsive">
          <div v-if="playerStore.playerInfo.wallet.length > 1" class="walletBox">
            <span class="">{{ $lang('錢包') }}</span>
            <select v-model="selectWalletId" class="selectStyle">
              <option v-for="item in playerStore.playerInfo.wallet" :key="item.id" :value="item.id"
                @click="(selectWalletId = item.id), (selectWallet = item)">
                {{ walletTypeName(item.type) }}
              </option>
            </select>
          </div>
          <table class="ul-cart-table">
            <thead>
              <tr>
                <th>{{ $lang('時間') }}</th>
                <th>{{ $lang('內容') }}</th>
                <th style="padding-right: 0;">{{ $lang('金額') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, index) in recordList" :key="index">
                <!-- 時間 -->
                <td>
                  <p>
                    {{ formatDate(item.createdAt) }}
                  </p>
                </td>
                <!-- 內容 -->
                <td>
                  <div v-if="item.type !== 5">
                    <!-- {{ $lang('類型') }}: {{ transactionRecordType(item.type) }} -->
                  </div>
                  <!-- <div>{{ $lang('備註') }}:{{ item.memo }}</div> -->
                  <div style="white-space: pre">
                    {{ t(formatMemo(item.memo)) }}
                  </div>
                </td>

                <!-- 金額 -->
                <td>
                  <p>
                    $ {{ new Intl.NumberFormat('zh-TW').format(item.amount) }}</p>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
        <div v-if="recordType === 'withdraw'" class="table-responsive">
          <table class="ul-cart-table">
            <thead>
              <tr>
                <th>{{ $lang('時間') }}</th>
                <th>{{ $lang('內容') }}</th>
                <th>{{ $lang('金額') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, index) in withdrawRecordList">
                <!-- 時間 -->
                <td>
                  <p>
                    {{ formatDate(item.createdAt) }}
                  </p>
                </td>

                <!-- 內容 -->
                <td v-if="showSectionType(item.bankData.bankName)">
                  <div>{{ $lang('交易所') }}:{{ item.bankData.bankName }}</div>
                  <div>
                    {{ $lang('錢包地址') }}:{{ item.bankData.accountNo }}
                  </div>
                  <div>{{ $lang('狀態') }}:{{ applyStatus(item.status) }}</div>
                  <div>
                    {{ $lang('手續費') }}: ¥
                    {{ new Intl.NumberFormat('zh-TW').format(item.fee) }}
                  </div>
                </td>
                <td v-else>
                  <div>{{ $lang('銀行') }}:{{ item.bankData.bankName }}</div>
                  <div>{{ $lang('分行') }}:{{ item.bankData.branch }}</div>
                  <div>{{ $lang('戶名') }}:{{ item.bankData.account }}</div>
                  <div>{{ $lang('帳號') }}:{{ item.bankData.accountNo }}</div>
                  <div>{{ $lang('狀態') }}:{{ applyStatus(item.status) }}</div>
                  <div>
                    {{ $lang('手續費') }}: ¥
                    {{ new Intl.NumberFormat('zh-TW').format(item.fee) }}
                  </div>
                </td>
                <!-- 金額 -->
                <td>
                  <p>
                    ¥ {{ new Intl.NumberFormat('zh-TW').format(item.amount) }}
                  </p>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
    <indexFooter />
  </div>
</template>

<style scoped lang="sass">
.pages
  box-sizing: border-box
  @media screen and (max-width: 992px)
    padding: 0
</style>

<style scoped lang="sass">
.tag
  padding-top: 123px
  text-align: center
  font-weight: 600
  @media (max-width: 992px)
    padding-top: 70px
  h2
    font-size: 1.375rem
    color: #fff
    padding: 8px 0
    background: repeating-linear-gradient( -45deg, #ff6c7a, #ff6c7a 5px,  #ff8691 0, #ff8691 8px )
    .fa-solid
      color: rgba(255, 255, 255, 0.5)
.ul-cart-container
  min-height: 100vh
  .cart-top
    margin: clamp(40px, 4.2vw, 80px) clamp(15px, 19.44vw, 370px)
    .ul-cart-table
      width: 100%
      text-align: -webkit-match-parent
      thead
        font-weight: 700
        font-size: clamp(17px, 1.05vw, 20px)
        padding-bottom: 17px
        border-bottom: 1px solid rgba(0, 0, 0, 0.2)
      tr
        border-bottom: 1px solid rgba(0, 0, 0, 0.2)
        &:last-child
          border-bottom: none
      th
        padding-bottom: 20px
        padding-right: clamp(30px, 4.73vw, 90px)
        word-spacing: normal
        min-width: 150px
      td
        padding-top: clamp(18px, 1.47vw, 28px)
        padding-bottom: clamp(18px, 1.47vw, 28px)
        padding-right: clamp(30px, 1.47vw, 28px)
        p
          margin-top: 0
          margin-bottom: 1rem
.changeType
  display: flex
  justify-content: center
  gap: 15px
.buttonWhite
  margin: 30px 0 30px
  width: 140px
  height: 40px
  background-color: #fff
  // border-radius: 5px
  text-align: center
  cursor: pointer
  font-size: 16px
  outline: none
  color: #ff6c7a
  border-radius: 10px
  border: 2px solid #ff6c7a
  transition: all .3s ease
  @media screen and (max-width: 768px)
    width: 40%
  &:hover
    background-color: #ff6c7a
    color: #fff
@media screen and (max-width: 1140px)
  .cart-top
    margin: clamp(40px, 4.2vw, 80px) 30px !important
@media screen and (max-width: 992px)
  .cart-top
    margin: clamp(40px, 4.2vw, 80px) 15px !important
@media screen and (max-width: 768px)
  .cart-top
    margin: clamp(40px, 4.2vw, 80px) 0px !important
</style>

<style scoped lang="sass">
.walletBox
  padding: 10px 0
  select
    margin-left: 10px
.selectStyle
  height: 32px
  padding: 2px 15px
  font-size: 19px
  line-height: 32px
  font-weight: 400
  background-color: #fff
  border: 1px solid #000
  border-radius: 4px
  width: 130px !important
.formSection
  margin: 20px 0 0 0
  width: 100%
  height: 58vh
  position: relative
  border-radius: 5px
  font-size: 16px
  overflow-y: auto
  @media screen and (max-width: 768px)
    height: 48vh
  .formContent
    width: 100%
</style>

<style scoped lang="sass">
.title
  background-color: #c8a375
  width: 100%
  margin: 0 auto 20px
  border-radius: 50px
  line-height: 40px
  height: 40px
  font-size: 20px
  padding: 0 30px
  font-weight: bold
  color: #fff
  cursor: pointer
  @media screen and (min-width: 768px)
    padding: 0 30px
    line-height: 55px
    height: 55px
    font-size: 24px
    width: calc(100% - 30px)
.buttonWhGreen
  width: 120px
  height: 40px
  line-height: 38px
  text-align: center
  color: #fff
  background-color: #c8a375
  border: none
  border-radius: 50px
  font-size: 19px
  outline: 0
  border: solid 1px #fff
  margin: 30px auto
  display: block
  &:hover
    opacity: 0.8
  @media screen and (max-width: 768px)
    width: 135px
  svg
    padding: 0 10px 0 0
.active
  background-color: #ff6c7a
  color: #FFF
</style>
