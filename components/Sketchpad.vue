<template>
  <div>
    <div style="display: flex; flex-direction: row" >
      <v-autocomplete
        :items="objects"
        v-model="selectedObject"
        label="Objects"
        style="width: 300px"
      />
      <v-checkbox v-model="includeColor" label="Include Color"/>
        <input v-if="includeColor" v-model="hexColor" type="color" @change="selectColor" style="height: 50px; width: 50px; margin: 10px">
    </div>

    <!--table v-if="includeColor">
      <tr class="d-flex flex-row flex-wrap" style="max-width: 455px; border: 1px solid white">
        <td v-for="(color, index) in colors" class="colorBox" :id="index" :style={backgroundColor:color.rgb}
            @click="selectGridColor"/>
      </tr>
    </table -->
    <canvas @mousedown="startPainting" @mouseup="finishedPainting" id="canvas"/>
    <br>
    <v-btn @click="clear">Clear</v-btn>
    <v-btn @click="query">Query</v-btn>
    <span v-if="loading">Loading...</span>
  </div>
</template>

<script>
export default {
  name: "Sketchpad",
  props: {
    colors: Array,
    objects: Array,
    backgroundImage: String,
    maxResults: Number
  },
  data: () => ({
    vueCtx: null,
    vueCanvas: null,
    canvasHeight: 450,
    canvasWidth: 650,
    painting: false,
    startX: 0,
    startY: 0,
    endX: 0,
    endY: 0,
    selectedColor: "rgb(0,0,0)",
    hexColor: "#000",
    selectedObject: '',
    boxes: [],
    loading: false,
    includeColor: false
  }),
  async mounted() {
    this.initializeCanvas()
  },
  methods: {
    startPainting(e) {
      this.painting = true
      this.vueCtx.strokeStyle = this.selectedColor
      let coordinates = this.calculateCoordinates(e.clientX, e.clientY)
      this.startX = coordinates.x
      this.startY = coordinates.y
      console.log('start (', this.startX, ',', this.startY, ')');
      this.vueCtx.beginPath()
      this.vueCtx.moveTo(this.startX, this.startY)
    },
    finishedPainting(e) {
      this.painting = false
      let coordinates = this.calculateCoordinates(e.clientX, e.clientY)
      this.endX = coordinates.x
      this.endY = coordinates.y
      let width = this.endX - this.startX
      let height = this.endY - this.startY
      console.log('stop (', this.endX, ',', this.endY, ')');
      this.vueCtx.strokeRect(this.startX, this.startY, width, height)
      this.vueCtx.stroke()
      let c = this.getRGB(this.selectedColor)
      this.boxes.push({
        object: this.selectedObject,
        box: {
          x1: this.startX,
          y1: this.startY,
          x2: this.endX,
          y2: this.endY
        },
        color: {
          red: c.red,
          green: c.green,
          blue: c.blue
        },
      })
      console.log(this.boxes)
    },
    initializeCanvas() {
      let canvas = document.getElementById("canvas")
      canvas.height = this.canvasHeight
      canvas.width = this.canvasWidth
      this.vueCanvas = canvas
      let ctx = canvas.getContext("2d")
      ctx.strokeStyle = this.selectedColor
      ctx.lineWidth = 4
      ctx.lineCap = "round"
      this.vueCtx = ctx

      var img = new Image();
      img.onload = function () {
        ctx.drawImage(img, 0, 0);
      }
      img.src = this.backgroundImage; //transparent png
    },
    clear() {
      this.vueCtx.clearRect(0, 0, this.vueCanvas.width, this.vueCanvas.height)
      this.boxes = []
      this.initializeCanvas()
    },
    selectColor() {
      var color = this.hexColor
      console.log("OG color: ", color)
      color = this.hexToRGB(color)
      console.log(color)
      this.selectedColor = color
    },
    hexToRGB(hex) {
        var result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
        return result ?
         "rgb(" + parseInt(result[1], 16) + ", " + parseInt(result[2], 16) + ", " + parseInt(result[3], 16) + ")"
         : null;
    },
    selectGridColor(e) {
      console.log(e.target.id)
      this.selectedColor = this.colors[e.target.id].rgb
    },
    calculateCoordinates(x, y) {
      // TODO: Fix coordinates scaling
      let BB = this.vueCanvas.getBoundingClientRect()
      //console.log(BB)
      let mouseX = x - BB.left
      let mouseY = y - BB.top
      return {x: mouseX, y: mouseY}
    },
    getRGB(str) {
      console.log(str)
      var match = str.match(/rgb?\((\d{1,3}), ?(\d{1,3}), ?(\d{1,3})\)?(?:, ?(\d(?:\.\d?))\))?/);
      return match ? {
        red: match[1],
        green: match[2],
        blue: match[3]
      } : {};
    },
    formatRGB(red, green, blue) {
      return "rgb(" + red + ", " + green + ", " + blue + ")"
    },
    async query() {
      if (this.includeColor) {
        console.log('****API SEARCH BY COLOR SKETCH****', this.boxes[0])
        this.loading = true
        const payload = this.boxes[0].max_results = this.maxResults
        try {
          let response1 = await this.$axios.$post('/api/searchByColorSketch', payload)
          console.log('SearchByColorSketch: ', response1)
          if (response1.results.length > 0) {
            console.log('emit query: ', response1.results.length)
            this.$emit('query', response1);
            this.$emit('snackbar', response1.results.length + " results found.")
          } else {
            console.log('emit snackbar: ', response1.results.length)
            this.$emit('snackbar', 'No results')
          }
        } catch (e) {
          console.log(e);
          this.$emit('snackbar', e.message, 'red')
        }
        this.loading = false
      } else {
        if (this.boxes.length === 1) {
          console.log('****API SEARCH BY OBJECT SKETCH****', this.boxes[0])
          this.loading = true
          var box = this.boxes[0].box
          var data = {object: this.boxes[0].object, sketch: {x1: box.x1, y1: box.y1, x2: box.x2, y2: box.y2}, max_results: this.maxResults}
          try {
            let response2 = await this.$axios.$post('/api/searchByObjectSketch', data)
            console.log('SearchByObjectSketch Results: ', response2.results)
            if (response2.results.length > 0) {
              console.log('emit query: ', response2.results.length)
              this.$emit('query', response2);
              this.$emit('snackbar', response2.results.length + " results found.")
            } else {
              console.log('emit snackbar: ', response2.results.length)
              this.$emit('snackbar', 'No results')
            }
          } catch (e) {
            console.log(e);
            this.$emit('snackbar', e.message, 'red')
          }
          this.loading = false
        } else if (this.boxes.length === 2) {
          this.loading = true
          var boxes = this.boxes
          var data = {
            object1: boxes[0].object,
            sketch1: {
              x1: boxes[0].box.x1,
              y1: boxes[0].box.y1,
              x2: boxes[0].box.x2,
              y2: boxes[0].box.y2
            },
            object2: boxes[1].object,
            sketch2: {
              x1: boxes[1].box.x1,
              y1: boxes[1].box.y1,
              x2: boxes[1].box.x2,
              y2: boxes[1].box.y2
            },
            max_results: this.maxResults
          }
          console.log('****API SEARCH BY TWO OBJECTS SKETCH****', data)

          try {
            let response2 = await this.$axios.$post('/api/searchByTwoObjects', data)
            console.log('searchByTwoObjects Results: ', response2.results)
            if (response2.results.length > 0) {
              console.log('emit query: ', response2.results.length)
              this.$emit('query', response2);
              this.$emit('snackbar', response2.results.length + " results found.")

            } else {
              console.log('emit snackbar: ', response2.results.length)
              this.$emit('snackbar', 'No results')
            }
          } catch (e) {
            console.log(e);
            this.$emit('snackbar', e.message, 'red')
          }
          this.loading = false
        } else if (this.boxes.length === 3) {
          console.log('****API SEARCH BY THREE OBJECTS SKETCH****', this.boxes[0])
          this.loading = true
          var boxes = this.boxes
          var data = {
            object1: boxes[0].object,
            sketch1: {
              x1: boxes[0].box.x1,
              y1: boxes[0].box.y1,
              x2: boxes[0].box.x2,
              y2: boxes[0].box.y2
            },
            object2: boxes[1].object,
            sketch2: {
              x1: boxes[1].box.x1,
              y1: boxes[1].box.y1,
              x2: boxes[1].box.x2,
              y2: boxes[1].box.y2
            },
            object3: boxes[2].object,
            sketch3: {
              x1: boxes[2].box.x1,
              y1: boxes[2].box.y1,
              x2: boxes[2].box.x2,
              y2: boxes[2].box.y2
            },
            max_results: this.maxResults
          }
          try {
            let response2 = await this.$axios.$post('/api/searchByThreeObjects', data)
            console.log('searchByThreeObjects Results: ', response2.results)
            if (response2.results.length > 0) {
              console.log('emit query: ', response2.results.length)
              this.$emit('query', response2);
              this.$emit('snackbar', response2.results.length + " results found.")

            } else {
              console.log('emit snackbar: ', response2.results.length)
              this.$emit('snackbar', 'No results')
            }
          } catch (e) {
            console.log(e);
            this.$emit('snackbar', e.message, 'red')
          }
          this.loading = false
        }
      }

    }
  }
}
</script>

<style scoped>
#canvas {
  /* 650 x 450 ratio */
  height: 450px;
  width: 650px;
  background: white;
}

.colorBox {
  height: 50px;
  width: 50px;
}
</style>
