<template>
  <header class="header">
    <section class="main">
      <h1>
        <img src="./assets/images/logo.webp" />
        PayQrcode
      </h1>
      <h2>二合一收款码物理合并版</h2>
    </section>
  </header>
  <main>
    <t-space direction="vertical">
      <t-alert
        theme="success"
        message="物理合并，直接将 图片合并成一张图片，无需服务器，不怕被篡改，可打印使用，稳定，绿色，安全！"
      />
      <t-alert theme="info">
        <template #message>
          开源地址：
          <t-link
            theme="primary"
            underline
            href="https://github.com/uxiaohan/PayQrcode"
            target="_blank"
          >
            PayQrcode
          </t-link>
          ，技术支持：
          <t-link theme="primary" underline href="https://www.vvhan.com" target="_blank">
            韩小韩博客
          </t-link>
        </template>
      </t-alert>
    </t-space>
    <Main />
  </main>
</template>
<script lang="ts" setup>
// import Header from '@/components/Header/Header.vue'
import Main from '@/components/Main/Main.vue'
// import Footer from '@/components/Footer/Footer.vue'
</script>
