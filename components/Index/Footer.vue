<script lang="ts" setup>
const { t } = useI18n()
const router = useRouter()
const siteStore = useSiteStore()
</script>

<template>
  <div>
    <footer class="footer-content" v-if="false">
      <div class="footer-content__box">
        <div class="footer-content__box-up">
          <ul class="footer-content__box-up__menu">
            <li><a href="https://www.aidma-hd.jp/privacy/" target="_blank"><span>
                  <font dir="auto" style="vertical-align: inherit;">
                    <font dir="auto" style="vertical-align: inherit;">{{ $lang('隱私權政策') }}</font>
                  </font>
                </span></a></li>
            <li><a href="/terms"><span>
                  <font dir="auto" style="vertical-align: inherit;">
                    <font dir="auto" style="vertical-align: inherit;">{{ $lang('服務條款') }}</font>
                  </font>
                </span></a></li>
            <li><a href="/contact"><span>
                  <font dir="auto" style="vertical-align: inherit;">
                    <font dir="auto" style="vertical-align: inherit;">{{ $lang('詢問') }}</font>
                  </font>
                </span></a></li>
            <li><a href="https://www.aidma-hd.jp/profile/"><span>
                  <font dir="auto" style="vertical-align: inherit;">
                    <font dir="auto" style="vertical-align: inherit;">{{ $lang('營運公司') }}</font>
                  </font>
                </span></a></li>
          </ul>
        </div>
        <div class="footer-content__box-down">
          <p>
            <font dir="auto" style="vertical-align: inherit;">
              <font dir="auto" style="vertical-align: inherit;">{{ $lang('愛多瑪控股有限公司') }}</font>
            </font>
          </p>
          <p>
            <font dir="auto" style="vertical-align: inherit;">
              <font dir="auto" style="vertical-align: inherit;">〒141-0021</font>
            </font>
          </p>
          <p>
            <font dir="auto" style="vertical-align: inherit;">
              <font dir="auto" style="vertical-align: inherit;">東京都品川區上大崎2-13-30 Oak目黑5樓、10樓</font>
            </font>
          </p>
        </div>
      </div>
      <div class="footer-content__copyright -worker">
        <p>
          <font dir="auto" style="vertical-align: inherit;">
            <font dir="auto" style="vertical-align: inherit;">
              {{ $lang('版權所有') }}
              Aidma Holdings Inc.
              {{ $lang('保留所有權利') }}.
            </font>
          </font>
        </p>
      </div>
    </footer>
  </div>
</template>
<style scoped lang="sass">
.footer-content
  .footer-content__box
    width: 100%
    display: grid
    gap: 20px
    justify-content: center
    align-items: center
    text-align: center
    grid-template-rows: repeat(2, 1fr)
    padding: 25px 0
    font-size: 16px
    color: #000
    background-color: hsla(0, 0%, 40%, .15)
    .footer-content__box-up
      .footer-content__box-up__menu
        display: flex
        gap: 20px
        justify-content: center
        align-items: center
        li
          a
            text-decoration: none
            &:hover
              text-decoration: underline
    .footer-content__box-down
      display: flex
      gap: 10px
      @media (max-width: 992px)
        display: block
  .footer-content__copyright
    background: repeating-linear-gradient(-45deg, #ff6c7a, #ff6c7a 5px, #ff8691 0, #ff8691 8px)
    height: 56px
    width: 100%
    display: flex
    align-items: center
    justify-content: center
    text-align: center
    color: #fff
    p
      color: #fff
      text-align: center
</style>
