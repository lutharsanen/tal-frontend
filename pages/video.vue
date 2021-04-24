<template>
  <v-container style="max-width: 100%">
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
        <v-checkbox label="Text in Tag" v-model="searchByTag"></v-checkbox>
        <v-btn @click="searchText">Search</v-btn>
        <v-select
          v-model="itemIndex"
          :items="queryResponseItems"
          label="Select Response Video"
          @input="submitVideoIndex"
        />

        <!-- SUBMIT SECTION -->
        <div style="margin: 2rem">{{currentTimestamp}} s</div>
        <v-btn @click="getCurrentTime">Get Timestamp</v-btn>
        <v-btn @click="finalSubmission">Submit</v-btn>
      </v-col>
      <v-col md="9" id="vue-plyrLocalhost" class="text-center">
        <v-row>
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
              :src="videoLink"
              type="video/mp4"
            />
          </video>
        </vue-plyr>
        <v-btn @click="testMethod">Test Button</v-btn>
        </v-row>
        <v-row>
          <v-col v-for="result in queryResults" @click="updateVideo(result.videoId, result.startTime)" class="videoTile">
            <v-img :src="updateThumbnailUrl(result.videoId, result.keyframeId)"></v-img>
            VideoId: {{result.videoId}}
            StartTime: {{result.startTime}}
            KeyframeId: {{result.keyframeId}}
          </v-col>
        </v-row>
      </v-col>
    </v-row>
    <v-row class="text-center"></v-row>
    <v-snackbar
      v-model="showSnackbar"
      timeout="2500"
      :color="snackbarColor"
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
    this.$refs.plyr.player.on("ready", this.testMethod())
    this.login()
  },
  components: {
    VideoPlayer
  },
  data: () => ({
    sessionId: '',
    textInput: '',
    videoNum: '00032',
    videoUrl: process.env.VIDEO_SOURCE_URL + '/videos/videos/00032/00032.mp4',
    thumbnailUrl: process.env.VIDEO_SOURCE_URL + '/thumbnails/thumbnails/00032/shot00032_1.png',
    currentTimestamp: 0,
    searchByVideoText: false,
    searchByDescription: false,
    searchByTitle: false,
    searchByTag: false,
    showSnackbar: false,
    snackbarText: '',
    snackbarColor: 'blue',
    lessItems: ['00032','00037', '00061', '00063', '00078', '00081', '00111', '00181', '00188', '00192',],
    queryResponseItems: [],
    itemIndex: 0,
    queryResults: [],
    options: { quality: { default: '576p' }}
  }),
  methods: {
    createSnackbar(text, color = 'blue') {
      this.snackbarText = text
      this.snackbarColor = color
      this.showSnackbar = true
    },
    submitVideoNum() {
      this.updateVideoUrl()
      this.updateVideo()
    },
    submitVideoIndex() {
      //console.log('index: ',this.itemIndex)
      //console.log('startTime: ',this.queryResults[this.itemIndex].startTime)
      const startTime = this.queryResults[this.itemIndex].startTime
      this.videoNum = this.queryResults[this.itemIndex].videoId
      this.updateVideoUrl()
      this.updateVideo(startTime)
    },
    updateVideoUrl(videoId = this.videoNum) {
      this.videoUrl = process.env.VIDEO_SOURCE_URL + '/videos/videos/' + videoId + '/' + videoId + '.mp4'
      console.log('***** GET VIDEO URL *****', this.videoUrl)
    },
    updateThumbnailUrl(videoId, keyframeId) {
      console.log('***** UPDATE THUMBNAIL URL *****')
      const thumbUrl = process.env.VIDEO_SOURCE_URL + '/thumbnails/thumbnails/' + videoId + '/shot' + videoId + keyframeId + '.png'
      console.log(thumbUrl)
      return thumbUrl
    },
    updateThumbnailLink(videoId = this.videoNum, keyframeId = 0) {
      return process.env.VIDEO_SOURCE_URL + '/thumbnails/thumbnails/' + videoId + '/shot' + videoId + keyframeId + '.png'
    },
    async searchText() {
      if (!(this.searchByVideoText || this.searchByDescription || this.searchByTitle || this.searchByTag)) {
        this.createSnackbar("Please select a query.", 'red')
      } else {
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
        if (this.queryResponseItems.length > 0) {
          this.createSnackbar("Success! Select a video.", 'green')
        } else {
          this.createSnackbar("No results.", 'blue')
        }
      }
    },
    addVideoToList(response) {
      for (let i=0; i < response.results.length; i++) {
        var s = response.results[i].video_id ? response.results[i].video_id+"" : response.results[i].id+"";
        while (s.length < 5) s = "0" + s;
        var item = {text: s, value: i};
        this.queryResponseItems = this.queryResponseItems.concat(item);
        var video = {videoId: s, startTime: response.results[i].start_time, keyframeId: response.results[i].keyframe_id.slice(response.results[i].keyframe_id.indexOf('_'), -4)};
        this.queryResults = this.queryResults.concat(video);
      }
    },
    updateVideo(videoId = this.videoNum, startTime = 0) {
      console.log('***** UPDATE VIDEO *****', videoId, startTime)
      this.updateVideoUrl(videoId)
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
      this.$refs.plyr.player.currentTime = 100
      console.log('currentTime = ', this.$refs.plyr.player.currentTime)
    },
    async login() {
      if (localStorage.sessionId) {
        this.sessionId = localStorage.sessionId;
      }
      else {
        console.log('/api/login')
        try {
          let response = await this.$axios.$post(process.env.DRES_URL + '/api/login', { username: 'participant', password: 'password'})
          console.log(response);
          this.sessionId = response.sessionId;
          localStorage.sessionId = response.sessionId;
          console.log('sessionid: ', this.sessionId);
        } catch (e) {
          console.log(e)
        }
      }

    },
    async finalSubmission() {
      console.log('/submit?session=/' + this.sessionId + '&item=' + this.videoNum + '&timecode=' + this.convertTime(this.$refs.plyr.player.currentTime))
      try {
        let response = await this.$axios.$get(process.env.DRES_URL + '/submit?session=' + this.sessionId + '&item=' + this.videoNum + '&timecode=' + this.convertTime(this.$refs.plyr.player.currentTime))
        console.log(response)
        this.createSnackbar(response.description, 'green')
      } catch (e) {
        console.log(JSON.stringify(e.response.data))
        this.createSnackbar(e.response.data.description, 'red')
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
      this.$refs.plyr.player.currentTime = 100
    }
  },
  computed: {
    videoLink: function () {
      return process.env.VIDEO_SOURCE_URL + '/videos/videos/' + this.videoNum + '/' + this.videoNum + '.mp4'
    },
  }
}
</script>

<style scoped>

.videoTile {
  max-width: 150px;
}

</style>
