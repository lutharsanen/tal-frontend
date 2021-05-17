<template>
  <div>
    <!--Select a color: <input id="colorPicker" type="color" @change="selectColor" value="selectedColor">-->

    <tr class="d-flex flex-row flex-wrap" style="max-width: 455px; border: 1px solid white" >
      <td v-for="(color, index) in colors" class="colorBox" :id="index" :style={backgroundColor:color.rgb} @click="selectGridColor"/>
    </tr>
    <br> <canvas @mousedown="startPainting" @mouseup="finishedPainting" id="canvas"/>
    <br> <v-btn @click="clear">Clear</v-btn>
  </div>
</template>

<script>
export default {
  name: "Sketchpad",
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
    colors: [
      { name: 'Red', rgb: 'rgb(255,0,0)' },
      { name: 'Orange', rgb: 'rgb(255, 165, 0)' },
      { name: 'Yellow', rgb: 'rgb(255,255,0)' },
      { name: 'Lime', rgb: 'rgb(0,255,0)' },
      { name: 'Cyan', rgb: 'rgb(0,255,255)' },
      { name: 'Blue', rgb: 'rgb(0,0,255)' },
      { name: 'Fuchsia', rgb: 'rgb(255,0,255)' },
      { name: 'Silver', rgb: 'rgb(192,192,192)' },
      { name: 'White', rgb: 'rgb(255,255,255)' },

      { name: 'Maroon', rgb: 'rgb(128,0,0)' },
      { name: 'Brown', rgb: 'rgb(165, 42, 42)' },
      { name: 'Olive', rgb: 'rgb(128,128,0)' },
      { name: 'DarkGreen', rgb: 'rgb(0,128,0)' },
      { name: 'Turquoise', rgb: 'rgb(0,128,128)' },
      { name: 'DarkBlue', rgb: 'rgb(0,0,128)' },
      { name: 'Purple', rgb: 'rgb(128,0,128)' },
      { name: 'Gray', rgb: 'rgb(128,128,128)' },
      { name: 'Black', rgb: 'rgb(0,0,0)' },
    ],
    boxes: []
  }),
  mounted() {
    let canvas = document.getElementById("canvas")
    canvas.height = this.canvasHeight
    canvas.width = this.canvasWidth
    this.vueCanvas = canvas
    let ctx = canvas.getContext("2d")
    ctx.strokeStyle = this.selectedColor
    ctx.lineWidth = 4
    ctx.lineCap = "round"
    this.vueCtx = ctx
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
      this.vueCtx.strokeRect(this.startX,this.startY,width,height)
      this.vueCtx.stroke()
      this.boxes.push({label: "", x1: this.startX, y1: this.startY, x2: this.endX, y2: this.endY, color: this.selectedColor})
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
    calculateCoordinates(x,y) {
      // TODO: Fix coordinates scaling
      let BB=this.vueCanvas.getBoundingClientRect()
      //console.log(BB)
      let mouseX= x - BB.left
      let mouseY= y - BB.top
      return {x: mouseX, y: mouseY}
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
