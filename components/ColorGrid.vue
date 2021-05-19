<template>
  <div>
    Select a color: <input id="colorPicker" type="color" @change="selectColor" value="selectedColor">

    <tr class="d-flex flex-row flex-wrap" style="max-width: 455px; border: 1px solid white" >
      <td v-for="(color, index) in apiColors" class="colorBox" :id="index" :style={backgroundColor:color.rgb} @click="selectGridColor"/>
    </tr>
    <br>
    <canvas @mousedown="fillBox" id="canvasGrid"/>
    <br>
    <v-btn @click="clear">Clear</v-btn>
    <v-btn @click="query">Query</v-btn>
  </div>
</template>

<script>

export default {
  name: "ColorGrid",
  data: () => ({
    vueCtx: null,
    vueCanvas: null,
    canvasHeight: 300,
    canvasWidth: 500,
    apiColors: [],
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
    selectedColor: "#000",
    colorGridArray: []
  }),
  async mounted() {
    let canvas = document.getElementById("canvasGrid")
    canvas.height = this.canvasHeight
    canvas.width = this.canvasWidth
    this.vueCanvas = canvas
    let ctx = canvas.getContext("2d")
    ctx.strokeStyle = this.selectedColor
    ctx.lineWidth = 4
    ctx.lineCap = "round"
    this.vueCtx = ctx

    var img = new Image();
    img.onload = function() {
      ctx.drawImage(img, 0, 0);
    }
    img.src = "https://thediyfoodie.com/wp-content/uploads/2015/10/Light-Grey-Background-Texture-4.jpg"; //transparent png

    //Color array
    try {
      let response = await this.$axios.$get('/api/getAllColors')
      console.log('GetAllColors: ', response)
      for (let k = 0; k < response.result.length; k++) {
        this.apiColors.push({rgb: this.formatRGB(response.result[k][0], response.result[k][1], response.result[k][2])})
      }
      console.log('API Colors List', this.apiColors)
    } catch (e) {
      console.log(e);
    }

    for (let i = 0; i < 12; i++) {
      this.colorGridArray.push({"red": 0, "green": 0, "blue": 0})
    }
    console.log('Color Grid Array', this.colorGridArray)
  },
  methods: {
    clear() {
      this.vueCtx.clearRect(0, 0, this.vueCanvas.width, this.vueCanvas.height)
    },
    selectColor() {
      this.selectedColor = document.getElementById("colorPicker").value
      console.log(this.selectedColor)
    },
    selectGridColor(e) {
      console.log(e.target.id)
      this.selectedColor = this.colors[e.target.id].rgb
    },
    fillBox(e) {
      let col, row, x, y = 0
      let wid = this.canvasWidth / 4
      let hei = this.canvasHeight / 3
      let coordinates = this.calculateCoordinates(e.clientX, e.clientY)
      console.log('touched (', coordinates.x, ',', coordinates.y, ')');

      // Find column number
      if (coordinates.x < wid) {
        col = 0;
        x = 0
      } else if (coordinates.x < 2 * wid) {
        col = 1;
        x = wid
      } else if (coordinates.x < 3 * wid) {
        col = 2;
        x = 2 * wid
      } else {
        col = 3;
        x = 3 * wid
      }

      // Find row number
      if (coordinates.y < hei) {
        row = 0;
        y = 0
      } else if (coordinates.y < 2 * hei) {
        row = 1;
        y = hei
      } else if (coordinates.y < 3 * hei) {
        row = 2;
        y = 2 * hei
      } else {
        row = 3;
        y = 3 * hei
      }

      let num = row * 4 + col
      let c = this.getRGB(this.selectedColor)
      console.log("rgb:", c)
      this.colorGridArray[num] = {"red": c.red, "green": c.green, "blue": c.blue}
      console.log(this.colorGridArray)

      this.vueCtx.fillStyle = this.selectedColor
      this.vueCtx.fillRect(x, y, wid, hei)
    },
    calculateCoordinates(x, y) {
      let BB = this.vueCanvas.getBoundingClientRect()
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
      return "rgb("+ red +", " + green + ", " + blue + ")"
    },
    async query() {
      let data = {
        _0: this.colorGridArray[0],
        _1: this.colorGridArray[1],
        _2: this.colorGridArray[2],
        _3: this.colorGridArray[3],
        _4: this.colorGridArray[4],
        _5: this.colorGridArray[5],
        _6: this.colorGridArray[6],
        _7: this.colorGridArray[7],
        _8: this.colorGridArray[8],
        _9: this.colorGridArray[9],
        _10: this.colorGridArray[10],
        _11: this.colorGridArray[11]
      }
      console.log('****API SEARCH BY COLOR****', data)
      try {
        let response = await this.$axios.$post('/api/searchByColor', data)
        console.log('SearchByColor: ', response)
      } catch (e) {
        console.log(e);
      }
    }
  }
}
</script>

<style scoped>
#canvasGrid {
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
