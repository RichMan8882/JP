<script setup>
const siteStore = useSiteStore()

const openChatBox = () => {
  window.open(siteStore.chatbox, '_blank')
}
</script>
<template>
  <div class="container">
    <div class="row">
      <div class="col-lg-9">
        <div class="single-post">
          <h1 style="font-size: 30px; font-weight: bold; color: #332a86">
            關於我們
          </h1>
          <p style="font-size: 20px; font-weight: bold; color: #016ba9">
            我們將為您提供專業服務支援。康泰倫特在全球範圍內為您的藥物開發、給藥和供應提供解決方案，因此需要您提供確切信息，幫助您匹配對應領域的專家。
          </p>
          <br />
          <h1 style="font-size: 30px; font-weight: bold; color: #332a86">
            我們是誰
          </h1>
          <p style="font-size: 20px; font-weight: bold; color: #016ba9">
            催化劑+才智
          </p>
          <p style="font-size: 20px; font-weight: bold; color: #016ba9">
            我們的名字結合兩者的概念。作為全球領先的製藥業合作夥伴，康泰倫特的業務涵蓋藥品研發、管理與供應，生物製品和消費者健康產品領域，我們是您成功的催化劑。
          </p>
          <br />
          <p style="font-size: 20px; font-weight: bold; color: #000000">
            康泰倫特擁有85年的行業服務經驗，在將更多的客戶產品更快地推向市場、提高產品性能和確保可靠的臨床和商業產品供應方面具備專業知識。
          </p>
          <p style="font-size: 20px; font-weight: bold; color: #000000">
            我們在上百個市場服務了數千家企業，其中包括41家世界排名TOP 50
            的生物技術公司和48家全球排名TOP 50 的製藥企業。
          </p>
          <p style="font-size: 20px; font-weight: bold; color: #000000">
            ●
            業界40%的新藥核准由我們1,000多位科學人才協助完成，全球30多家設施年產量達1,000億件單品。
          </p>
          <p style="font-size: 20px; font-weight: bold; color: #000000">
            ●
            我們在口服、注射、呼吸系統藥品劑型以及先進藥品包裝方面的重要智慧財產權成就了1,200以上專利和專利申請。
          </p>
          <br />

        </div>
        <hr />
      </div>
      <aside class="col-lg-3" style="padding-top: 80px">
        <div class="widget">
          <div class="widget-title" style="background-color: #016ba9">
            <h4 style="
                color: rgb(255, 255, 255);
                text-align: center;
                font-size: 18px;
              ">
              據點分布
            </h4>
          </div>
          <img src="@/assets/image/index/about-1.png" alt="" style="padding-bottom: 20px" />
          <img src="@/assets/image/index/about-2.png" alt="" style="padding-bottom: 20px" />
          <img src="@/assets/image/index/about-3.png" alt="" />
        </div>
        <div class="widget">
          <div class="widget-title" style="background-color: #016ba9">
            <a @click="openChatBox">
              <h4 style="
                  color: rgb(255, 255, 255);
                  text-align: center;
                  font-size: 18px;
                ">
                聯繫我們
              </h4>
            </a>
          </div>
        </div>
      </aside>
    </div>
    <!-- /row -->
  </div>
</template>
<style scoped lang="sass">

.container, .container-lg, .container-md, .container-sm, .container-xl, .container-xxl
  margin: 30px auto !important
  @media (min-width: 1400px)
    max-width: 1320px
.margin-30
  margin-block: 30px
.row
  --bs-gutter-x: 1.5rem
  --bs-gutter-y: 0
  display: flex
  flex-wrap: wrap
  margin-top: calc(var(--bs-gutter-y)* -1)
  margin-right: calc(var(--bs-gutter-x)* -.5)
  margin-left: calc(var(--bs-gutter-x)* -.5)
  @media screen and (max-width: 991px)
    display: block
    padding: 12px !important
  .col-lg-9
    flex: 9
    margin: 0 .75rem
  .col-lg-3
    flex: 3
    margin: 0 .75rem
.single-post
  background-color: #fff
  border-radius: 3px
  padding: 30px 30px 1px 30px
  margin-bottom: 25px
  box-shadow: 0px 0px 30px 0px rgba(0, 0, 0, 0.1)
  p
    font-size: 17px
    font-weight: bold
    color: #000000
    line-height: 1.8
    margin-bottom: 15px
.widget
  position: relative
  display: block
  margin-bottom: 15px
  .widget-title
    padding: 15px 0
    margin-bottom: 20px
    border-bottom: 1px solid #dddddd
@media screen and (max-width: 991px)
  // .col-lg-3
  //   display: none
  .col-lg-9
    margin: 0 auto !important
    width: 100%
    max-width: 100%
    // padding-right: .75rem
    // padding-left: .75rem
    margin-top: 0
</style>
