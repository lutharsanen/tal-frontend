<template>
  <v-container style="max-width: 100%">
    <v-row>
      <v-col md="6">
        <v-row>
          <!--vue-plyr ref="plyr" @player="setPlayer">
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
                :src="videoUrl"
                type="video/mp4"
              />
            </video>
          </vue-plyr-->
          <video ref="vidRef" id="vidRef" controls autoplay style="width: 390px; height: 230px">
            <source :src="videoUrl" type="video/mp4">
          </video>
          <v-btn @click="updateTime">Update Time</v-btn>
        </v-row>
        <v-row>
          <v-tabs v-model="tab" class="transparent">
            <v-tab v-for="item in queryTabs" class="transparent">{{ item }}</v-tab>
          </v-tabs>
          <v-tabs-items v-model="tab" class="transparent">
            <v-tab-item key="ColorGrid">
              <color-grid :colors="colors" @query="appendQueryResults"
                          @snackbar="createSnackbar"/>
            </v-tab-item>
            <v-tab-item key="Sketchpad">
              <sketchpad :colors="colors" :objects="objects" @query="appendQueryResults" @snackbar="createSnackbar"/>
            </v-tab-item>
            <v-tab-item key="Text">
              <v-text-field
                v-model="textInput"
                label="Input Text"
                @keyup.enter="searchText"
              />
              <v-checkbox label="Text in Video" v-model="searchByVideoText"></v-checkbox>
              <v-checkbox label="Text in Title" v-model="searchByTitle"></v-checkbox>
              <v-checkbox label="Text in Description" v-model="searchByDescription"></v-checkbox>
              <v-checkbox label="Text in Tag" v-model="searchByTag"></v-checkbox>
              <v-btn @click="searchText">Search</v-btn>
              <span v-if="loading">Loading...</span>
            </v-tab-item>
          </v-tabs-items>
        </v-row>
        <v-row align="center" justify="center">
          <v-btn @click="finalSubmission" color="purple" style="height: 70px; width: 130px">Submit</v-btn>
        </v-row>
      </v-col>
      <v-col md="6" id="vue-plyrLocalhost" class="text-center">
        <v-row>
          <v-col v-for="result in queryResults" @click="updateVideo(result.videoId, result.startTime)"
                 class="videoTile" style="padding: 0">
            <v-img :src="updateThumbnailUrl(result.videoId, result.keyframeId)"></v-img>
            <!--VideoId: {{ result.videoId }}
            StartTime: {{ result.startTime }}
            KeyframeId: {{ result.keyframeId }}-->
          </v-col>
        </v-row>
      </v-col>
    </v-row>

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
import Sketchpad from "../components/Sketchpad";
import ColorGrid from "../components/ColorGrid";

require('nuxt-video-player/src/assets/css/main.css')

export default {
  name: "video",
  mounted() {
    this.login()
    this.initColors()
    this.initObjects()
    //this.displayProtectedImage(process.env.VIDEO_SOURCE_URL + '/videos/00032/00032.mp4')
  },
  components: {
    Sketchpad,
    ColorGrid,
    VideoPlayer
  },
  data: () => ({
    player: {},
    sessionId: '',
    textInput: '',
    videoNum: '00032',
    videoUrl: process.env.VIDEO_SOURCE_URL + '/videos/00032/00032.mp4',
    thumbnailUrl: process.env.VIDEO_SOURCE_URL + '/thumbnails/00032/shot00032_1.png',
    searchByVideoText: false,
    searchByDescription: false,
    searchByTitle: false,
    searchByTag: false,
    showSnackbar: false,
    tab: null,
    queryTabs: [
      'Color Grid', 'Sketchpad', 'Text',
    ],
    text: 'I am in the tab!',
    snackbarText: '',
    snackbarColor: 'blue',
    lessItems: ['00032', '00037', '00061', '00063', '00078', '00081', '00111', '00181', '00188', '00192', '00250'],
    queryResponseItems: [],
    itemIndex: 0,
    queryResults: [],
    options: {quality: {default: '576p'}},
    colors: [],
    objects: [],
    loading: false
  }),
  methods: {
    async initColors() {
      try {
        let {results} = await this.$axios.$get('/api/getAllColors')
        console.log('GetAllColors: ', results)
        for (let k = 0; k < results.length; k++) {
          this.colors.push({rgb: "rgb(" + results[k][0] + ", " + results[k][1] + ", " + results[k][2] + ")"})
        }
        console.log('API Colors List', this.colors)
      } catch (e) {
        console.log(e);
      }
    },
    async initObjects() {
      // Objects array
      try {
        // let response1 = await this.$axios.$get('/api/getAllObjects')
        let {results} = await this.$axios.$get('/api/getAllObjects')
        console.log('GetAllObjects: ', results)
        for (let k = 0; k < results.length; k++) {
          this.objects.push(results[k])
        }
        console.log('Objects List', this.objects)
      } catch (e) {
        console.log(e);
      }
    },
    createSnackbar(text, color = 'blue') {
      this.snackbarText = text
      this.snackbarColor = color
      this.showSnackbar = true
    },
    setPlayer(player) {
      console.log('***** UPDATING PLAYER *****')
      this.player = player
    },
    fetchWithAuthentication(url) {
      const myHeaders = new Headers();
      myHeaders.set("Authorization", "Basic VEFMOlRhQWRMdS4yMDIx");
      const requestOptions = {
        method: 'GET',
        headers: myHeaders,
        redirect: 'follow',
        mode: 'no-cors'
      }
      console.log(url, requestOptions)
      let response = this.$axios.$get('https://tal.diskstation.me:5006/home/videos/00032/00032.mp4', {
        method: 'GET',
        headers: {
          common: {
            'Authorization': 'Basic VEFMOlRhQWRMdS4yMDIx'
          }
        },
        mode: 'no-cors',
        credentials: 'omit'
      })
      return response
      //return fetch(url, { headers: {"Authorization": "Basic VEFMOlRhQWRMdS4yMDIx"} });
    },
    async displayProtectedImage(imageUrl) {
      // Fetch the image.
      const response = await this.fetchWithAuthentication(imageUrl);

      // Create an object URL from the data.
      const blob = await response.blob();
      const objectUrl = URL.createObjectURL(blob);
      console.log(objectUrl)
      // Update the source of the video.
      //this.videoUrl = objectUrl
    },
    submitVideoNum() {
      this.updateVideo()
    },
    submitVideoIndex() {
      const startTime = this.queryResults[this.itemIndex].startTime
      const videoId = this.queryResults[this.itemIndex].videoId
      this.videoNum = videoId
      this.updateVideo(videoId, startTime)
    },
    updateVideoUrl(videoId = this.videoNum) {
      this.videoNum = videoId
      this.videoUrl = process.env.VIDEO_SOURCE_URL + '/videos/' + videoId + '/' + videoId + '.mp4'
      console.log('***** GET VIDEO URL *****', this.videoUrl)
    },
    updateThumbnailUrl(videoId, keyframeId = 1) {
      return process.env.VIDEO_SOURCE_URL + '/thumbnails/' + videoId + '/shot' + videoId + '_' + keyframeId + '.png'
    },
    appendQueryResults(response) {
      console.log('response from child: ', response)
      this.queryResults = []
      this.addVideoToList(response);
    },
    async searchText() {
      if (!(this.searchByVideoText || this.searchByDescription || this.searchByTitle || this.searchByTag)) {
        this.createSnackbar("Please select a query.", 'red')
      } else {
        this.loading = true
        this.queryResponseItems = [];
        this.queryResults = [];
        if (this.searchByVideoText) {
          try {
            let response1 = await this.$axios.$get('/api/searchByVideoText?text=' + this.textInput)
            console.log('SearchByVideoText: ', response1)
            this.addVideoToList(response1)
          } catch (e) {
            console.log(e);
            this.createSnackbar(e.message, 'red')
          }
        }
        if (this.searchByDescription) {
          try {
            let response2 = await this.$axios.$get('/api/searchByDescription?text=' + this.textInput)
            console.log('SearchByDescription: ', response2)
            this.addVideoToList(response2)
          } catch (e) {
            console.log(e);
            this.createSnackbar(e.message, 'red')
          }
        }
        if (this.searchByTitle) {
          try {
            let response3 = await this.$axios.$get('/api/searchByTitle?text=' + this.textInput)
            console.log('SearchByTitle: ', response3)
            this.addVideoToList(response3)
          } catch (e) {
            console.log(e);
            this.createSnackbar(e.message, 'red')
          }
        }
        if (this.searchByTag) {
          try {
            let response4 = await this.$axios.$get('/api/searchByTag?text=' + this.textInput)
            console.log('SearchByTag: ', response4)
            this.addVideoToList(response4)
          } catch (e) {
            console.log(e);
            this.createSnackbar(e.message, 'red')
          }
        }
        console.log(this.queryResponseItems);
        if (this.queryResponseItems.length > 0) {
          this.createSnackbar("Success! Select a video.", 'green')
        } else {
          this.createSnackbar("No results.", 'blue')
        }
        this.loading = false
      }
    },
    addVideoToList(response) {
      for (let i = 0; i < response.results.length; i++) {
        var s = response.results[i].video_id ? response.results[i].video_id + "" : response.results[i].id + "";
        while (s.length < 5) s = "0" + s;
        var item = {text: s, value: i};
        this.queryResponseItems = this.queryResponseItems.concat(item);
        var startTime = response.results[i].start_time ? response.results[i].start_time : 0;
        var keyframeId = response.results[i].keyframe_id ? response.results[i].keyframe_id : 1
        var video = {
          videoId: s,
          startTime: startTime,
          keyframeId: keyframeId
        };
        //console.log('video_id: ', video.videoId, '  exists?: ', this.queryResults.includes(video))
        if (!this.queryResults.includes(video)) {
          this.queryResults = this.queryResults.concat(video);
        }
      }
    },
    updateVideo(videoId = this.videoNum, startTime = 0) {
      console.log('***** UPDATE VIDEO *****', videoId, startTime)
      this.updateVideoUrl(videoId)
      var vid = document.getElementById("vidRef");
      vid.src = this.videoUrl
      /*this.$refs.vidRef.player.source = {
        type: 'video',
        title: 'Example title',
        sources: [
          {
            src: this.videoUrl,
            type: 'video/mp4',
            size: 720,
          },
        ],
      };*/
      vid.play()
      console.log('currentTime = ', vid.currentTime)
      vid.currentTime = startTime
      console.log('currentTime = ', vid.currentTime)
    },
    async login() {
      this.sessionId = 'node0v26qb8f2urd4w8ce07n1u7qc1'
      /*if (localStorage.sessionId) {
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
      }*/

    },
    async finalSubmission() {
      var vid = document.getElementById("vidRef");
      console.log('/submit?session=/' + this.sessionId + '&item=' + this.videoNum + '&timecode=' + this.convertTime(vid.currentTime))
      try {
        let response = await this.$axios.$get(process.env.DRES_URL + '/submit?session=' + this.sessionId + '&item=' + this.videoNum + '&timecode=' + this.convertTime(vid.currentTime))
        console.log(response)
        this.createSnackbar(response.description, 'green')
      } catch (e) {
        console.log(JSON.stringify(e.response.data))
        this.createSnackbar(e.response.data.description, 'red')
      }
    },
    convertTime(time) {
      const ss = (time % 60).toFixed(0).toString()
      const mm = (time / 60).toFixed(0).toString()
      const hh = (time / 3600).toFixed(0).toString()
      return (hh < 10 ? '0' + hh : hh) + ':' + (mm < 10 ? '0' + mm : mm) + ':' + (ss < 10 ? '0' + ss : ss) + ':00'
    },
    updateTime() {
      console.log('***TEST METHOD***');
      var vid = document.getElementById("vidRef");
      vid.currentTime = "10";
      console.log(vid.currentTime);
      vid.play();
      //vid.currentTime = 100
      //console.log('current time: ', vid.currentTime)
    }
  },
  computed: {
    videoLink: function () {
      return process.env.VIDEO_SOURCE_URL + '/videos/' + this.videoNum + '/' + this.videoNum + '.mp4'
    },
  }
}
</script>

<style scoped>

.videoTile {
  width: 150px;
}

.transparent {
  background: transparent;
}

</style>
