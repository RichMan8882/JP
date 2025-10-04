<script lang="ts" setup>
const { t } = useI18n()
const router = useRouter()
const { signin } = useAuthStore()
const passwordEyes = ref(false)
const recaptchaCode = ref('')
const recaptchaCheckFunction = ref(null)
const focusItem = ref('')
const siteStore = useSiteStore()

const fetchSigninApi = ref({
  account: '',
  password: ''
})
const showPassword = (type: boolean) => {
  passwordEyes.value = type
}

const setLogin = async () => {
  // return navigateTo('/user')
  if (!fetchSigninApi.value.account) {
    ElMessage({
      message: '請輸入帳號',
      type: 'error',
      showClose: false
    })
    return
  }
  if (!fetchSigninApi.value.password) {
    ElMessage({
      message: '請輸入密碼',
      type: 'error',
      showClose: false
    })
    return
  }
  const recaptchaCheck = await recaptchaCheckFunction.value.validate(
    recaptchaCode.value
  )
  if (!recaptchaCheck) {
    ElMessage({
      message: '驗證碼錯誤',
      type: 'error',
      showClose: false
    })
  } else {
    const res = await signin(fetchSigninApi.value)
    if (res.success) {
      navigateTo('/user')
    }
  }
}
const checkHepler = async (data) => {
  recaptchaCheckFunction.value = data
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
</script>

<template>
  <div class="bg-temp">
    <client-only>
      <div class="login">
        <div class="login-main">
          <div class="login-top">
            <h1 style="font-weight: bold">職員專區</h1>
            <!--<img src="../static/picture/logo01.png" alt="" style="width: 200px;padding-bottom: 15px;">-->
            <h3 style="font-weight: bold">
              <span style="
                  background-color: #fff;
                  padding: 0 10px;
                  position: relative;
                  z-index: 3;
                ">登入系統</span>
            </h3>
            <div class="form">
              <input v-model="fetchSigninApi.account" v-trim-input type="text" placeholder="請輸入帳號"
                @focus="focusItem = 'account'" @focusout="focusItem = ''" />
              <input v-model="fetchSigninApi.password" v-trim-input :type="passwordEyes ? 'text' : 'password'"
                placeholder="請輸入密碼" @keyup.enter="setLogin" @focus="focusItem = 'password'"
                @focusout="focusItem = ''" />
              <br />

              <!-- 驗證碼圖片 -->
              <recaptcha style="
                  cursor: pointer;
                  width: 152px;
                  border: 1px solid #ccc;
                  margin: 0 auto;
                " @check-hepler="checkHepler"></recaptcha>
              <input v-model="recaptchaCode" v-trim-input type="text" placeholder="請輸入驗證碼" @keyup.enter="setLogin"
                @focus="focusItem = 'recaptcha'" @focusout="focusItem = ''" />
              <input type="submit" value="登入" @click="setLogin" />
              <input type="submit" class="regis" value="注冊" @click="() => { navigateTo('/register') }">
            </div>

            <h3 style="font-weight: bold;margin-top: 20px;">
              <span style="
                  background-color: #fff;
                  padding: 0 10px;
                  position: relative;
                  z-index: 3;
                ">快捷服務</span>
            </h3>
            <div class="login-bottom">
              <div class="login-check">
                <a @click="
                  () => {
                    navigateTo('/user')
                  }
                "><label class="checkbox">返回首頁</label></a>
              </div>
              <div class="login-para" @click="openChatBox()">
                <p><a> 聯絡客服 </a></p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </client-only>
  </div>
</template>

<style scoped lang="sass">
.bg-temp
  background: #f5f5f6
  padding: 70px 0px 70px 0px
  min-height: 100dvh
  font-size: 100%
  margin: 0
  display: flex
  align-items: center
  justify-content: center
  .login
    width: 40%
    background: #fff
    border-radius: 15px
    margin: 0 auto
    min-height: 400px
    padding: 10px
    @media screen and (max-width: 1440px)
      width: 60%

    @media screen and (max-width: 960px)
      width: 90%
  .login-main
    border: 2px solid #f58194
    min-height: 400px
    border-radius: 15px

  .login-top
    padding: 45px 0px
    text-align: center
    position: relative

  h1
    font-size: 26px
    font-weight: 500
    color: #f58194
    margin: 10px 0px 35px 0px

  h3
    font-size: 19px
    font-weight: 400
    color: #8e8e8e
    margin: 0px 0px 16px 0px
    position: relative

    &:before
      width: 135px
      height: 1px
      background: #8e8e8e
      position: absolute
      top: 50%
      left: 21%
      z-index: 1
      content: ''

    &:after
      width: 135px
      height: 1px
      background: #8e8e8e
      position: absolute
      top: 50%
      right: 20%
      z-index: 1
      content: ''

  .form
    margin: 0
    padding: 0 20px
    border: 0
    font-size: 100%
    font: inherit
    vertical-align: baseline
    @media (max-width: 992px)
      padding: 0 15px

  .login-top input
    &[type="text"]
      font-size: 17px
      font-weight: bold
      color: #000
      padding: 15px 10px 15px 13px
      width: 100%
      border-radius: 10px
      display: block
      border: 1px solid #f581949a
      outline: none
      margin: 30px auto 20px
      -webkit-appearance: none

    &[type="password"]
      font-size: 17px
      font-weight: bold
      color: #000
      padding: 15px 10px 15px 13px
      width: 100%
      border-radius: 10px
      display: block
      border: 1px solid #f581949a
      margin: 0 auto
      outline: none
      -webkit-appearance: none

    &[type="submit"]
      background: #f58194
      color: #FFF
      border: none
      font-size: 23px
      font-weight: 400
      padding: 10px 0px
      width: 100%
      border-radius: 10px
      cursor: pointer
      outline: none
      transition: all 0.3s ease-in-out
      &:hover
        background: #f58194d9
        color: #FFF
  .regis
    border: 2px solid #f58194 !important
    background: #fff !important
    color: #f58194 !important
    margin-top: 24px
    &:hover
      background: #f58194 !important
      color: #fff !important
  .login-bottom
    width: 60%
    margin: 25px auto
    display: flex
    align-items: center
    justify-content: space-between

  .login-top .checkbox
    font-size: 17px
    font-weight: bold
    color: #696969
    cursor: pointer
    position: relative
    display: block

  a
    text-decoration: none

  .login-top p a
    font-size: 17px
    font-weight: bold
    color: #8e8e8e

@media screen and (max-width: 1140px)

@media screen and (max-width: 992px)


@media screen and (max-width: 768px)
  .bg-temp .login-top input
    width: 100% !important
</style>
