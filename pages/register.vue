<script lang="ts" setup>
const { t } = useI18n()
const router = useRouter()
const referrerCodeCookiee = useCookie('referrerCode') as any
const siteStore = useSiteStore()
const signupData = ref({
  account: '',
  password: '',
  transactionPassword: '',
  username: '',
  referrerCode: '',
  countryCode: siteStore.siteData.localsOptions[0],
  mobile: '',
  socialPlatform: '',
  socialId: '',
  additionalInfo: {}
})
const additionalInfo1Value = ref('Line')
const showReferrerInput = ref(false)
const verifyPassword = ref('')
const verifyTransactionPassword = ref('')
const isChecked = ref(true)
const recaptchaCode = ref('')
const recaptchaCheckFunction = ref(null)
const passwordEyes = ref(false)
const tPasswordEyes = ref(false)
const { signup } = useAuthStore()
const validationStatus = reactive({
  accountValid: false,
  passwordValid: false,
  passwordSame: true,
  transactionPasswordValid: false,
  transactionPasswordSameWithPassword: false,
  transactionPasswordSame: true,
  phoneValid: false
})
const passwordSameRef = ref(null)
const transactionPasswordRef = ref(null)
const transactionPasswordSameRef = ref(null)
const usernameRef = ref(null)
const mobileRef = ref(null)
const socialIdRef = ref(null)
const isFormValid = computed(() => {
  console.log('passwordSameRef.value', passwordSameRef.value)
  console.log(
    'transactionPasswordSameRef.value',
    transactionPasswordSameRef.value
  )

  // 如果有額外欄位資訊，則需要添加額外驗證
  let isValid = validationStatus.accountValid && validationStatus.passwordValid

  if (passwordSameRef.value !== null) {
    isValid = isValid && validationStatus.passwordSame
  }

  if (transactionPasswordRef.value !== null) {
    isValid =
      isValid &&
      validationStatus.transactionPasswordValid &&
      !validationStatus.transactionPasswordSameWithPassword
  }
  if (transactionPasswordSameRef.value !== null) {
    isValid =
      isValid &&
      validationStatus.transactionPasswordSame &&
      !validationStatus.transactionPasswordSameWithPassword
  }
  if (usernameRef.value !== null) {
    isValid = isValid && signupData.value.username
  }
  if (mobileRef.value !== null) {
    isValid = isValid && signupData.value.mobile
  }
  if (signupData.value.mobile) {
    isValid = isValid && validationStatus.phoneValid
  }
  if (socialIdRef.value !== null) {
    isValid = isValid && signupData.value.socialId
  }

  console.log('isFormValid:', isValid) // 添加日志
  return isValid
})

const validateAccount = () => {
  const regex = /^[a-zA-Z0-9]{4,20}$/
  validationStatus.accountValid = regex.test(signupData.value.account)
  checkTransactionPasswordSameWithPassword()
}

const checkPasswordRequired = () => {
  const passwordRegex = /^.{3,}$/
  // const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)[a-zA-Z\d]{3,}$/
  validationStatus.passwordValid =
    passwordRegex.test(signupData.value.password) &&
    signupData.value.password !== signupData.value.account
  if (verifyPassword.value) {
    checkPasswordSame()
  }
  if (signupData.value.transactionPassword) {
    checkTransactionPasswordSameWithPassword()
  }
}

const checkPasswordSame = () => {
  validationStatus.passwordSame =
    signupData.value.password === verifyPassword.value
}

const checkTransactionPasswordRequired = () => {
  const passwordRegex = /^.{3,}$/
  // const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)[a-zA-Z\d]{3,}$/
  validationStatus.transactionPasswordValid = passwordRegex.test(
    signupData.value.transactionPassword
  )
  checkTransactionPasswordSameWithPassword()
  if (verifyTransactionPassword.value) {
    checkTransactionPasswordSame()
  }
}

const checkTransactionPasswordSameWithPassword = () => {
  validationStatus.transactionPasswordSameWithPassword =
    signupData.value.transactionPassword === signupData.value.password ||
    signupData.value.transactionPassword === signupData.value.account
}

const checkTransactionPasswordSame = () => {
  validationStatus.transactionPasswordSame =
    signupData.value.transactionPassword === verifyTransactionPassword.value
}

const showPhoneValid = computed(() => {
  if (signupData.value.countryCode == '+886') {
    return true
  } else {
    return false
  }
})

const checkPhoneValid = () => {
  const regex = /^09\d{8}$/
  if (signupData.value.countryCode == '+886') {
    validationStatus.phoneValid = regex.test(signupData.value.mobile)
  } else {
    validationStatus.phoneValid = !!signupData.value.mobile
  }
}

const checkHepler = async (data) => {
  recaptchaCheckFunction.value = data
}
// methods
const showPassword = (type: boolean) => {
  passwordEyes.value = type
}
const showTranscationPassword = (type: boolean) => {
  tPasswordEyes.value = type
}

const handleRegisterClick = () => {
  console.log('Button clicked, isFormValid:', isFormValid.value)
  if (isFormValid.value) {
    goRegister()
  }
}

const goRegister = async () => {
  console.log('goRegister')
  if (!isChecked.value) {
    ElNotification({
      title: `${t('請勾選同意條款')}`,
      type: 'error',
      duration: 1000,
      showClose: false
    })
    return
  }
  const recaptchaCheck = await recaptchaCheckFunction.value.validate(
    recaptchaCode.value
  )
  if (!recaptchaCheck) {
    ElMessage({
      message: `${t('驗證碼錯誤')}`,
      type: 'error',
      showClose: false
    })
  } else {
    const signupRes = await signup(signupData.value)
    console.log('signupRes', signupRes)
    if (signupRes.success) {
      referrerCodeCookiee.value = ''
      navigateTo('/user')
    }
  }
}

const { isLogin } = useAuthStore()
await useAsyncData(async () => {
  if (!router.currentRoute.value.name.includes('model')) {
    if (isLogin()) {
      navigateTo('/user')
    }
  }
  if (router.currentRoute.value.query.referrer) {
    signupData.value.referrerCode = router.currentRoute.value.query.referrer
  } else if (referrerCodeCookiee.value) {
    signupData.value.referrerCode = referrerCodeCookiee.value
  }
})

await onMounted(() => {
  showReferrerInput.value = !!(
    router.currentRoute.value.query.referrer || referrerCodeCookiee.value
  )
})
</script>

<template>
  <div class="register">
    <!-- <div class="logo">
      <img :src="siteStore?.siteData.logo" />
    </div> -->
    <client-only>
      <div class="register-card">
        <div class="bg-temp">
          <h2 class="register-title">
            {{ $lang('注冊') }}
          </h2>

          <div class="form">
            <div class="form-container">
              <div class="form-left">
                <!-- <div v-if="showReferrerInput" class="form-item">
                <div class="form-title">
                  {{ $lang('推薦碼') }}
                </div>
                <div class="form-main">
                  <div class="form-input">
                    <input v-model="signupData.referrerCode" v-trim-input type="text" class="input_style"
                      :readonly="router.currentRoute.value.query.referrer" />
                  </div>
                </div>
              </div> -->
                <div class="form-item" :class="{
                  active: focusItem === 'account'
                }">
                  <div class="form-title">
                    {{ $lang('帳號') }}
                  </div>
                  <div class="form-main">
                    <div class="form-input">
                      <input v-model="signupData.account" v-trim-input type="text" class="input_style"
                        :placeholder="t('請輸入帳號')" @input="validateAccount" @copy.prevent @paste.prevent
                        @contextmenu.prevent />
                    </div>
                    <div class="tips">
                      <div :class="validationStatus.accountValid
                        ? 'valid-feedback'
                        : 'invalid-feedback'
                        ">
                        <span v-if="validationStatus.accountValid">
                          <i class="fas fa-check"></i>
                        </span>
                        <!-- {{ $lang('需使用4-20位英文或數字') }} -->
                        {{ $lang('至少需使用4位以上數字') }}
                      </div>
                    </div>
                  </div>
                </div>
                <div class="form-item" :class="{
                  active: focusItem === 'password'
                }">
                  <div class="form-title">
                    {{ $lang('密碼') }}
                  </div>
                  <div class="form-main">
                    <div class="form-input">
                      <input v-model="signupData.password" v-trim-input :type="passwordEyes ? 'text' : 'password'"
                        class="input_style" :placeholder="t('請輸入密碼')" @input="checkPasswordRequired" @copy.prevent
                        @paste.prevent @contextmenu.prevent />
                      <span v-if="!passwordEyes" class="eyes" @click="showPassword(true)">
                        <i class="fa-regular fa-eye-slash"></i>
                      </span>
                      <span v-else class="eyes" @click="showPassword(false)">
                        <i class="fa-regular fa-eye"></i>
                      </span>
                    </div>
                    <div class="tips">
                      <div :class="validationStatus.passwordValid
                        ? 'valid-feedback'
                        : 'invalid-feedback'
                        ">
                        <span v-if="validationStatus.passwordValid">
                          <i class="fas fa-check"></i>
                        </span>
                        <!-- {{ $lang('需混合使用 3 個字元以上的大小寫英文或數字') }} -->
                        {{ $lang('需使用3位以上數字') }}
                      </div>
                      <div v-if="signupData.password" :class="signupData.password !== signupData.account
                        ? 'valid-feedback'
                        : 'invalid-feedback'
                        ">
                        <span v-if="signupData.password !== signupData.account">
                          <i class="fas fa-check"></i>
                        </span>
                        {{ $lang('登入密碼不與帳號相同') }}
                      </div>
                    </div>
                  </div>
                </div>
                <div class="form-item" :class="{
                  active: focusItem === 'verifyPassword'
                }">
                  <div class="form-title">
                    {{ $lang('確認密碼') }}
                  </div>
                  <div class="form-main">
                    <div class="form-input">
                      <input ref="passwordSameRef" v-model="verifyPassword" v-trim-input
                        :type="passwordEyes ? 'text' : 'password'" class="input_style" :placeholder="$lang('請再次確認密碼')"
                        @input="checkPasswordSame" @copy.prevent @paste.prevent @contextmenu.prevent />
                      <span v-if="!passwordEyes" class="eyes" @click="showPassword(true)">
                        <i class="fa-regular fa-eye-slash"></i>
                      </span>
                      <span v-else class="eyes" @click="showPassword(false)">
                        <i class="fa-regular fa-eye"></i>
                      </span>
                    </div>
                    <div class="tips">
                      <div v-if="verifyPassword">
                        <div v-if="validationStatus.passwordSame" class="valid-feedback">
                          <i class="fas fa-check"></i> {{ $lang('確認相同') }}
                        </div>
                        <div v-else class="invalid-feedback">
                          <i class="fas fa-times"></i>
                          {{ $lang('與登入密碼不相同') }}
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
                <div v-if="siteStore.siteData.transactionPasswordRequired" class="form-item">
                  <div class="form-title">
                    {{ $lang('提款密碼') }}
                  </div>
                  <div class="form-main">
                    <div class="form-input">
                      <input ref="transactionPasswordRef" v-model="signupData.transactionPassword" v-trim-input
                        :type="tPasswordEyes ? 'text' : 'password'" class="input_style"
                        @input="checkTransactionPasswordRequired" @copy.prevent @paste.prevent @contextmenu.prevent />
                      <span v-if="!passwordEyes" class="eyes" @click="showPassword(true)">
                        <i class="fa-regular fa-eye-slash"></i>
                      </span>
                      <span v-else class="eyes" @click="showPassword(false)">
                        <i class="fa-regular fa-eye"></i>
                      </span>
                    </div>
                    <div class="tips">
                      <div :class="validationStatus.transactionPasswordValid
                        ? 'valid-feedback'
                        : 'invalid-feedback'
                        ">
                        <span v-if="validationStatus.transactionPasswordValid">
                          <i class="fas fa-check"></i>
                        </span>
                        <!-- {{ $lang('需混合使用 3 個字元以上的大小寫英文或數字') }} -->
                        {{ $lang('需使用3位以上數字') }}
                      </div>
                      <div v-if="signupData.transactionPassword" :class="!validationStatus.transactionPasswordSameWithPassword
                        ? 'valid-feedback'
                        : 'invalid-feedback'
                        ">
                        <span v-if="
                          !validationStatus.transactionPasswordSameWithPassword
                        ">
                          <i class="fas fa-check"></i>
                        </span>
                        {{ $lang('交易密碼不可與登入密碼,帳號相同') }}
                      </div>
                    </div>
                  </div>
                </div>
              </div>
              <div class="form-right">
                <div v-if="siteStore.siteData.transactionPasswordRequired" class="form-item">
                  <div class="form-title">
                    {{ $lang('交易密碼確認') }}
                  </div>
                  <div class="form-main">
                    <div class="form-input">
                      <input ref="transactionPasswordSameRef" v-model="verifyTransactionPassword" v-trim-input
                        :type="tPasswordEyes ? 'text' : 'password'" class="input_style"
                        @input="checkTransactionPasswordSame" @copy.prevent @paste.prevent @contextmenu.prevent />
                      <span v-if="!passwordEyes" class="eyes" @click="showPassword(true)">
                        <i class="fa-regular fa-eye-slash"></i>
                      </span>
                      <span v-else class="eyes" @click="showPassword(false)">
                        <i class="fa-regular fa-eye"></i>
                      </span>
                    </div>
                    <div class="tips">
                      <div v-if="verifyTransactionPassword">
                        <div v-if="validationStatus.transactionPasswordSame" class="valid-feedback">
                          <i class="fas fa-check"></i> {{ $lang('確認相同') }}
                        </div>
                        <div v-else class="invalid-feedback">
                          <i class="fas fa-times"></i>
                          {{ $lang('與交易密碼不相同') }}
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
                <div class="form-item" :class="{
                  active: focusItem === 'username'
                }">
                  <div class="form-title">
                    {{ $lang('名稱') }}
                  </div>
                  <div class="form-main">
                    <div class="form-input">
                      <input v-model="signupData.username" v-trim-input type="text" class="input_style"
                        :placeholder="t('請輸入名稱')" />
                    </div>
                    <div class="tips">
                      <div ref="usernameRef" :class="signupData.username
                        ? 'valid-feedback'
                        : 'invalid-feedback'
                        ">
                        <span v-if="signupData.username">
                          <i class="fas fa-check"></i>
                        </span>
                        {{ $lang('必填') }}
                      </div>
                    </div>
                  </div>
                </div>
                <div class="form-item code" :class="{
                  active: focusItem === 'phoneCode'
                }">
                  <div class="form-title">
                    {{ $lang('聯絡方式') }}
                  </div>
                  <div class="form-main">
                    <div class="form-input">
                      <select v-model="signupData.countryCode" @change="checkPhoneValid">
                        <option v-for="item in siteStore.siteData.localsOptions" :key="item">
                          {{ item }}
                        </option>
                      </select>
                      <input v-model="signupData.mobile" v-trim-input :placeholder="t('用於接收簡訊')" type="text"
                        class="input_style" @input="checkPhoneValid" />
                    </div>
                    <div class="tips">
                      <div v-if="showPhoneValid" :class="validationStatus.phoneValid
                        ? 'valid-feedback'
                        : 'invalid-feedback'
                        ">
                        <span v-if="validationStatus.phoneValid">
                          <i class="fas fa-check"></i>
                        </span>
                        {{ $lang('手機號碼開頭須為09，共10碼') }}
                      </div>
                      <div ref="mobileRef" :class="signupData.mobile
                        ? 'valid-feedback'
                        : 'invalid-feedback'
                        ">
                        <span v-if="signupData.mobile">
                          <i class="fas fa-check"></i>
                        </span>
                        {{ $lang('必填') }}
                      </div>
                    </div>
                  </div>
                </div>
                <!-- <div
              class="form-item code"
              :class="{
                active: focusItem === 'socialPlatform'
              }"
            >
              <div class="form-title">
                {{ $lang('聯絡方式') }}
              </div>
              <div class="form-main">
                <div class="form-input">
                  <select v-model="signupData.socialPlatform">
                    <option
                      v-for="item in siteStore.siteData.socialOptions"
                      :key="item"
                    >
                      {{ item }}
                    </option>
                  </select>
                  <input
                    v-model="signupData.socialId"
                    v-trim-input
                    type="text"
                    @focus="focusItem = 'socialPlatform'"
                    @focusout="focusItem = ''"
                  />
                </div>
              </div>
            </div> -->
                <div class="form-item auth" :class="{
                  active: focusItem === 'recaptcha'
                }">
                  <div class="form-title">
                    {{ $lang('驗證碼') }}
                  </div>
                  <div class="form-main">
                    <div class="form-input">
                      <input v-model="recaptchaCode" v-trim-input type="text" :placeholder="t('請輸入驗證碼')"
                        @keyup.enter="setregister" @focus="focusItem = 'recaptcha'" @focusout="focusItem = ''" />
                      <span>
                        <recaptcha style="cursor: pointer; width: 100px" @check-hepler="checkHepler">
                        </recaptcha>
                      </span>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <div class="form-agree">
              <div class="inputCheck">
                <input v-model="isChecked" type="checkbox" />
              </div>
              <div class="text">
                {{
                  $lang(
                    '我已年滿 18 歲，並已閱讀、接受並同意條款和條件、規則、隱私政策、Cookie 政策以及與年齡驗證相關的政策條款和條件'
                  )
                }}
              </div>
            </div>

            <div class="form-btns">
              <button type="button" class="register-btn" @click="navigateTo('/login')">
                {{ $lang('取消') }}
              </button>
              <button type="button" class="login-btn btn-submit" :class="{ disabled: !isFormValid }"
                @click="handleRegisterClick">
                {{ $lang('送出') }}
              </button>
            </div>
          </div>
        </div>
      </div>
    </client-only>
  </div>
</template>

<style scoped lang="sass">
.logo
  position: absolute
  top: 5px
  left: 5px
  width: 100px

  @media (min-width: 768px)
    display: none

  img
    width: 100%

.register
  position: relative
  width: 100dvw
  height: 100dvh
  background: #f5f5f6
  background-size: cover
  background-position: center
  padding: 30px 15px
  overflow-y: auto
  @media (min-width: 768px)
    display: flex
    justify-content: center
    align-items: center
  .register-card
    border-radius: 15px
  &-card
    width: 100%
    max-width: 500px
    // border-radius: 20px
    background-color: rgba(255, 255, 255)
    // , 0.8
    padding: 15px
    .bg-temp
      border: 2px solid #f58194
      width: 100%
      border-radius: 15px
      @media (max-width: 768px)
        padding: 0 15px
    @media (min-width: 768px)
      width: 65%
      max-width: 920px

  &-title
    // font-size: 30px
    // color: #c8a375
    // font-weight: bold
    margin-bottom: 35px
    text-align: center
    font-size: 26px
    font-weight: 500
    color: #f58194
    margin: 10px 0px 35px 0px
    @media (min-width: 768px)
      font-size: 42px
      margin-bottom: 55px
</style>

<style scoped lang="sass">
.form

  &-container
    @media (min-width: 768px)
      display: flex
      justify-content: space-between
      gap: 35px
      max-width: 90%
      margin: 0 auto

    .form-left,
    .form-right
      flex: 1

  &-item
    color: #666666
    margin-bottom: 5px

    &.auth
      .form-input
        position: relative
        input
          padding-left: 120px

        span
          right: none
          left: 15px
          pointer-events: none

    &.code
      .form-input
        input
          padding-left: 120px
        select
          position: absolute
          top: 50%
          left: 15px
          transform: translateY(-50%)
          z-index: 10
          padding: 5px 0
          border: 1px solid #666666
          color: #666666 !important

    @media (min-width: 768px)
      // max-width: 330px
      margin: 0 auto 5px

    &.active
      color: #c8a375
      .form-input
        input
          border-color: #c8a375 !important
        span
          color: #c8a375


  &-title
    font-size: 16px
    transition: all 0.3s

    @media (min-width: 768px)
      font-size: 18px

  &-btns
    display: flex
    justify-content: center
    align-items: center
    gap: 20px
    margin-bottom: 40px

    button
      width: 30%
      background:  #f58194
      color: #FFF
      border: none
      border-radius: 10px
      font-size: 1.4375rem
      font-weight: 400
      padding: 10px 0px
      cursor: pointer
      outline: none
      transition: all 0.3s
      &:hover
        opacity: 0.8
      @media (max-width: 768px)
        width: 40%
      &.login-btn
        //background-color: #c8a375
        border: solid 1px #fff
      &.register-btn
        background-color: #eee
        color: #666
  &-input
    position: relative

    input
      width: 100%
      font-size: 17px
      font-weight: 500
      color: #000
      padding: 15px 10px 15px 13px
      display: block
      border: 1px solid #f58194
      border-radius: 10px
      outline: none
      margin: 0 auto 5px
      -webkit-appearance: none

    span
      position: absolute
      right: 15px
      top: 50%
      transform: translateY(-50%)
      color: #666666
      font-size: 20px
      transition: all 0.3s

    @media (min-width: 768px)
      input
        font-size: 17px
        font-weight: 500
        color: #000
        padding: 15px 10px 15px 13px
        display: block
        border: 1px solid #f58194
        border-radius: 10px
        outline: none
        margin: 0 auto 5px
        -webkit-appearance: none

  &-agree
    display: flex
    margin-bottom: 40px
    margin-top: 30px

    .inputCheck
      display: inline
      input
        margin-right: 10px

    .text
      font-size: 14px
      color: #666666

    @media (min-width: 768px)
      width: 85%
      margin: 45px auto

      .text
        font-size: 16px
        text-align: center
</style>
<style scoped lang="sass">
.tips
  width: 100%
  display: flex
  justify-content: flex-end
  font-size: 12px
  color: red
</style>
<style scoped lang="sass">
.valid-feedback
  width: 100%
  color: green
  text-align: right
  font-size: 12px
  @media screen and (max-width: 768px)
    font-size: 10px
.invalid-feedback
  width: 100%
  color: #cf0000
  text-align: right
  font-size: 12px
  @media screen and (max-width: 768px)
    font-size: 10px

</style>
