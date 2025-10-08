<script lang="ts" setup>
const playerStore = usePlayerStore()
const siteStore = useSiteStore()
const { bankApply } = usePlayerStore()
const pageType = ref('bank')
const bankInfo = ref({
  bankName: siteStore.siteData?.depositOptions.bank[0],
  branch: '',
  account: '',
  accountNo: '',
  image: ['', '', '']
})
await useAsyncData(async () => {
  // 如果綁定銀行卡跳轉錢包
  if (playerStore?.playerInfo?.bankInfo !== null) {
    if (playerStore?.playerInfo?.bankInfo.status === 1) {
      navigateTo('/user/withdraw')
    }
  }
})
// methods
const submitBankApply = async () => {
  console.log(bankInfo)
  if (bankInfo.value.branch === '') {
    return ElMessage.error('請填寫分行')
  }
  if (bankInfo.value.account === '') {
    return ElMessage.error('請填寫戶名')
  }
  if (bankInfo.value.accountNo === '') {
    return ElMessage.error('請填寫帳號')
  }
  // if (bankInfo.value.image[0] === '') {
  //   return ElMessage.error('請上傳身分證正面')
  // }
  // if (bankInfo.value.image[1] === '') {
  //   return ElMessage.error('請上傳身分證反面')
  // }
  if (bankInfo.value.image[2] === '') {
    return ElMessage.error('請上傳檔案')
  }
  const bankApplyRes = await bankApply(bankInfo.value)
  if (bankApplyRes) {
    setTimeout(async () => {
      const res = await playerStore.fetchInfo()
      if (res.success) {
        if (playerStore.playerInfo.bankInfo.status === 1) {
          navigateTo('/user/withdraw')
        }
      }
    }, 1000)
  }
}
const changeType = (type: any) => {
  pageType.value = type
  switch (type) {
    case 'bank':
      bankInfo.value = {
        bankName: siteStore.siteData?.depositOptions.bank[0],
        branch: '',
        account: '',
        accountNo: '',
        image: ['', '', '']
      }
      break
    case 'address':
      bankInfo.value = {
        bankName: siteStore.siteData?.depositOptions.crypto[0],
        branch: '',
        account: '',
        accountNo: '',
        image: []
      }
      break
  }
}
const updateImage = (data: any) => {
  // console.log(data)
  if (data.url[0]) {
    bankInfo.value.image[data.index] = data.url[0].url
  } else {
    bankInfo.value.image[data.index] = ''
    console.log(bankInfo.value.image)
  }
}
</script>

<template>
  <div class="pages">
    <headerTop />
    <div class="tag">
      <h2>
        <i class="fa-regular fa-file-lines"></i>
        {{ $lang('薪資申請') }}
      </h2>
    </div>
    <div v-if="Object.keys(playerStore.playerInfo.bankInfo || {}).length === 0" class="inf-bg">
      <div class="ul-contact-form-container">
        <h3 class="ul-contact-form-container__title" style="
            text-shadow: 0px 0px 6px rgba(253, 117, 212, 0.7);
            text-decoration: underline;
          ">
          {{ $lang('請新增一筆銀行資訊') }}
        </h3>
        <div class="ul-contact-form">
          <div class="">
            <!--  銀行  -->
            <div class="form-group">
              <div class="position-relative">
                <label style="
                    
                    font-weight: bold;
                    font-size: 18px;
                  ">{{ $lang('銀行') }}</label>
                <select v-model="bankInfo.bankName" name="phoneCode" class="form-control"
                  style="border: 2px solid #ff6c7a; border-radius: 10px;margin-left: 5px;padding: 0 10px;"
                  v-if="siteStore.siteData?.depositOptions.bank">
                  <option v-for="item in siteStore.siteData?.depositOptions.bank" :key="item">
                    {{ item }}
                  </option>
                </select>
              </div>
            </div>

            <!-- 分行 -->
            <div class="form-group">
              <div class="position-relative">
                <label style="
                    
                    font-weight: bold;
                    font-size: 18px;
                  ">{{ $lang('分行') }}</label>
                <input v-model="bankInfo.branch" v-trim-input type="text" name="name" class="form-control" />
              </div>
            </div>

            <!-- 帳戶號碼  -->
            <div class="form-group">
              <div class="position-relative">
                <label style="
                    
                    font-weight: bold;
                    font-size: 18px;
                  ">{{ $lang('帳戶號碼') }}</label>
                <input v-model="bankInfo.accountNo" v-trim-input type="text" name="tel" class="form-control" />
              </div>
            </div>
            <!-- 帳戶戶名 -->
            <div class="form-group">
              <div class="position-relative">
                <label style="
                    
                    font-weight: bold;
                    font-size: 18px;
                  ">{{ $lang('帳戶戶名') }}</label>
                <input v-model="bankInfo.account" v-trim-input type="text" name="name" class="form-control" />
              </div>
            </div>
            <!--  上傳照片 -->
            <div class="form-group">
              <div class="position-relative">
                <label style="
                    
                    font-weight: bold;
                    font-size: 18px;
                  ">{{ $lang('上傳照片') }}</label>
                <div class="form-control" style="display: flex; flex-wrap: wrap">
                  <pureImgUploader :limit="1" :index="2" @update-image="updateImage">
                  </pureImgUploader>
                </div>
                <!-- <input type="file" name="name" v-model="playerStore.playerInfo.username" class="form-control"> -->
                <span style="font-size: 12px; color: #8d1515">請上傳身分證(其他證件亦可)與存摺照片，證件+存摺務必同名，請勿修圖後製，請耐心等待審核並開通</span>
              </div>
            </div>
            <br />
            <button type="submit" name="update_password" class="buttonWhGreen" @click="submitBankApply">
              {{ $lang('送出') }}
            </button>
          </div>
        </div>
      </div>
    </div>
    <div v-else style="">
      <div class="input-classic" style="display: flex;justify-content: center;align-items: center;padding: 200px 0;">
        <span class="input-title" style="text-align: center;">{{ $lang('銀行資訊狀態') }}</span>
        <div class="input-text text-center" style="color: #ff6c7a; font-size: 22px;">
          {{ $lang('審核中') }}
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
    .fa-regular
      color: rgba(255, 255, 255, 0.5)
.inf-bg
  margin-top: 50px
  padding: 0 clamp(15px, 3.15vw, 60px) clamp(40px, 4.2vw, 80px)
  @media (max-width: 992px) 
    padding: 0
  .ul-contact-form-container 
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
    margin-block-end: clamp(15px, 1.58vw, 30px)
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
