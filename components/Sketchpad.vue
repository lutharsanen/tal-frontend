<template>
  <div>

    <v-autocomplete
      :items="objects"
      v-model="selectedObject"
      label="Objects"
    />
    <!--Select a color: <input id="colorPicker" type="color" @change="selectColor" value="selectedColor">-->

    <tr class="d-flex flex-row flex-wrap" style="max-width: 455px; border: 1px solid white">
      <td v-for="(color, index) in colors" class="colorBox" :id="index" :style={backgroundColor:color.rgb}
          @click="selectGridColor"/>
    </tr>
    <br>
    <canvas @mousedown="startPainting" @mouseup="finishedPainting" id="canvas"/>
    <br>
    <v-btn @click="clear">Clear</v-btn>
    <v-btn @click="query">Query</v-btn>

  </div>
</template>

<script>
export default {
  name: "Sketchpad",
  props: {
    colors: Array
  },
  data: () => ({
    vueCtx: null,
    vueCanvas: null,
    canvasHeight: 300,
    canvasWidth: 500,
    painting: false,
    startX: 0,
    startY: 0,
    endX: 0,
    endY: 0,
    selectedColor: "#000",
    objects: [],
    selectedObject: '',
    boxes: []
  }),
  async mounted() {
    let canvas = document.getElementById("canvas")
    canvas.height = this.canvasHeight
    canvas.width = this.canvasWidth
    this.vueCanvas = canvas
    let ctx = canvas.getContext("2d")
    ctx.strokeStyle = this.selectedColor
    ctx.lineWidth = 4
    ctx.lineCap = "round"
    this.vueCtx = ctx

    // Objects array
    try {
      // let response1 = await this.$axios.$get('/api/getAllObjects')
      let { results } = await this.$axios.$get('/api/getAllObjects')
      console.log('GetAllObjects: ', results)
      for (let k = 0; k < results.length; k++) {
        this.objects.push(results[k])
      }
      console.log('Objects List', this.objects)
    } catch (e) {
      console.log(e);
    }
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
    clear() {
      this.vueCtx.clearRect(0, 0, this.vueCanvas.width, this.vueCanvas.height)
      this.boxes = []
    },
    selectColor() {
      this.selectedColor = document.getElementById("colorPicker").value
      console.log(this.selectedColor)
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
      console.log('****API SEARCH BY COLOR SKETCH****', this.boxes[0])
      try {
        let response = await this.$axios.$post('/api/searchByColorSketch', this.boxes[0])
        console.log('SearchByColorSketch: ', response)
        if (response.results.length > 0) {
          console.log('emit query: ', response.results.length)
          this.$emit('query', response);
        } else {
          console.log('emit snackbar: ', response.results.length)
          this.$emit('snackbar', 'No results')
        }
      } catch (e) {
        console.log(e);
        this.$emit('snackbar', e.message, 'red')
      }
    }
  }
}
</script>

<style scoped>
#canvas {
  /* 650 x 450 ratio */
  height: 300px;
  width: 500px;
  background: white;
}

.colorBox {
  height: 50px;
  width: 50px;
}
</style>
