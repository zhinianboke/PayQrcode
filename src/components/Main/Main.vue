<template>
  <header>
    <section class="upload-list">
      <div class="item">
        <t-input-adornment>
          <template #append>
            <t-upload
              accept="image/*"
              :showUploadProgress="false"
              :useMockProgress="false"
              :autoUpload="false"
              theme="custom"
              :onChange="alipayChange"
              :trigger-button-props="{ theme: 'primary', variant: 'base' }"
            />
          </template>
          <t-input
            :status="alipayVal ? 'success' : ''"
            readonly
            v-model="alipayVal"
            placeholder="请上传 支付宝 收款码"
          />
        </t-input-adornment>
      </div>
      <div class="item">
        <t-input-adornment>
          <template #append>
            <t-upload
              accept="image/*"
              :showUploadProgress="false"
              :useMockProgress="false"
              :autoUpload="false"
              theme="custom"
              :onChange="wechatChange"
              :trigger-button-props="{ theme: 'success', variant: 'base' }"
            />
          </template>
          <t-input
            :status="wechatVal ? 'success' : ''"
            readonly
            v-model="wechatVal"
            placeholder="请上传 微信 收款码"
          />
        </t-input-adornment>
      </div>
      <div class="item flex">
        <span class="title">美化</span>
        <t-switch v-model="themeStatus" />
      </div>
    </section>
    <t-card class="alipay-hide">
      <span class="title">缺省比例</span>
      <t-slider v-model="qrHideSize" :onChange="alipayHideChange" />
    </t-card>
  </header>
  <main :class="{ show: wechatQrcodeImg && alipayQrcodeImg }">
    <div class="qr-main" :class="{ nobg: !themeStatus }" ref="qrcodeDom">
      <img class="wechat-qr" :src="wechatQrcodeImg" />
      <img class="alipay-qr" :src="alipayQrcodeImg" />
    </div>
    <t-button theme="success" class="download-qrcode" :onClick="downloadQrcode">
      下载二维码
    </t-button>
  </main>
</template>
<script lang="ts" setup>
import { ref, onMounted } from 'vue'
import html2canvas from 'html2canvas'
import jsQR from 'jsqr'
import QrCode from 'qrcode'
import { NotifyPlugin } from 'tdesign-vue-next'

//  是否美化
const themeStatus = ref<boolean>(false)

//  上传支付宝收款码
const alipayFileList = ref<any>()
const alipayVal = ref<any>()
const alipayQrcodeImg = ref<string>()
const alipayChange = async (v: any) => {
  alipayFileList.value = v && v.length ? v[0].raw : alipayFileList.value
  if (!alipayFileList.value) return
  const res = await loadImg(alipayFileList.value)
  if (!res || !String(res).includes('alipay.com'))
    return NotifyPlugin.error({
      title: 'Error',
      content: '请上传正确的 支付宝 收款码',
    })
  alipayVal.value = String(res)
  alipayQrcodeImg.value = await drawQR(alipayVal.value, true)
  showQrcode()
}

// 上传微信
const wechatFileList = ref<any>()
const wechatVal = ref<any>()
const wechatQrcodeImg = ref<string>()
const wechatChange = async (v: any) => {
  wechatFileList.value = v && v.length ? v[0].raw : wechatFileList.value
  if (!wechatFileList.value) return
  const res = await loadImg(wechatFileList.value)
  if (!res || !String(res).includes('wxp://'))
    return NotifyPlugin.error({
      title: 'Error',
      content: '请上传正确的 微信 收款码',
    })
  wechatVal.value = String(res)
  wechatQrcodeImg.value = await drawQR(wechatVal.value)
  showQrcode()
}

// 支付宝隐藏比例
const qrHideSize = ref<number>(50)
const alipayHideChange = (v: any) => {
  qrHideSize.value = v
  alipayChange(null)
}

// 生成二维码
const showQrcode = async () => {
  if (!alipayQrcodeImg.value || !wechatQrcodeImg.value) return
}

// 加载图片
const loadImg = (file: any) => {
  if (!file) return
  const reader = new FileReader()
  reader.readAsDataURL(file)
  return new Promise((resolve) => {
    reader.onload = (e) => {
      const image = new Image()
      image.src = e.target!.result as any
      image.onload = () => {
        const code = qrcodeResult(image)
        resolve(code)
      }
    }
  })
}

//  识别二维码
const qrcodeResult = (image: any) => {
  const canvas = document.createElement('canvas')
  const context: any = canvas.getContext('2d')
  canvas.width = image.width
  canvas.height = image.height
  context.drawImage(image, 0, 0, canvas.width, canvas.height)
  const imageData = context.getImageData(0, 0, canvas.width, canvas.height)
  const decodedResult = jsQR(imageData.data, imageData.width, imageData.height, {
    inversionAttempts: 'dontInvert',
  })
  return decodedResult?.data
}

// 添加清除右上角的函数
const qrcodeDom = ref<any>()
const qrSize = ref<number>(0)
const drawQR = async (qrValue: string, hide: boolean = false) => {
  const qrDom = document.createElement('canvas')
  await QrCode.toCanvas(qrDom, qrValue, {
    errorCorrectionLevel: 'H',
    width: qrSize.value,
    margin: 0,
    color: {
      dark: '#000000',
      light: '#ffffff',
    },
  })
  if (hide) {
    const ctx: any = qrDom.getContext('2d')
    const clearWidth = (qrSize.value / 2) * (qrHideSize.value / 100)
    ctx.clearRect(qrSize.value / 2, qrSize.value - clearWidth, qrSize.value / 2, clearWidth)
  }
  return qrDom.toDataURL('image/png')
}

// 下载二维码
const downloadQrcode = async () => {
  const canvas = await html2canvas(qrcodeDom.value)
  const dataURL = canvas.toDataURL('image/png')
  const link = document.createElement('a')
  link.href = dataURL
  link.download = 'PayQrcode.png'
  link.click()
}

onMounted(() => {
  qrSize.value = qrcodeDom.value!.clientWidth
})
</script>
<style lang="less" scoped>
@import url('./Main.less');
</style>
