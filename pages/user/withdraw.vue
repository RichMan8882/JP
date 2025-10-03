<script lang="ts" setup>
const { t } = useI18n()
const router = useRouter()
const playerStore = usePlayerStore()
const siteStore = useSiteStore()
const { withdraw, transfer } = useWalletStore()
const withdrawAmount = ref(0)
const tPwd = ref('')
const withdrawFee = computed(() => withdrawFeeCount())
const selectType = ref('withdraw')
const showSectionType = ref(true)
await useAsyncData(async () => {
  if (siteStore.siteData.smsVerify === 3) {
    if (!playerStore.playerInfo.smsVerify) {
      navigateTo('/user/smsVerify')
    }
  }
  if (!playerStore.playerInfo?.bankInfo) {
    navigateTo('/user/bank')
  }

  if (playerStore.playerInfo.bankInfo.status !== 1) {
    navigateTo('/user/bank')
  }
})
// methods
const goWithdraw = async () => {
  console.log(withdrawAmount.value, siteStore.siteData.minWithdrawAmount,
    siteStore.siteData.maxWithdrawAmount, '321');

  if (
    withdrawAmount.value < siteStore.siteData.minWithdrawAmount ||
    withdrawAmount.value > siteStore.siteData.maxWithdrawAmount
  ) {
    ElNotification({
      title: `${t('提款金額不符合規定')}`,
      type: 'error',
      duration: 1000,
      showClose: false
    })
    return
  }
  if (siteStore.siteData.transactionPasswordRequired) {
    if (!tPwd.value) {
      ElNotification({
        title: `${t('請輸入交易密碼')}`,
        type: 'error',
        duration: 1000,
        showClose: false
      })
      return
    }
  }
  console.log('withdrawAmount.value', withdrawAmount.value)
  console.log('Number(mainBalance.value)', Number(mainBalance.value))
  if (withdrawAmount.value > Number(mainBalance.value)) {
    ElNotification({
      title: `${t('可提領餘額不足')}`,
      type: 'error',
      duration: 1000,
      showClose: false
    })
    return
  }
  const withdrawRes = await withdraw(
    siteStore.siteData.transactionPasswordRequired
      ? {
        amount: JSON.stringify(withdrawAmount.value),
        password: tPwd.value,
        walletType: siteStore.siteData?.mainWalletType
      }
      : {
        amount: JSON.stringify(withdrawAmount.value),
        walletType: siteStore.siteData?.mainWalletType
      }
  )
  if (withdrawRes.success) {
    ElNotification({
      title: `${t('提款成功')}`,
      type: 'success',
      duration: 1000
    })
    withdrawAmount.value = 0
    tPwd.value = ''
    await playerStore.fetchInfo()
  }
}
const withdrawFeeCount = () => {
  const fee =
    withdrawAmount.value * siteStore.siteData.withdrawFeeRatio +
    siteStore.siteData.withdrawFeeFixed
  return fee < 1 ? 0 : new Intl.NumberFormat().format(fee)
}
const hiddenAccountNo = (value) => {
  if (value) {
    return value.replace(/^.{4}/, '****')
  } else {
    return ''
  }
}
onMounted(() => {
  if (playerStore.playerInfo.bankInfo !== null) {
    const bank = siteStore.siteData.depositOptions.bank.find((item) => {
      return item === playerStore.playerInfo.bankInfo.bankName
    })
    if (!bank) {
      showSectionType.value = false
    } else {
      showSectionType.value = true
    }
  }
})

const walletTypeName = (item: any) => {
  const type = item.type
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

const transferWalletData = ref({
  from: '',
  to: '',
  amount: ''
})

const transferBtn = ref(false)

const goTransfer = async () => {
  transferBtn.value = false
  const fromData = playerStore.playerInfo.wallet.find(
    (item: any) => item.id === transferWalletData.value.from
  )
  const toData = playerStore.playerInfo.wallet.find(
    (item: any) => item.id === transferWalletData.value.to
  )
  if (!fromData) {
    ElNotification({
      title: `${t('請選擇轉出錢包')}`,
      type: 'error',
      duration: 1000
    })
    return
  }
  if (!toData) {
    ElNotification({
      title: `${t('請選擇轉入錢包')}`,
      type: 'error',
      duration: 1000
    })
    return
  }
  const transferRes = await transfer({
    fromWalletId: transferWalletData.value.from,
    toWalletId: transferWalletData.value.to,
    amount: JSON.stringify(transferWalletData.value.amount),
    memo: `用戶操作 ${walletTypeName(fromData)} 劃轉至 ${walletTypeName(toData)} ,數量: ${transferWalletData.value.amount}`
  })
  if (transferRes.success) {
    ElNotification({
      title: `${t('轉帳成功')}`,
      type: 'success',
      duration: 1000
    })
    await playerStore.fetchInfo()
    transferWalletData.value.amount = 0
  }
}

const mainBalance = computed(() => {
  const mainWallet = playerStore.playerInfo.wallet.find(
    (item: any) => item.type === siteStore.siteData?.mainWalletType
  )
  return mainWallet ? mainWallet.balance : 0
})

watch(
  () => transferWalletData.value.from,
  (newValue) => {
    console.log('transferWalletData.value', transferWalletData.value.from)
    const toWalletList = playerStore.playerInfo.wallet.filter(
      (item: any) => item.id !== transferWalletData.value.from
    )
    transferWalletData.value.to = toWalletList[0].id
  },
  { deep: true }
)
watch(
  () => transferWalletData.value.amount,
  (newValue) => {
    if (newValue > 0) {
      transferBtn.value = true
    } else {
      transferBtn.value = false
    }
  },
  { deep: true }
)
</script>

<template>
  <div class="pages">
    <div class="tag">
      <i class="fa-solid fa-wallet" style="color: #0069a8;"></i>
      我的錢包
    </div>
    <div class="inf-bg">
      <div v-if="playerStore.playerInfo.bankInfo !== null" class="formSection">
        <div class="changeType">
          <button type="button" class="buttonWhite" :class="selectType === 'withdraw' ? 'active' : ''"
            @click="selectType = 'withdraw'">
            {{ $lang('提領') }}
          </button>
          <button type="button" class="buttonWhite" :class="selectType === 'transfer' ? 'active' : ''"
            v-if="playerStore.playerInfo.wallet.length > 1" @click="selectType = 'transfer'">
            {{ $lang('劃轉') }}
          </button>
        </div>
        <div v-if="selectType === 'withdraw'" class="formContent">
          <form action="" class="ul-contact-form">
            <!--  銀行  -->
            <div class="form-group">
              <div class="position-relative">
                <label style="color: rgb(12, 12, 158);font-weight: bold;font-size: 18px;">銀行 ▼</label>
                <input type="text" name="acc"
                  :value="playerStore.playerInfo.bankInfo.bankName + ' ' + playerStore.playerInfo.bankInfo.branch"
                  class="form-control" readonly>

              </div>
            </div>
            <!-- 戶名 -->
            <div class="form-group">
              <div class="position-relative">
                <label style="color: rgb(12, 12, 158);font-weight: bold;font-size: 18px;">戶名 ▼</label>
                <input type="text" name="name" :value="playerStore.playerInfo.bankInfo.account" class="form-control"
                  readonly>

              </div>
            </div>
            <!-- 我的資產 -->
            <div class="form-group">
              <div class="position-relative">
                <label style="color: rgb(12, 12, 158);font-weight: bold;font-size: 18px;">我的資產(主錢包) ▼</label>
                <input type="text" name="name" :value="new Intl.NumberFormat('zh-TW').format(mainBalance)"
                  class="form-control" readonly>
              </div>
            </div>
            <!-- 帳號 -->
            <div class="form-group">
              <div class="position-relative">
                <label style="color: rgb(12, 12, 158);font-weight: bold;font-size: 18px;">帳號 ▼</label>
                <input type="text" name="name" v-model="playerStore.playerInfo.bankInfo.accountNo" class="form-control"
                  readonly>
              </div>
            </div>
            <!-- 金額 -->
            <div class="form-group">
              <div class="position-relative">
                <label style="color: rgb(12, 12, 158);font-weight: bold;font-size: 18px;">金額 ▼</label>
                <input type="number" v-model="withdrawAmount" class="form-control">
              </div>
              <div>
                <span style="font-size: 12px;color: #8d1515;">{{ $lang('限制金額') }}：{{
                  new Intl.NumberFormat('zh-TW').format(
                    siteStore.siteData.minWithdrawAmount
                  )
                }}
                  ~
                  {{
                    new Intl.NumberFormat('zh-TW').format(
                      siteStore.siteData.maxWithdrawAmount
                    )
                  }}</span>&nbsp
                <span style="font-size: 12px;color: rgb(47, 169, 88);">{{ $lang('手續費') }}：{{
                  withdrawAmount > 0 ? withdrawFee : 0
                }}</span>
              </div>
            </div>

            <div class="form-group" style="padding-top: 10px;" v-if="siteStore.siteData.transactionPasswordRequired">
              <div class="position-relative">
                <label style="color: rgb(12, 12, 158);font-weight: bold;font-size: 18px;">交易密碼 ▼</label>
                <input type="password" v-model="tPwd" v-trim-input name="new_password" autocomplete="off"
                  class="form-control">
              </div>
            </div>
            <button type="button" class="buttonWhGreen" @click="goWithdraw">
              {{ $lang('送出') }}
            </button>
          </form>

        </div>
        <!--  -->
        <div class="formContent" v-if="selectType === 'transfer'">
          <form action="" class="ul-contact-form">
            <div class="form-group" style="padding-top: 10px;">
              <div class="position-relative">
                <label style="color: rgb(12, 12, 158);font-weight: bold;font-size: 18px;">轉出錢包 ▼</label>
                <div class="form-control">
                  <select v-model="transferWalletData.from" style="width: 100%;">
                    <option v-for="item in playerStore.playerInfo.wallet" :key="item.id" :value="item.id"
                      @click="transferWalletData.from = item.id">
                      {{ walletTypeName(item) }} ($
                      {{ new Intl.NumberFormat('zh-tw').format(item.balance) }})
                    </option>
                  </select>
                </div>
              </div>
            </div>
            <div class="form-group" style="padding-top: 10px;">
              <div class="position-relative">
                <label style="color: rgb(12, 12, 158);font-weight: bold;font-size: 18px;">轉入錢包 ▼</label>
                <div class="form-control">
                  <select v-model="transferWalletData.to" style="width: 100%;">
                    <option v-for="item in playerStore.playerInfo.wallet" :key="item.id" :value="item.id"
                      @click="transferWalletData.from = item.id">
                      {{ walletTypeName(item) }} ($
                      {{ new Intl.NumberFormat('zh-tw').format(item.balance) }})
                    </option>
                  </select>
                </div>
              </div>
            </div>
            <div class="form-group" style="padding-top: 10px;">
              <div class="position-relative">
                <label style="color: rgb(12, 12, 158);font-weight: bold;font-size: 18px;">數量 ▼</label>
                <input v-model="transferWalletData.amount" v-trim-input type="number" class="form-control"
                  style="padding-right:10px;" />
              </div>
            </div>
            <button type="button" class="buttonWhGreen" @click="goTransfer">
              {{ $lang('送出') }}
            </button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped lang="sass">
.pages
  box-sizing: border-box
  @media screen and (max-width: 768px)
    padding: 0 10px 10px
</style>

<style scoped lang="sass">
.inf-bg
  margin-top: 50px
  background-image: url(@/assets/image/index/bank-b01.png)
  background-size: cover
  padding: 0 clamp(15px, 3.15vw, 60px) clamp(40px, 4.2vw, 80px)
    
</style>

<style scoped lang="sass">
.tag
  margin: 0 clamp(15px, 3.15vw, 60px)
  background-color: #b8e6fcc0
  border-radius: clamp(20px, 2.1vw, 40px)
  padding-top: 3px
  margin-top: 10px
  text-align: center
  font-weight: 600
  font-size: clamp(22px, 1.58vw, 30px)
  color: var(--black)
  line-height: 1.3
.formSection
  background-color: white
  margin: 0 clamp(15px, 19.71vw, 375px)
  padding: clamp(60px, 6.31vw, 70px) clamp(20px, 2.1vw, 40px)
  box-shadow: 0px -10px 40px rgba(0, 0, 0, 0.15)
  @media screen and (max-width: 991px) 
    margin: 0 clamp(15px, 11.71vw, 375px)
  @media screen and (max-width: 767px) 
    margin: clamp(15px, 3.15vw, 375px)
  .ul-contact-form-container 
    background-color: white
    margin: 0 clamp(15px, 19.71vw, 375px)
    padding: clamp(60px, 6.31vw, 70px) clamp(20px, 2.1vw, 40px)
    box-shadow: 0px -10px 40px rgba(0, 0, 0, 0.15)
  .ul-contact-form input, .ul-contact-form textarea, .ul-contact-form .form-control
    display: block
    height: clamp(45px, 2.94vw, 56px)
    border: #F8E6E2 1px solid
    background-color: #F8E6E2
    padding: 0 clamp(15px, 1.58vw, 10px)
    padding-right: clamp(30px, 3.15vw, 60px)
    max-width: 100%
    width: 100%
    border-radius: 10px
    outline: none
    position: relative
  select
    position: absolute
    outline: none
    left: 0
    right: 0
    top: 0
    bottom: 0
    background: none
    /* 移除默认样式 */
    appearance: none
    -webkit-appearance: none
    -moz-appearance: none
    
    /* 自定义样式 */
    width: 100%
    padding: 8px 35px 8px 15px
     /* 右侧留出箭头空间 */    
    /* 自定义箭头 */
    background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="%23333"><path d="M7 10l5 5 5-5z"/></svg>')
    background-repeat: no-repeat
    background-position: right 8px center /* 调整这里控制位置 */
    background-size: 16px
  .form-control:focus 
    color: #212529
    background-color: #fff
    border-color: #86b7fe
    outline: 0
    box-shadow: 0 0 0 .25rem rgba(13, 110, 253, .25)
  .form-group
    margin-bottom: clamp(15px, 1.58vw, 30px)
  .ul-contact-form-container__title
    text-align: center
    font-weight: 600
    font-size: clamp(22px, 2.52vw, 35px)
  .ul-contact-form button 
    flex-shrink: 0
    background-color: #EF2853
    height: clamp(45px, 2.94vw, 56px)
    color: white
    padding: 0 20px
    font-weight: 400
    text-transform: uppercase
    width: 100%
    border-radius: 10px
    transition: all .2s ease
    &:hover 
      color:  #EF2853
      background-color: #000
.buttonWhite
  margin: 0 30px 30px 0
  width: 100px
  height: 40px
  cursor: pointer
  background-color: rgb(230, 230, 230)
  // border-radius: 5px
  outline: none
  text-align: center
  line-height: 40px
  cursor: pointer
  font-size: 16px
  color: #0d6efd
  transition: all .3s ease
  @media screen and (max-width: 768px)
    width: 135px
  &:hover
    color: red
</style>
