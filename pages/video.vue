<template>
  <v-container>
    <v-row>
      <v-col md="3" class="text-center">
        <v-select
          v-model="videoNum"
          :items="lessItems"
          label="Select Video"
          @input="submitVideoNum"
        />
        <v-text-field
          v-model="textInput"
          label="Input Text"
        />
        <v-checkbox label="Text in Video" v-model="searchByVideoText"></v-checkbox>
        <v-checkbox label="Text in Title" v-model="searchByTitle"></v-checkbox>
        <v-checkbox label="Text in Description" v-model="searchByDescription"></v-checkbox>
        <!--v-checkbox label="Text in Tag" v-model="searchByTag"></v-checkbox-->
        <v-btn @click="searchText">Search</v-btn>
        <v-select
          v-model="videoNum"
          :items="queryResponseItems"
          label="Select Response Video"
          @input="submitVideoNum"
        />
        <div style="margin: 2rem">{{currentTimestamp}} s</div>
        <v-btn @click="getCurrentTime">Get Timestamp</v-btn>
        <v-btn @click="finalSubmission">Submit</v-btn>
      </v-col>
      <v-col id="vue-plyrLocalhost" class="text-center">
        <vue-plyr ref="plyr">
          <video
            controls
            crossorigin
            playsinline
            autoplay
            clickToPlay
            resetOnEnd
            muted
            fullscreen='{ "enabled": "false" }'
            speed='{ selected: 1, options: [0.5, 0.75, 1, 1.25, 1.5, 1.75, 2] }'
            data-plyr-config='{ "title": "Video Title" }'
          >
            <source
              size="720"
              :src="videoUrl"
              type="video/mp4"
            />
          </video>
        </vue-plyr>
        <v-btn @click="testMethod">Test Button</v-btn>
      </v-col>
    </v-row>
    <v-row class="text-center"></v-row>
    <v-snackbar
      v-model="showSnackbar"
      timeout="2500"
      color="blue"
      bottom
    >
      {{ snackbarText }}
    </v-snackbar>
  </v-container>
</template>

<script>
import VideoPlayer from 'nuxt-video-player'

require('nuxt-video-player/src/assets/css/main.css')

export default {
  name: "video",
  mounted() {
    console.log(this.$refs.plyr.player)
    //this.$refs.plyr.player.on('event', () => console.log('event fired'))
    this.$refs.plyr.player.on("ready", this.testMethod())
  },
  components: {
    VideoPlayer
  },
  data: () => ({
    textInput: '',
    videoNum: '00032',
    videoUrl: process.env.VIDEO_SOURCE_URL + '/videos/videos/00032/00032.mp4',
    currentTimestamp: 0,
    searchByVideoText: false,
    searchByDescription: false,
    searchByTitle: false,
    searchByTag: false,
    showSnackbar: false,
    snackbarText: '',
    lessItems: ['00032','00037', '00061', '00063', '00078', '00081', '00111', '00181', '00188', '00192',],
    queryResponseItems: []
  }),
  methods: {
    submitVideoNum() {
      this.getVideoUrl()
      this.updateVideo()
    },
    getVideoUrl() {
      this.videoUrl = process.env.VIDEO_SOURCE_URL + '/videos/videos/' + this.videoNum + '/' + this.videoNum + '.mp4'
    },
    async searchText() {
      if (!(this.searchByVideoText || this.searchByDescription || this.searchByTitle || this.searchByTag)) {
        this.snackbarText = "Please select a query."
        this.showSnackbar = true
      }
      this.queryResponseItems = [];
      if (this.searchByVideoText) {
        let response1 = await this.$axios.$get('/api/searchByVideoText?text=' + this.textInput)
        console.log('SearchByVideoText: ', response1)
        this.addVideoToList(response1)
      }
      if (this.searchByDescription) {
        let response2 = await this.$axios.$get('/api/searchByDescription?text=' + this.textInput)
        console.log('SearchByDescription: ', response2)
        this.addVideoToList(response2)
      }
      if (this.searchByTitle) {
        let response3 = await this.$axios.$get('/api/searchByTitle?text=' + this.textInput)
        console.log('SearchByTitle: ', response3)
        this.addVideoToList(response3)
      }
      if (this.searchByTag) {
        let response4 = await this.$axios.$get('/api/searchByTag?text=' + this.textInput)
        console.log('SearchByTag: ', response4)
        this.addVideoToList(response4)
      }
      console.log(this.queryResponseItems);
    },
    addVideoToList(response) {
      for (let i=0; i < response.results.length; i++) {
        console.log(response.results[i].video_id)
        var s = response.results[i].video_id ? response.results[i].video_id+"" : response.results[i].id+"";
        while (s.length < 5) s = "0" + s;
        if (!this.queryResponseItems.includes(s)) this.queryResponseItems = this.queryResponseItems.concat(s);
      }
    },
    updateVideo() {
      this.$refs.plyr.player.source = {
        type: 'video',
        title: 'Example title',
        sources: [
          {
            src: this.videoUrl,
            type: 'video/mp4',
            size: 720,
          },
        ],
      };
    },
    getCurrentTime() {
      this.currentTimestamp = this.$refs.plyr.player.currentTime;
    },
    async finalSubmission() {
      console.log('/submit?item=' + this.videoNum + '?timecode=' + this.convertTime(this.currentTimestamp))
      try {
        let response = await this.$axios.$get('/submit?item=' + this.videoNum + '?timecode=' + this.convertTime(this.currentTimestamp))
        console.log(response)
      } catch(e) {
        console.log(e)
      }
    },
    convertTime(time) {
      const ss = (time%60).toFixed(0).toString()
      const mm = (time/60).toFixed(0).toString()
      const hh = (time/3600).toFixed(0).toString()
      return (hh < 10 ? '0'+hh : hh) + ':' + (mm < 10 ? '0'+mm : mm) + ':' + (ss < 10 ? '0'+ss : ss) + ':00'
    },
    testMethod() {
      let time = this.$refs.plyr.player.currentTime;
      console.log(time);
      this.$refs.plyr.player.currentTime = 10 // Seeks to 10 seconds
      //this.$refs.plyr.player.forward(10);
    }
  }
}
</script>

<style scoped>

</style>
