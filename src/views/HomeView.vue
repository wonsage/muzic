<template>
  <div>
    <!-- Introduction -->
    <section class="mb-8 py-20 text-white text-center relative">
      <div class="absolute inset-0 w-full h-full bg-contain introduction-bg"
        style="background-image: url(assets/img/header.png)"></div>
      <div class="container mx-auto">
        <div class="text-white main-header-content">
          <h1 class="font-bold text-5xl mb-5">Listen to Great Music!</h1>
          <p class="w-full md:w-8/12 mx-auto">
            Lorem ipsum dolor sit amet, consectetur adipiscing elit. Phasellus
            et dolor mollis, congue augue non, venenatis elit. Nunc justo eros,
            suscipit ac aliquet imperdiet, venenatis et sapien. Duis sed magna
            pulvinar, fringilla lorem eget, ullamcorper urna.
          </p>
        </div>
      </div>

      <img class="relative block mx-auto mt-5 -mb-20 w-auto max-w-full" src="/assets/img/introduction-music.png" />
    </section>

    <!-- Main Content -->
    <section class="container mx-auto">
      <div class="bg-white rounded border border-gray-200 relative flex flex-col">
        <div class="px-6 pt-6 pb-5 font-bold border-b border-gray-200">
          <span class="card-title">Songs</span>
          <!-- Icon -->
          <i class="fa fa-headphones-alt float-right text-green-400 text-xl"></i>
        </div>
        <!-- Playlist -->
        <ol id="playlist">
          <AppSongItme v-for="song in songs" :key="song.docID" :song="song"></AppSongItme>
        </ol>
        <!-- .. end Playlist -->
      </div>
    </section>
  </div>
</template>

<script>
import { songsCollection } from '../includes/firebase'
import AppSongItme from '@/components/SongItem.vue'
export default {
  name: 'Home',
  components: {
    AppSongItme,
  },
  data() {
    return {
      songs: [],
      maxPerPage: 30,
      pendingRequest: false,
    }
  },
  methods: {
    async getSongs() {
      // 请求过程中不再触发请求
      if (this.pendingRequest) {
        return
      }
      this.pendingRequest = true
      let snapshots
      if (this.songs.length) {
        const lastDoc = await songsCollection.doc(this.songs[this.songs.length - 1].docID).get() // DocumentSnapshot
        snapshots = await songsCollection
          .orderBy('modified_name')
          .startAfter(lastDoc)
          .limit(this.maxPerPage)
          .get() // QuerySnapshots
      } else {
        snapshots = await songsCollection
          .orderBy('modified_name')
          .limit(this.maxPerPage)
          .get() // QuerySnapshots
      }
      snapshots.forEach((document) => {
        this.songs.push({
          ...document.data(),
          docID: document.id,
        })
      })
      this.pendingRequest = false
    },
    handleScroll() {
      const { offsetHeight, scrollTop } = document.documentElement;
      const { innerHeight } = window;
      const bottomOfWindow = Math.round(scrollTop) + innerHeight === offsetHeight
      if (bottomOfWindow) {
        this.getSongs();
      }
    },
  },
  async created() {
    await this.getSongs()
    window.addEventListener('scroll', this.handleScroll)
  },
  beforeUnmount() {
    window.removeEventListener('scroll', this.handleScroll)
  },
}
</script>