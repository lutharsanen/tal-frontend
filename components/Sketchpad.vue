<template>
  <div>
    Select a color: <input id="colorPicker" type="color" @change="selectColor" value="selectedColor">
    <!--table>
      <tr>
        <td style="height: 50px; width: 50px; background-color: rgb(255,255,255)"/>
        <td style="height: 50px; width: 50px; background-color: rgb(255,0,0)"/>
        <td style="height: 50px; width: 50px; background-color: rgb(0,255,0)"/>
        <td style="height: 50px; width: 50px; background-color: rgb(0,0,255)"/>
        <td style="height: 50px; width: 50px; background-color: rgb(255,255,0)"/>
        <td style="height: 50px; width: 50px; background-color: rgb(0,255,255)"/>
        <td style="height: 50px; width: 50px; background-color: rgb(255,0,255)"/>
      </tr>
      <tr>
        <td style="height: 50px; width: 50px; background-color: rgb(128,128,128)"/>
        <td style="height: 50px; width: 50px; background-color: rgb(128,0,0)"/>
        <td style="height: 50px; width: 50px; background-color: rgb(0,128,0)"/>
        <td style="height: 50px; width: 50px; background-color: rgb(0,0,128)"/>
        <td style="height: 50px; width: 50px; background-color: rgb(128,128,0)"/>
        <td style="height: 50px; width: 50px; background-color: rgb(0,128,128)"/>
        <td style="height: 50px; width: 50px; background-color: rgb(128,0,128)"/>
        <td style="height: 50px; width: 50px; background-color: rgb(192,192,192)"/>
      </tr>
    </table-->
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
    painting: false,
    startX: 0,
    startY: 0,
    endX: 0,
    endY: 0,
    selectedColor: "#000",
    boxes: []
  }),
  mounted() {
    let canvas = document.getElementById("canvas")
    canvas.height = window.innerHeight
    canvas.width = window.innerWidth
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
   height: 300px;
   width: 500px;
   background: white;
 }
</style>
