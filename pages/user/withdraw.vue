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
    memo: `${t('用戶操作')} ${walletTypeName(fromData)} ${t('劃轉至')} ${walletTypeName(toData)} ,${t('數量')}: ${transferWalletData.value.amount}`
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
    <headerTop />
    <div class="tag">
      <h2><i class="fa-solid fa-wallet"></i>
        {{ $lang("我的錢包") }}</h2>
    </div>
    <div class=" inf-bg">
      <div v-if="playerStore.playerInfo.bankInfo !== null" class="formSection">
        <div class="changeType">
          <button type="button" class="buttonWhite" :class="selectType === 'withdraw' ? 'active' : ''"
            @click="selectType = 'withdraw'">
            {{ $lang('提領') }}
          </button>
          <button type="button" class="buttonWhite" :class="selectType === 'transfer' ? 'active' : ''"
            @click="selectType = 'transfer'">
            {{ $lang('劃轉') }}
          </button>
        </div>
        <div v-if="selectType === 'withdraw'" class="formContent">
          <form action="" class="ul-contact-form">
            <!--  銀行  -->
            <div class="form-group">
              <div class="position-relative">
                <label style="font-weight: bold;font-size: 18px;">{{ $lang('銀行') }}</label>
                <input type="text" name="acc"
                  :value="playerStore.playerInfo.bankInfo.bankName + ' ' + playerStore.playerInfo.bankInfo.branch"
                  class="form-control" readonly>

              </div>
            </div>
            <!-- 戶名 -->
            <div class="form-group">
              <div class="position-relative">
                <label style="font-weight: bold;font-size: 18px;">{{ $lang('戶名') }}</label>
                <input type="text" name="name" :value="playerStore.playerInfo.bankInfo.account" class="form-control"
                  readonly>

              </div>
            </div>
            <!-- 我的資產 -->
            <div class="form-group">
              <div class="position-relative">
                <label style="font-weight: bold;font-size: 18px;">{{ $lang('我的資產(主錢包)') }}</label>
                <input type="text" name="name" :value="new Intl.NumberFormat('zh-TW').format(mainBalance)"
                  class="form-control" readonly>
              </div>
            </div>
            <!-- 帳號 -->
            <div class="form-group">
              <div class="position-relative">
                <label style="font-weight: bold;font-size: 18px;">{{ $lang('帳號') }}</label>
                <input type="text" name="name" v-model="playerStore.playerInfo.bankInfo.accountNo" class="form-control"
                  readonly>
              </div>
            </div>
            <!-- 金額 -->
            <div class="form-group">
              <div class="position-relative">
                <label style="font-weight: bold;font-size: 18px;">{{ $lang('金額') }}</label>
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
                <label style="font-weight: bold;font-size: 18px;">{{ $lang('交易密碼') }}</label>
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
                <label style="font-weight: bold;font-size: 18px;">{{ $lang('轉出錢包') }}</label>
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
                <label style="font-weight: bold;font-size: 18px;">{{ $lang('轉入錢包') }}</label>
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
                <label style="font-weight: bold;font-size: 18px;">{{ $lang('數量') }}</label>
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
.inf-bg
  margin-top: 50px
  padding: 0 clamp(15px, 3.15vw, 60px) clamp(40px, 4.2vw, 80px)
  @media (max-width: 992px) 
    padding: 0
  .formSection
    border-radius: clamp(15px, 1.1vw, 30px)  
    background-color: white
    border: 2px solid #ff6c7ab9
    margin: 0 clamp(15px, 19.71vw, 375px)
    padding: clamp(60px, 6.31vw, 70px) clamp(20px, 2.1vw, 40px)
    box-shadow: 0px -10px 40px rgba(0, 0, 0, 0.15)
    @media screen and (max-width: 991px) 
      margin: 0 clamp(15px, 11.71vw, 375px)
    @media screen and (max-width: 767px) 
      margin: clamp(15px, 3.15vw, 375px)
    .changeType
      display: flex
      justify-content: flex-start
      padding-bottom: 20px
      gap: clamp(15px, 1.58vw, 30px)
      .buttonWhite
        padding: 8px 35px
        border-radius: 10px
        color: #ff6c7a
        border: 2px solid #ff6c7a
        &:hover    
          color: white
          background-color: #ff6c7a
        &.active
          color: white
          background-color: #ff6c7a
  .ul-contact-form input, .ul-contact-form textarea 
    display: block
    height: clamp(45px, 2.94vw, 56px)
    border: #ff6c7aaa 1px solid
    background-color: #ff6c7a22
    padding: 0 clamp(15px, 1.58vw, 10px)
    padding-right: clamp(30px, 3.15vw, 60px)
    max-width: 100%
    width: 100%
    border-radius: 10px
  .form-control:focus 
    color: #212529
    background-color: #fff
    border-color: #ff6c7a88
    outline: 0
    box-shadow: 0 0 .25rem #ff6c7a55
  select
    border: #ff6c7aaa 1px solid
    background-color: #ff6c7a22
    border-radius: 10px
    padding: 8px 20px
  .form-group
    margin-bottom: clamp(15px, 1.58vw, 30px)
    .position-relative
      position: relative
      .eyes
        position: absolute
        right: 10px
        bottom: 10px
        cursor: pointer
  .ul-contact-form-container
    @media screen and (max-width: 992px) 
      margin: 20px 5px
      padding: 15px
  .ul-contact-form-container__title
    color: #ff6c7a
    text-align: center
    font-weight: 600
    font-size: clamp(22px, 2.52vw, 35px)
  .ul-contact-form .update_password 
    flex-shrink: 0
    background-color: #ff6c7a
    height: clamp(45px, 2.94vw, 56px)
    color: white
    padding: 0 20px
    font-weight: 400
    text-transform: uppercase
    width: 100%
    border-radius: 10px
    display: flex
    align-items: center
    justify-content: center
    cursor: pointer
.buttonWhGreen
  padding: 8px 45px
  border-radius: 10px
  color: white
  border: 2px solid #ff6c7a
  background: repeating-linear-gradient( -45deg, #ff6c7a, #ff6c7a 5px,  #ff8691 0, #ff8691 8px )
  transition: all .2s ease-in-out
  &:hover    
    transform: scale(1.05)
</style>
