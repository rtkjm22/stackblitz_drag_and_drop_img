<template>
  <div class="dragAndDrop">
    <div
      class="dragAndDrop_field"
      :class="{ over: isDragOver }"
      @drop.stop="upload"
      @dragover.prevent="onDrag('over')"
      @dragleave="onDrag('leave')"
    >
      <input type="file" @change="upload" />
      <p>画像をドラッグ＆ドロップ <br />またはクリックでファイル選択</p>
    </div>
    <button type="button" class="delete-button" @click="deleteImage">
      delete
    </button>
    <img :src="url" alt="" />

    <ul v-if="fileErrMsg.length > 0" class="error-message">
      <li v-for="(message, i) in fileErrMsg" :key="i">{{ message }}</li>
    </ul>
  </div>
</template>

<script setup lang="ts">
const IS_DRAGGED = {
  OVER: 'over',
  LEAVE: 'leave'
}
const IMAGE_TYPE = {
  JPEG: 'image/jpeg',
  PNG: 'image/png'
}

type IS_DRAGGED = typeof IS_DRAGGED[keyof typeof IS_DRAGGED]
type IMAGE_TYPE = typeof IMAGE_TYPE[keyof typeof IMAGE_TYPE]

const SIZE_LIMIT = 5000000

let isDragOver = ref(false)
let url = ref('https://placehold.jp/a6deda/ffffff/150x150.png')
let fileErrMsg = ref([''])

const onDrag = (type: IS_DRAGGED): void => {
  isDragOver.value = type === IS_DRAGGED.OVER
}

const deleteImage = (): void => {
  url.value = 'https://placehold.jp/a6deda/ffffff/150x150.png'
  fileErrMsg.value = []
}

const getBase64 = (item: Blob): Promise<string | ArrayBuffer> => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.readAsDataURL(item)
    reader.onload = () => {
      resolve(reader.result)
    }
    reader.onerror = (err) => reject(err)
  })
}

const checkFile = (file: File): boolean => {
  let result = true

  if (!file) {
    fileErrMsg.value.push('エラーメッセージ1')
    result = false
  }
  if (file.type !== IMAGE_TYPE.JPEG && file.type !== IMAGE_TYPE.PNG) {
    fileErrMsg.value.push('エラーメッセージ2')
    result = false
  }
  if (file.size > SIZE_LIMIT) {
    fileErrMsg.value.push('エラーメッセージ3')
    result = false
  }

  return result
}

const upload = async (e) => {
  isDragOver.value = false
  const files: FileList =
    (e.target as HTMLInputElement).files ||
    (e.dataTransfer as DataTransfer).files
  const file = files[0]

  if (checkFile(file)) {
    const picture = await getBase64(file)
    url.value = picture as string
  }
}
</script>

<style scoped lang="scss">
.dragAndDrop {
  background-color: #eee;
  width: 300px;
  height: 300px;
  &_field {
    width: 100%;
    height: 100%;
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    &.over {
      background-color: #666;
    }
    input {
      position: absolute;
      width: 100%;
      height: 100%;
      top: 0;
      left: 0;
      opacity: 0;
      cursor: pointer;
    }
    p {
      color: #aaa;
      text-align: center;
    }
  }
}
.error-message {
  color: red;
  list-style: none;
  margin: 0.4rem 0 0 0;
  padding: 0 0.2rem;
  font-size: 1.6rem;
}
</style>

