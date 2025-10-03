<script lang="ts" setup>
const { t } = useI18n()
const router = useRouter()
const playerStore = usePlayerStore()
const siteStore = useSiteStore()

const { updatePassword, updateTrasactionPassword } = playerStore
const pwd = ref({
  oPwd: '',
  nPwd: '',
  nPwdConfirm: '',
  showoPwd: false,
  shownPwd: false,
  shownPwdConfirm: false
})
const transactionPwd = ref({
  oPwd: '',
  nPwd: '',
  nPwdConfirm: '',
  showoPwd: false,
  shownPwd: false,
  shownPwdConfirm: false
})
const changePwd = ref(false)
const changenTPwd = ref(false)
// updatePassword()  更新密碼
// updateTrasactionPassword()  更新交易密碼
watch(
  () => pwd.value.nPwdConfirm,
  (val) => {
    if (val) {
      changePwd.value = true
    } else {
      changePwd.value = false
    }
  }
)
watch(
  () => transactionPwd.value.nPwdConfirm,
  (val) => {
    if (val) {
      changenTPwd.value = true
    } else {
      changenTPwd.value = false
    }
  }
)
const sendPasswordChange = async () => {
  console.log('changePwd.value', changePwd.value)
  console.log('changenTPwd.value', changenTPwd.value)
  if (changePwd.value) {
    if (pwd.value.nPwd !== pwd.value.nPwdConfirm) {
      ElNotification({
        title: `${t('密碼不一致')}`,
        type: 'error',
        duration: 1000,
        showClose: false
      })
      return
    }
    const res = await updatePassword({
      password: pwd.value.oPwd,
      newPassword: pwd.value.nPwd
    })
    if (res.success) {
      pwd.value.oPwd = ''
      pwd.value.nPwd = ''
      pwd.value.nPwdConfirm = ''
    }
  }
  if (changenTPwd.value) {
    if (transactionPwd.value.nPwd !== transactionPwd.value.nPwdConfirm) {
      ElNotification({
        title: `${t('交易密碼不一致')}`,
        type: 'error',
        duration: 1000,
        showClose: false
      })
      return
    }
    const res = await updateTrasactionPassword({
      password: transactionPwd.value.oPwd,
      newPassword: transactionPwd.value.nPwd
    })
    if (res.success) {
      transactionPwd.value.oPwd = ''
      transactionPwd.value.nPwd = ''
      transactionPwd.value.nPwdConfirm = ''
    }
  }
}
</script>

<template>
  <div class="pages">
    <div class="tag">
      <i class="fa-solid fa-user" style="color: #0068a7;"></i> {{ $lang('職員資料') }}
    </div>
    <div class="inf-bg">
      <div class="ul-contact-form-container">
        <h3 class="ul-contact-form-container__title"
          style="text-shadow: 0px 0px 6px rgba(253, 190, 117, 0.7);text-decoration: underline;">{{ $lang('職員基本資料') }}
        </h3>
        <div class="ul-contact-form">
          <div class="grid">

            <!-- 職員編號 (只讀) -->
            <div class="form-group">
              <div class="position-relative">
                <label style="color: rgb(12, 12, 158);font-weight: bold;font-size: 18px;">{{ $lang('職員編號') }} ▼</label>
                <input type="text" name="acc" :value="playerStore?.playerInfo?.account || ''" readonly
                  class="form-control">

              </div>
            </div>

            <!-- 職員名稱 (只讀) -->
            <div class="form-group">
              <div class="position-relative">
                <label style="color: rgb(12, 12, 158);font-weight: bold;font-size: 18px;">{{ $lang('職員名稱') }} ▼</label>
                <input type="text" name="name" :value="playerStore?.playerInfo?.username" readonly class="form-control">

              </div>
            </div>

            <!-- 職員電話 (只讀) -->
            <div class="form-group">
              <div class="position-relative">
                <label style="color: rgb(12, 12, 158);font-weight: bold;font-size: 18px;">{{ $lang('職員電話') }} ▼</label>
                <input type="text" name="tel"
                  :value="playerStore?.playerInfo?.countryCode + ' ' + playerStore?.playerInfo?.mobile" readonly
                  class="form-control">
              </div>
            </div><br>
          </div><br>
        </div>

        <!-- 更新密碼表單 -->
        <h3 class="ul-contact-form-container__title"
          style="text-shadow: 0px 0px 6px rgba(253, 190, 117, 0.7);text-decoration: underline;">{{ $lang('更換密碼') }}</h3>
        <div class="ul-contact-form">
          <div class="form-group" style="padding-top: 10px;">
            <div class="position-relative">
              <label style="color: rgb(102, 3, 58);font-weight: bold;font-size: 18px;">{{ $lang('舊密碼') }} ▼</label>
              <input v-model="pwd.oPwd" v-trim-input :type="pwd.showoPwd ? 'text' : 'password'" class="form-control"
                @copy.prevent @paste.prevent @contextmenu.prevent />
              <div v-if="!pwd.showoPwd" class="eyes" @click="pwd.showoPwd = true">
                <i class="fa-regular fa-eye-slash"></i>
              </div>
              <div v-else class="eyes" @click="pwd.showoPwd = false">
                <i class="fa-regular fa-eye"></i>
              </div>
            </div>
          </div>
          <div class="form-group" style="padding-top: 10px;">
            <div class="position-relative">
              <label style="color: rgb(102, 3, 58);font-weight: bold;font-size: 18px;">{{ $lang('新密碼') }} ▼</label>
              <input v-model="pwd.nPwd" v-trim-input :type="pwd.shownPwd ? 'text' : 'password'" class="form-control"
                @copy.prevent @paste.prevent @contextmenu.prevent />
              <div v-if="!pwd.shownPwd" class="eyes" @click="pwd.shownPwd = true">
                <i class="fa-regular fa-eye-slash"></i>
              </div>
              <div v-else class="eyes" @click="pwd.shownPwd = false">
                <i class="fa-regular fa-eye"></i>
              </div>
            </div>
          </div>
          <div class="form-group">
            <div class="position-relative">
              <label style="color: rgb(102, 3, 58);font-weight: bold;font-size: 18px;">{{ $lang('確認新密碼') }} ▼</label>
              <input v-model="pwd.nPwdConfirm" v-trim-input :type="pwd.shownPwdConfirm ? 'text' : 'password'"
                class="form-control" @copy.prevent @paste.prevent @contextmenu.prevent />
              <div v-if="!pwd.shownPwdConfirm" class="eyes" @click="pwd.shownPwdConfirm = true">
                <i class="fa-regular fa-eye-slash"></i>
              </div>
              <div v-else class="eyes" @click="pwd.shownPwdConfirm = false">
                <i class="fa-regular fa-eye"></i>
              </div>
            </div>
          </div><br>
          <!-- submit btn -->
          <div class="btn btn-primary update_password" @click="sendPasswordChange"><span
              style="font-size: 20px;font-weight: bold;">{{ $lang('更新密碼') }}</span></div>
        </div><br><br>

        <!-- 更新金鑰密碼表單 -->
        <h3 class="ul-contact-form-container__title"
          style="text-shadow: 0px 0px 6px rgba(253, 190, 117, 0.7);text-decoration: underline;">{{ $lang('更換金鑰密碼') }}
        </h3><br>
        <div action="" class="ul-contact-form">
          <div class="form-group" style="padding-top: 10px;">
            <div class="position-relative">
              <label style="color: rgb(182, 87, 10);font-weight: bold;font-size: 18px;">{{ $lang('舊金鑰密碼') }} ▼</label>
              <input v-model="transactionPwd.oPwd" v-trim-input :type="transactionPwd.showoPwd ? 'text' : 'password'"
                class="form-control" @copy.prevent @paste.prevent @contextmenu.prevent />
              <div v-if="!transactionPwd.showoPwd" class="eyes" @click="transactionPwd.showoPwd = true">
                <i class="fa-regular fa-eye-slash"></i>
              </div>
              <div v-else class="eyes" @click="transactionPwd.showoPwd = false">
                <i class="fa-regular fa-eye"></i>
              </div>
            </div>
          </div>
          <div class="form-group" style="padding-top: 10px;">
            <div class="position-relative">
              <label style="color: rgb(182, 87, 10);font-weight: bold;font-size: 18px;">{{ $lang('新金鑰密碼') }} ▼</label>
              <input v-model="transactionPwd.nPwd" v-trim-input :type="transactionPwd.shownPwd ? 'text' : 'password'"
                class="form-control" @copy.prevent @paste.prevent @contextmenu.prevent />
              <div v-if="!transactionPwd.shownPwd" class="eyes" @click="transactionPwd.shownPwd = true">
                <i class="fa-regular fa-eye-slash"></i>
              </div>
              <div v-else class="eyes" @click="transactionPwd.shownPwd = false">
                <i class="fa-regular fa-eye"></i>
              </div>
            </div>
          </div>
          <div class="form-group">
            <div class="position-relative">
              <label style="color: rgb(182, 87, 10);font-weight: bold;font-size: 18px;">{{ $lang('確認新金鑰密碼') }} ▼</label>
              <input v-model="transactionPwd.nPwdConfirm" v-trim-input
                :type="transactionPwd.shownPwdConfirm ? 'text' : 'password'" class="form-control" @copy.prevent
                @paste.prevent @contextmenu.prevent />
              <div v-if="!transactionPwd.shownPwdConfirm" class="eyes" @click="transactionPwd.shownPwdConfirm = true">
                <i class="fa-regular fa-eye-slash"></i>
              </div>
              <div v-else class="eyes" @click="transactionPwd.shownPwdConfirm = false">
                <i class="fa-regular fa-eye"></i>
              </div>
            </div>
          </div><br>
          <!-- submit btn -->
          <div class="btn btn-primary update_password" @click="sendPasswordChange"><span
              style="font-size: 20px;font-weight: bold;">{{
                $lang('更新金鑰密碼') }}</span></div>
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
.inf-bg
  margin-top: 50px
  background-image: url(@/assets/image/index/info-b01.png)
  background-size: cover
  padding: 0 clamp(15px, 3.15vw, 60px) clamp(40px, 4.2vw, 80px)
  .ul-contact-form-container 
    background-color: white
    margin: 0 clamp(15px, 19.71vw, 375px)
    padding: clamp(60px, 6.31vw, 70px) clamp(20px, 2.1vw, 40px)
    box-shadow: 0px -10px 40px rgba(0, 0, 0, 0.15)
    @media screen and (max-width: 991px) 
      margin: 0 clamp(15px, 11.71vw, 375px)
    @media screen and (max-width: 767px) 
      margin: clamp(15px, 3.15vw, 375px)
  .ul-contact-form input, .ul-contact-form textarea 
    display: block
    height: clamp(45px, 2.94vw, 56px)
    border: #F8E6E2 1px solid
    background-color: #F8E6E2
    padding: 0 clamp(15px, 1.58vw, 10px)
    padding-right: clamp(30px, 3.15vw, 60px)
    max-width: 100%
    width: 100%
    border-radius: 10px
  .form-control:focus 
    color: #212529
    background-color: #fff
    border-color: #86b7fe
    outline: 0
    box-shadow: 0 0 0 .25rem rgba(13, 110, 253, .25)
  .form-group
    margin-bottom: clamp(15px, 1.58vw, 30px)
    .position-relative
      position: relative
      .eyes
        position: absolute
        right: 10px
        bottom: 10px
        cursor: pointer
  .ul-contact-form-container__title
    text-align: center
    font-weight: 600
    font-size: clamp(22px, 2.52vw, 35px)
  .ul-contact-form .update_password 
    flex-shrink: 0
    background-color: #EF2853
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
</style>
