<template>
  <main class="w-full pt-6 flex flex-col">
    <header class="flex justify-between gap-[10%] h-max shrink-0">
      <div class="w-full flex flex-col gap-[10px]">
        <section
          class="relative w-full h-[118px] border rounded-md flex items-center justify-center"
        >
          <Label class="opacity-[0.8]">+ 支付宝收款码</Label>
          <Input
            class="absolute top-0 left-0 w-full h-full opacity-0 cursor-pointer"
            type="file"
            accept="image/*"
            @change="alipayChange"
          />
        </section>
        <Input v-model="alipayVal" readonly type="text" placeholder="支付宝收款码待上传..." />
      </div>
      <div class="w-full flex flex-col gap-[10px]">
        <section
          class="relative w-full h-[118px] border rounded-md flex items-center justify-center"
        >
          <Label class="opacity-[0.8]">+ 微信收款码</Label>
          <Input
            class="absolute top-0 left-0 w-full h-full opacity-0 cursor-pointer"
            type="file"
            accept="image/*"
            @change="wechatChange"
          />
        </section>
        <Input v-model="wechatVal" readonly type="text" placeholder="微信收款码待上传..." />
      </div>
    </header>
    <div class="flex-1 flex pt-6" ref="qrcodeDom" @click="saveQrcode">
      <div class="relative w-full h-auto bg-[url('bg.png')] bg-no-repeat bg-contain bg-center">
        <QrCode
          class="w-auto"
          value="wxp://f2f0pdhzZ5T5Yt-nibaEs8QQVByuwn1Kkp7KKL8scTsDxwo"
          level="H"
          render-as="svg"
        />
        <QrCode
          value="https://qr.alipay.com/fkx13481winzch0wgpn2ada?t=1745975720083"
          render-as="svg"
        />
      </div>
    </div>
  </main>
</template>
<script lang="ts" setup>
import { ref } from 'vue'
import html2canvas from 'html2canvas'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import jsQR from 'jsqr'
import vh from 'vh-plugin'

//  上传支付宝收款码
const alipayVal = ref<string>()
const alipayChange = async (v: Event) => {
  alipayVal.value = ''
  const res = await loadImg(v.target)
  if (!res || !String(res).includes('alipay.com')) return vh.Toast('请上传正确的 支付宝 收款码')
  alipayVal.value = String(res)
}

// 上传微信
const wechatVal = ref<string>()
const wechatChange = async (v: Event) => {
  wechatVal.value = ''
  const res = await loadImg(v.target)
  if (!res || !String(res).includes('wxp://')) return vh.Toast('请上传正确的 微信 收款码')
  wechatVal.value = String(res)
}

// 加载图片
const loadImg = (v: any) => {
  const file = v.files[0]
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

import QrCode from 'qrcode.vue'

const qrcodeDom = ref<HTMLDivElement>()
const saveQrcode = async () => {
  if (!qrcodeDom.value) return
  // 将div转换为canvas
  const canvas = await html2canvas(qrcodeDom.value)
  // 将canvas转换为dataURL
  const dataURL = canvas.toDataURL('image/png')
  // 创建一个a标签用于下载
  const link = document.createElement('a')
  link.href = dataURL
  link.download = 'div_content.png'
  // 触发点击事件进行下载
  link.click()
}
</script>
