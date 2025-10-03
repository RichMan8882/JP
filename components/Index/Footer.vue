<script lang="ts" setup>
const { t } = useI18n()
const router = useRouter()
const siteStore = useSiteStore()
</script>

<template>
  <footer class="footer">
    <!--<div class="footer-info">
      <p class="tel">
        {{ $lang('footerPhone') }}: &nbsp;<a href="tel:+88622647948"
          >02-2647948</a
        >
      </p>
      <p class="add">
        {{ $lang('add1') }}&nbsp; &nbsp;
        {{ $lang('add2') }}
      </p>
      <p class="mail">
        {{ $lang('電子郵件') }}:&nbsp;<a
          href="mailto:acuranano@acura-nanomed.com?subject=%E6%84%8F%E8%A6%8B%E5%9B%9E%E9%A5%8B"
          >acuranano@acura-nanomed.com</a
        >
      </p>
    </div>-->

    <div class="footer-copyright">
      <ul class="only-pc">
        <li>
          <button type="button" @click="navigateTo('/about/blurb')">
            {{ t('關於我們') }}
          </button>
        </li>
        <li>
          <button type="button" @click="navigateTo('/product/article')">
            {{ t('產品介紹') }}
          </button>
        </li>
        <li>
          <button type="button" @click="navigateTo('/cdmo')">
            {{ t('CDMO') }}
          </button>
        </li>
        <li>
          <button type="button" @click="navigateTo('/capability')">
            {{ t('核心能力') }}
          </button>
        </li>
        <!--<li>
          <button type="button" @click="navigateTo('/news_1')">
            {{ t('最新消息') }}
          </button>
        </li>-->
        <li>
          <!-- <button type="button" @click="navigateTo('/contact_1')">
            {{ t('聯絡我們') }}
          </button> -->
          <div>
            <a :href="siteStore.chatbox" target="_blank">{{ t('聯絡我們') }}</a>
          </div>
        </li>
        <!--<li>
          <button type="button" @click="navigateTo('/disclaimer')">
            {{ t('免責聲明') }}
          </button>
        </li>-->
      </ul>

      <p>Copyright © 2022 Adimmune All rights reserved.</p>
    </div>
  </footer>
</template>

<!-- <style scoped lang="sass">
.footer
  &-info
    padding: 35px
    font-size: 14px
    background-color: #F7F7F7
    text-align: center
    line-height: 1.7

    @media (min-width: 768px)
      p
        display: inline-block
        margin-right: 12px

      p + p
        position: relative
        padding-left: 12px

        &::before
          content: ''
          position: absolute
          left: 0
          top: 50%
          transform: translateY(-50%)
          width: 1px
          height: 14px
          background-color: #0559a5


  &-copyright
    background: linear-gradient(to right, #0559a5 0%, #1186b5 72%, #0e8da9 100%)
    color: #fff
    font-size: 12px
    text-align: center

    p
      padding: 12px 15px


    @media (min-width: 768px)
      padding: 30px 0 15px

      ul
        color: #C1E3FF
        justify-content: center
        align-items: center
        font-size: 14px
        max-width: 780px
        margin: 0 auto 20px
        li
          width: calc(100% / 9)
          button
            &:hover
              color: #fff
              text-decoration: underline
</style> -->
<style scoped lang="sass">
.footer
  background-color: rgb(37, 87, 132)
  color: white

  .footer-copyright
    margin: 0 auto
    width: 1080px
    @media (max-width: 1000px)
      width: 100%
      padding: 0 2.5rem

    .only-pc
      display: flex
      border-bottom: 1px solid white
      font-size: 1.1rem
      padding: 2.5rem 0
      @media (max-width: 1000px)
        display: block
        margin: 0.65rem
      li
        margin-right: 50px
        @media (max-width: 1000px)
          margin: 1.2rem 0

    p
      padding: 2.5rem 0
      font-size: 0.75rem
</style>
