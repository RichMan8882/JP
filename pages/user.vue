<script lang="ts" setup></script>

<template>
  <div class="content">
    <client-only>
      <router-view></router-view>
    </client-only>
  </div>
</template>

<style scoped lang="sass">
.content
  background-color: #fff
  width: 100dvw
  position: relative
  height: 100dvh
  overflow-y: auto
</style>
