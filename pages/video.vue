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
        <v-btn @click="searchText(textInput)">Search</v-btn>
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
    <v-row class="text-center">

    </v-row>
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
    apiSpecificTextResult: '',
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
      let response = await this.$axios.$get('/api/specific-text?text=' + this.textInput)
      this.apiSpecificTextResult = response;
      this.queryResponseItems = [];
      for (let i=0; i < response.results.length; i++) {
        console.log(response.results[i].video_id)
        var s = response.results[i].video_id+"";
        while (s.length < 5) s = "0" + s;
        if (!this.queryResponseItems.includes(s)) this.queryResponseItems = this.queryResponseItems.concat(s);
      }
      console.log(this.queryResponseItems);
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
