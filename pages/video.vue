<template>
  <v-container style="max-width: 100%">
    <v-row>
      <login-dialog :sessionId="sessionId" @submit="setSessionId" @snackbar="createSnackbar"/>
      <v-col md="6">
        <v-row>
          {{this.videoNum}} {{this.keyframe}}
        </v-row>
        <v-row>
          <video ref="vidRef" id="vidRef" controls autoplay style="width: 430px; height: 250px">
            <source :src="videoUrl" type="video/mp4">
          </video>
        </v-row>
        <!--v-row>
          <v-select :items="videoList" v-model="videoNum" @change="updateVideoUrl"/>
          <v-select :items="keyframeList" v-model="keyframe" @change="updateKeyframe"/>
        </v-row-->
        <v-row>
          <v-tabs v-model="tab" class="transparent">
            <v-tab v-for="item in queryTabs" class="transparent">{{ item }}</v-tab>
          </v-tabs>
          <v-tabs-items v-model="tab" class="transparent">
            <v-tab-item key="ColorGrid">
              <color-grid :colors="colors" :background-image="backgroundImage" @query="appendQueryResults"
                          @snackbar="createSnackbar"/>
            </v-tab-item>
            <v-tab-item key="Sketchpad">
              <sketchpad :colors="colors" :objects="objects" :background-image="backgroundImage" @query="appendQueryResults" @snackbar="createSnackbar"/>
            </v-tab-item>
            <v-tab-item key="ObjectNumber">
              <object-number :objects="objects" @query="appendQueryResults" @snackbar="createSnackbar"/>
            </v-tab-item>
            <v-tab-item key="Text">
              <v-text-field
                v-model="textInput"
                label="Input Text"
                @keyup.enter="searchText"
              />
              <v-checkbox label="Text in Video" v-model="searchByVideoText"></v-checkbox>
              <v-checkbox label="Audio in Video" v-model="searchByAudio"></v-checkbox>
              <v-checkbox label="Text in Title" v-model="searchByTitle"></v-checkbox>
              <v-checkbox label="Text in Description" v-model="searchByDescription"></v-checkbox>
              <v-checkbox label="Text in Tag" v-model="searchByTag"></v-checkbox>
              <v-checkbox label="Image Caption" v-model="searchByImageCapture"></v-checkbox>
              <v-btn @click="searchText">Search</v-btn>
              <span v-if="loading">Loading...</span>
              <!--v-select :items="queryResponseItems" v-model="videoNum" @change="updateVideoUrl"/-->
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
            <!--VideoId: {{ result.videoId }}-->
            <!--StartTime: {{ result.startTime }}-->
            <!--KeyframeId: {{ result.keyframeId }}-->
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
import LoginDialog from "../components/LoginDialog";

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
    LoginDialog,
    Sketchpad,
    ColorGrid,
    VideoPlayer
  },
  data: () => ({
    player: {},
    sessionId: '',
    textInput: '',
    videoNum: '00032',
    keyframe: '98',
    startTime: 0,
    videoUrl: process.env.VIDEO_SOURCE_URL + '/videos/00032/00032.mp4',
    thumbnailUrl: process.env.VIDEO_SOURCE_URL + '/thumbnails/00032/shot00032_1.png',
    searchByVideoText: false,
    searchByAudio: false,
    searchByDescription: false,
    searchByTitle: false,
    searchByTag: false,
    searchByImageCapture: false,
    showSnackbar: false,
    tab: null,
    queryTabs: [
      'Color Grid', 'Sketchpad', 'ObjectNumber', 'Text',
    ],
    text: 'I am in the tab!',
    snackbarText: '',
    snackbarColor: 'blue',
    videoList: ['00032', '00037', '00061', '00063', '00078', '00081', '00111', '00181', '00188', '00192', '00250', '01219'],
    keyframeList: ['0', '98', '99'],
    queryResponseItems: [],
    itemIndex: 0,
    queryResults: [],
    options: {quality: {default: '576p'}},
    colors: [],
    objects: ['cat'],
    loading: false,
    backgroundImage: 'https://banner2.cleanpng.com/20180224/jrw/kisspng-white-black-angle-pattern-floating-dot-background-with-snowflakes-stock-vect-5a914aca6c6064.3090225015194713064439.jpg'
  }),
  methods: {
    setSessionId(id) {
      this.sessionId = id
      console.log('Session Id set to: ', id)
    },
    async initColors() {
      try {
        let {results} = await this.$axios.$get('/api/getAllColors')
        console.log('GetAllColors: ', results)
        for (let k = 0; k < results.length; k++) {
          this.colors.push({rgb: "rgb(" + results[k][0] + ", " + results[k][1] + ", " + results[k][2] + ")"})
        }
      } catch (e) {
        console.log(e);
      }
    },
    async initObjects() {
      // Objects array
      try {
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
    updateKeyframe() {

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
      if (!(this.searchByVideoText || this.searchByAudio || this.searchByDescription || this.searchByTitle || this.searchByTag || this.searchByImageCapture)) {
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
        if (this.searchByAudio) {
          try {
            let response11 = await this.$axios.$get('/api/searchByAudio?text=' + this.textInput)
            console.log('SearchByAudio: ', response11)
            this.addVideoToList(response11)
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
        if (this.searchByImageCapture) {
          try {
            let response5 = await this.$axios.$get('/api/searchByImageCapture?text=' + this.textInput)
            console.log('SearchByImageCapture: ', response5)
            this.addVideoToList(response5)
          } catch (e) {
            console.log(e);
            this.createSnackbar(e.message, 'red')
          }
        }
        console.log(this.queryResponseItems);
        if (this.queryResponseItems.length > 0) {
          this.createSnackbar(this.queryResponseItems.length + " results found.", 'blue')
        } else {
          this.createSnackbar("No results.", 'blue')
        }
        this.loading = false
      }
    },
    addVideoToList(response) {
      for (let i = 0; i < response.results.length; i++) {
        //console.log('videoId: ', response.results[i].video_id)
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
      this.startTime = startTime
      var vid = document.getElementById("vidRef");
      vid.src = this.videoUrl
      vid.play()
      console.log('currentTime = ', vid.currentTime)
      vid.currentTime = startTime
      console.log('currentTime = ', vid.currentTime)
      this.createSnackbar('Timestamp: '+ startTime)
    },
    async login() {
      if (localStorage.sessionId) {
        this.sessionId = localStorage.sessionId;
      }
      console.log('Session Id set to: ', this.sessionId)
      //this.sessionId = process.env.SESSION_ID

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
