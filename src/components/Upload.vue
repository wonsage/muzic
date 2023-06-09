<template>
  <div>
    <div class="bg-white rounded border border-gray-200 relative flex flex-col">
      <div class="px-6 pt-6 pb-5 font-bold border-b border-gray-200">
        <span class="card-title">Upload</span>
        <i class="fas fa-upload float-right text-green-400 text-2xl"></i>
      </div>
      <div class="p-6">
        <!-- Upload Dropbox -->
        <div
          class="w-full px-10 py-20 rounded text-center cursor-pointer border border-dashed border-gray-400 text-gray-400 transition duration-500 hover:text-white hover:bg-green-400 hover:border-green-400 hover:border-solid"
          :class="{ 'bg-green-400 border-green-400 border-solid ': is_dargover }" @drag.prevent.stop=""
          @dragstart.prevent.stop="" @dragend.prevent.stop="" @dragenter.prevent.stop="is_dargover = false"
          @dragleave.prevent.stop="is_dargover = true" @dragover.prevent.stop="is_dargover = true"
          @drop.prevent.stop="upload($event)">
          <h5>Drop your files here</h5>
        </div>
        <input type="file" name="" id="" multiple @change="upload($event)" />
        <hr class="my-6" />
        <!-- Progess Bars -->
        <div class="mb-4" v-for="upload in uploads" :key="upload.stored_name">
          <!-- File Name -->
          <div class="font-bold text-sm" :class="upload.text_class">
            <i :class="upload.icon"></i>{{ upload.modified_name }}
          </div>
          <div class="flex h-4 overflow-hidden bg-gray-200 rounded">
            <!-- Inner Progress Bar -->
            <div class="transition-all progress-bar bg-blue-400" :class="upload.variant"
              :style="{ width: upload.current_progress + '%' }"></div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { storage, auth, songsCollection } from '@/includes/firebase'

export default {
  name: 'Upload',
  props: {
    addSong: {
      required: true,
      type: Function,
    },
  },
  data() {
    return {
      is_dargover: false,
      uploads: [],
    }
  },
  methods: {
    upload(event) {
      this.is_dargover = false
      console.log(event)
      // 兼容使用input选择文件上传的情况
      const files = event.dataTransfer ? [...event.dataTransfer.files] : [...event.target.files]
      console.log(files)
      files.forEach((file) => {
        if (file.type !== 'audio/mpeg') {
          return;
        }
        // fb
        const storedName = `${auth.currentUser.uid} - ${file.name}`
        const storageRef = storage.ref() // music-ad8bb.appspot.com
        const songsRef = storageRef.child(`songs/${storedName}`) // music-ad8bb.appspot.com/songs
        const task = songsRef.put(file)
        const uploadIndex = this.uploads.push({
          task,
          current_progress: 0,
          stored_name: storedName,
          modified_name: file.name,
          variant: 'bg-blue-400',
          icon: 'fas fa-spinner fa-spin',
          text_class: ''
        }) - 1;
        task.on('state_changed', (snapshot) => {
          const progress = (snapshot.bytesTransferred / snapshot.totalBytes) * 100;
          this.uploads[uploadIndex].current_progress = progress;
        }, (error) => {
          // 错误时执行
          this.uploads[uploadIndex].variant = 'bg-red-400'
          this.uploads[uploadIndex].icon = 'fas fa-times'
          this.uploads[uploadIndex].text_class = 'text-red-400'
          console.log(error)
        }, async () => {
          // 上传快照结束时执行
          // 将文件上传者的相关信息存入数据库
          const song = {
            uid: auth.currentUser.uid, // 上传用户的UID
            display_name: auth.currentUser.displayName, // 上传用户的外显名
            stored_name: task.snapshot.ref.name, // 存储中的文件名
            modified_name: file.name, // 上传的文件名作为默认外显文件名
            genre: '',
            comment_count: 0, // 评论数
          }
          song.url = await task.snapshot.ref.getDownloadURL()
          const songRef = await songsCollection.add(song) // DocumentReference
          const songSnapshot = await songRef.get() // DocumentSnapshot
          this.addSong(songSnapshot)
          // 样式变化提示上传成功
          this.uploads[uploadIndex].variant = 'bg-green-400'
          this.uploads[uploadIndex].icon = 'fas fa-check'
          this.uploads[uploadIndex].text_class = 'text-green-400'
        })
      })
    },
    cancelUpload() {
      this.uploads.forEach((upload) => {
        upload.task.cancel();
      })
    },
  },
  beforeUnmount() {
    this.cancelUpload()
  }
}
</script>

<style lang="scss" scoped></style>