<script setup lang="ts">
import { NButton, NDivider, NInput, NSelect, NSwitch, useMessage } from 'naive-ui'
import { onMounted, ref } from 'vue'
// import dayjs from 'dayjs'
import uni from '@dcloudio/uni-webview-js'
import { useAppStore, useUserStore } from '@/store'
import { localStorage } from '@/utils/storage/localStorage'
import Page from '@/components/page/index.vue'
import { useBack, useGo } from '@/utils/router'
// let props = defineProps(['register'])
const emits = defineEmits(['modifyPassword', 'register'])
const back = useBack()
const go = useGo()
const userStore = useUserStore()

const ms = useMessage()
const appStore = useAppStore()

// const nickname = computed(() => {
//   return userStore.userInfo.user.nickname
// })

onMounted(() => {
  updated()
})
function updated() {
  userStore.residueCountAPI()
}


defineExpose({ updated })

const apiKey = ref(localStorage.getItem('apiKey') || '') as any
function settingBtn() {
  if (apiKey.value === '' || apiKey.value.startsWith('sk-')) {
    localStorage.setItem('apiKey', apiKey.value)
    ms.info('key设置成功~请保证填写正确的key，并且key有额度没有过期~', { duration: 5000 })
    userStore.residueCountAPI()
  }
  else {
    ms.error('正确的key是以sk-开头的', { duration: 5000 })
  }
}

const fontSizeNum = ref(localStorage.getItem('fontSizeNum') || '100%') as any
function fontSizeNumBtn() {
  localStorage.setItem('fontSizeNum', fontSizeNum.value.endsWith('%') ? fontSizeNum.value : `${fontSizeNum.value < 50 ? 50 : fontSizeNum.value}%`)
  const htmlDom = document.querySelector('html') as any
  htmlDom.style.zoom = localStorage.getItem('fontSizeNum')
}

// 主题
function handleUpdateValue(chatmossTheme: string) {
  ms.info(chatmossTheme === 'dark' ? '深色模式开启' : '浅色模式开启')
  localStorage.setItem('chatmossTheme', chatmossTheme)
  appStore.setTheme(localStorage.getItem('chatmossTheme') as any)
  uni.postMessage({
    data: {
      theme: chatmossTheme,
    },
  })
}
function getNSwitchValue(): any {
  return localStorage.getItem('chatmossTheme')
}


// 模型选择
// console.log(userStore.getOpenaiVersion)
if (userStore.userInfo.fourSwitch !== 'ON')
  userStore.saveOpenaiVersion('3.5')

const modelValue = ref(userStore.getOpenaiVersion)

const options = ref<any[]>(userStore.options)

// 专业模式
function handleModeValue(chatmossMode: string) {
  // 专业模式 speciality | 正常模式 normal
  ms.info(chatmossMode === 'speciality' ? '专业模式开启' : '正常模式开启')
  localStorage.setItem('chatmossMode', chatmossMode)
}
function getNSwitchModeValue(): any {
  return localStorage.getItem('chatmossMode')
}
</script>

<template>
  <Page>
    <template #title>
      <van-nav-bar title="设置中心" left-text="返回" left-arrow @click-left="back" />
    </template>
    <div class="setting-mian dark:text-white">
      <div class="flex items-center justify-between ">
        <div class="flex">
          <!-- <span class="mr-4">用户名称：{{ nickname || '未登录' }}</span> -->
          <!-- <span>{{ plusEndTime }}到期</span> -->
        </div>
        <div class="flex">
          <NButton
            v-if="userStore.userInfo.user.email" id="question-push" type="primary" size="tiny" quaternary
            @click="() => { go({ name: 'feedback' }) }"
          >
            问题反馈
          </NButton>
          <NButton type="primary" size="tiny" quaternary @click="() => { go({ name: 'login' }) }">
            修改密码
          </NButton>
        </div>
      </div>
      <div class="title-h1">
        ApiKeys设置
      </div>
      <div class="flex">
        <NInput v-model:value="apiKey" class="mr-2" type="text" placeholder="请输入您的apiKey" />
        <NButton type="primary" ghost @click="settingBtn">
          确定
        </NButton>
      </div>
      <div class="tip-text-input">
        小提示：设置成功，并不代表您的key有余额或者正确
      </div>
      <div class="tip-text-input">
        可以点击这个网址进行检查：
        <a style="color: #0099FF;" href="http://open.aihao123.cn/" target="_blank">http://open.aihao123.cn/</a>
      </div>
      <NDivider />
      <div>
        <div class="title-h1">
          OpenAI模型选择
        </div>
        <div class="flex">
          <NSelect
            v-model:value="modelValue" :options="options"
            @change="(value) => { userStore.saveOpenaiVersion(value) }"
          />
        </div>
        <div class="tip-text-input">
          小提示：在ChatMoss中，ChatGPT4.0消耗的字符数要比ChatGPT3.5多
          <span class="font-bold" style="color: #FF6666;">{{ userStore.userInfo.fourRate }}</span>
          倍，但是回答的更加专业
        </div>
      </div>
      <NDivider />
      <div>
        <div class="title-h1">
          ChatMoss主题设定
        </div>
        <div class="flex">
          <NSwitch
            :default-value="getNSwitchValue()" checked-value="dark" unchecked-value="light"
            @update:value="handleUpdateValue"
          />
          {{ getNSwitchValue() === 'dark' ? '深色模式' : '浅色模式' }}
        </div>
      </div>
      <NDivider />
      <div>
        <div class="title-h1">
          回答模式（专业模式下会自动在每个问题后面拼接 请详细回答 五个字，理论上回答内容更多）
        </div>
        <div class="flex">
          <NSwitch
            :default-value="getNSwitchModeValue()" checked-value="speciality" unchecked-value="normal"
            @update:value="handleModeValue"
          />
          {{ getNSwitchModeValue() === 'speciality' ? '专业模式' : '正常模式' }}
        </div>
      </div>
      <NDivider />
      <div class="title-h1">
        字体大小设置
      </div>
      <div class="flex">
        <NInput v-model:value="fontSizeNum" class="mr-2" type="text" placeholder="请输入字体设置比例" />
        <NButton type="primary" ghost @click="fontSizeNumBtn">
          确定
        </NButton>
      </div>
      <NDivider />
      <div>
        <span class="title-h2">本机累计使用字符数</span>：未知
      </div>
      <div class="tip-text-input">
        小提示：数据统计之前采用本地统计并不准确，目前我们在做服务器数据统计，数据更准，敬请期待
      </div>
    </div>
  </Page>
</template>

<style lang="less" scoped>
.item {
  height: 7rem;
  flex: 1;
  min-width: 6rem;
}

.desc {
  font-size: 12px;
  margin-top: 8px;
}

.tip-text-input {
  font-size: 12px;
  margin-top: 20px;
  margin-bottom: -10px;
}

.title-h1 {
  margin: 10px 0px;
  color: #FF6666;
}

.setting-mian {
	padding: 0 15px;
	padding-top: 20px;
	padding-bottom: 60px;
}
</style>
