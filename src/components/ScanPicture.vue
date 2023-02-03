<template>
  <div class="content-scan">
    <div class="box-load" v-show="loading">
      <form class="form" name="uploader" enctype="multipart/form-data" method="POST">
        <div
            @dragenter="preventDropStart"
            @dragover="preventDropStart"
            @dragleave="preventDropEnd"
            @drop="prevent"
            class="input__wrapper">
          <input
              @change="loadPicture"
              name="file"
              type="file"
              id="input__file"
              accept=".png,.jpg,.webp"
              class="input input__file">
          <label for="input__file" class="input__file-button">
          <span
              class="input__file-button-text">Выберите файл или перетащите его сюда <br> <strong>(.png, .jpg, .webp)</strong></span>
          </label>
        </div>
      </form>
      <div
          v-show="history.length"
          @click="historyMenuProcess"
          class="box-load-history">
        История сканирования
      </div>
    </div>
    <div class="box-content" v-show="!loading">
      <div class="box-content-click-zone">
        <div class="info-content">
          <div className="image">
            <div class="load-process" v-show="this.loadProcess">Загрузка картинки <span></span></div>
          </div>
          <div className="info">
            <table className="table">
              <tr>
                <td>X</td>
                <td>{{ offsetX >= 0 ? offsetX : 0 }}</td>
              </tr>
              <tr>
                <td>Y</td>
                <td>{{ offsetY >= 0 ? offsetY : 0 }}</td>
              </tr>
              <tr>
                <td>R</td>
                <td>{{ pixelColorRGB[0] ? pixelColorRGB[0] : 0 }}</td>
              </tr>
              <tr>
                <td>G</td>
                <td>{{ pixelColorRGB[1] ? pixelColorRGB[1] : 0 }}</td>
              </tr>
              <tr>
                <td>B</td>
                <td>{{ pixelColorRGB[2] ? pixelColorRGB[2] : 0 }}</td>
              </tr>
              <tr>
                <td>Alpha</td>
                <td>{{ pixelColorRGB[3] ? pixelColorRGB[3] : 0 }}</td>
              </tr>
              <tr>
                <td>HEX-код</td>
                <td>{{ pixelColorHEX != '' ? pixelColorHEX : 0 }}</td>
              </tr>
            </table>
            <h4>Цвет</h4>
            <div className="block"></div>
            <div
                :class="{
        'hide': stop == false
          }"
                class="buttons">
              <button class="button" @click.stop="copy(pixelColorRGB)">Скопировать RGBA</button>
              <button class="button" @click.stop="copy(pixelColorHEX)">Скопировать HEX</button>
              <button class="button" @click="reset">Сбросить цвет</button>
            </div>
          </div>
        </div>
        <div
            @click="cancelScan"
            v-show="cancel"
            class="cancel">Отменить сканирование</div>
      </div>
      <div class="statistic">
        <div class="statistic-table">
          <h3>Статистика</h3>
          <table class="stat-table">
            <tr>
              <td>Самый популярный цвет на картинке</td>
              <td v-show="!this.loadProcess">
                <div id="max-color" class="color color-table"></div>
                RGBA: <strong @click="copy(this.maxColorRGB)">({{ this.maxColorRGB }})</strong>
                HEX: <strong @click="copy(this.maxColorHEX)">{{ this.maxColorHEX }}</strong>
              </td>
              <td v-show="this.loadProcess">Загрузка...</td>
            </tr>
            <tr>
              <td>Пиксель (0, 0)</td>
              <td v-show="!this.loadProcess">
                <div id="color-1" class="color color-table"></div>
                RGBA: <strong @click="copy(this.tableColor[0])">({{ this.tableColor[0] }})</strong>
                HEX: <strong @click="copy(this.tableColor[1])">{{ this.tableColor[1] }}</strong>
              </td>
              <td v-show="this.loadProcess">Загрузка...</td>
            </tr>
            <tr>
              <td>Пиксель ({{ width }}, 0)</td>
              <td v-show="!this.loadProcess">
                <div id="color-2" class="color color-table"></div>
                RGBA: <strong @click="copy(this.tableColor[2])">({{ this.tableColor[2] }})</strong>
                HEX: <strong @click="copy(this.tableColor[3])">{{ this.tableColor[3] }}</strong>
              </td>
              <td v-show="this.loadProcess">Загрузка...</td>
            </tr>
            <tr>
              <td>Пиксель ({{ width }}, {{ height }})</td>
              <td v-show="!this.loadProcess">
                <div id="color-3" class="color color-table"></div>
                RGBA: <strong @click="copy(this.tableColor[4])">({{ this.tableColor[4] }})</strong>
                HEX: <strong @click="copy(this.tableColor[5])">{{ this.tableColor[5] }}</strong>
              </td>
              <td v-show="this.loadProcess">Загрузка...</td>
            </tr>
            <tr>
              <td>Пиксель (0, {{ height }})</td>
              <td v-show="!this.loadProcess">
                <div id="color-4" class="color color-table"></div>
                RGBA: <strong @click="copy(this.tableColor[6])">({{ this.tableColor[6] }})</strong>
                HEX: <strong @click="copy(this.tableColor[7])">{{ this.tableColor[7] }}</strong>
              </td>
              <td v-show="this.loadProcess">Загрузка...</td>
            </tr>
          </table>
        </div>
        <div class="statistic-head">
          <h3>Поиск цвета по пикселю</h3>
          <form @submit.prevent="search" class="search">
            <input type="text" placeholder="x, y">
            <button type="submit">Поиск</button>
          </form>
        </div>
        <div class="statistic-info">
          <div class="statistic-info-done" v-show="searchProcess">
            <div>На пикселе <span @click="copy(`${x}, ${y}`)">({{ x }}, {{ y }})</span> был найден цвет </div>
            <div class="color"></div>
            <div>RGBA:
              <span @click="copy(searchColorRGB)">({{ searchColorRGB }})</span> HEX:
              <span @click="copy(searchColorHEX)">{{ searchColorHEX }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="drop"
        :class="{
    'open': loadFile
      }">Отпустите файл</div>
    <div class="copy"
         :class="{
        'open': coping === true
          }">Скопировано</div>
  </div>
  <div class="history-menu"
      @click="historyMenuProcess"
      v-show="historyMenu">
    <div
        @click.stop
        id="menu"
        class="history-menu-content">
      <h3>История сканирования</h3>
        <div
            class="history-item"
            @click="loadPicture(item)"
            v-for="item in history">
          <div><img :src="item.url" alt="image"></div>
          <div>Изображение № {{ item.index }}</div>
        </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      pixel: '',
      pixelColorRGB: '',
      offsetX: 0,
      offsetY: 0,
      pixelColorHEX: '',
      maxColorRGB: '',
      maxColorHEX: '',
      tableColor: [],
      maxPixel: '',
      stop: false,
      loading: true,
      loadFile: false,
      errorImage: false,
      cancel: false,
      width: 0,
      height: 0,
      context: 0,
      coping: false,
      x: -1,
      count: 1,
      y: -1,
      historyMenu: false,
      searchColorRGB: '',
      searchColorHEX: '',
      searchProcess: false,
      history: [],
      historyNameFile: []
    }
  },
  methods: {
    componentToHex(c) {
      let hex = c.toString(16);
      return hex.length == 1 ? "0" + hex : hex;
    },
    rgb2hex(r, g, b) {
      return ("#" + this.componentToHex(r) + this.componentToHex(g) + this.componentToHex(b)).toUpperCase();
    },
    copy(copy) {
      this.coping = true
      navigator.clipboard.writeText(copy)
      setTimeout(() => {
        this.coping = false
      }, 1000)
    },
    reset() {
      let block = document.querySelector('.block')
      this.stop = false
      this.pixel = ''
      this.pixelColorRGB = ''
      this.pixelColorHEX = ''
      block.style.background = 'transparent'
    },
    stopEvent(e) {
      e.preventDefault()
      e.stopPropagation()
    },
    preventDropStart(e) {
      this.stopEvent(e)
      this.loadFile = true
    },
    preventDropEnd(e) {
      this.stopEvent(e)
      this.loadFile = false
    },
    prevent(e) {
      this.stopEvent(e)
      let dt = e.dataTransfer
      let files = dt.files
      if (files.length) {
        this.loadPicture(files[0])
      }
    },
    cancelScan() {
      this.pixel = ''
      this.pixelColorRGB = ''
      this.pixelColorHEX = ''
      this.maxColorRGB = ''
      this.maxColorHEX = ''
      this.tableColor = []
      this.loadProcess = false,
      this.stop = false
      this.loading = true
      this.loadFile = false
      this.cancel = false
      this.coping = false
      this.x = -1
      this.y = -1
      this.searchProcess = false
      this.searchColorRGB = ''
      this.searchColorHEX = ''
      this.context = 0
      this.width = 0
      this.offsetX = 0
      this.offsetY = 0
      this.height = 0
      let canvas = document.querySelector('.canvas')
      let cord = document.querySelector('.cord')
      let block = document.querySelector('.block')
      if (block) {
        block.style.background = 'transparent'
      }
      if (canvas) {
        canvas.remove()
      }
      if (cord) {
        cord.remove()
      }
    },
    historyMenuProcess() {
      this.historyMenu = !this.historyMenu
      let menu = document.getElementById('menu')
      if (menu) {
        menu.scrollTo(0, 0)
      }
    },
    tableProcess() {
      let colorBlock = document.getElementById('color-1')
      let pixelRGB = this.getPixelColor(0, 0)
      this.tableColor.push(pixelRGB.toString().split(',').join(', '))
      if (colorBlock)
        colorBlock.style.background = `rgba(${this.tableColor[this.tableColor.length - 1]})`
      colorBlock = document.getElementById('color-2')
      let pixelHEX = this.rgb2hex(pixelRGB[0], pixelRGB[1], pixelRGB[2])
      this.tableColor.push(pixelHEX)
      pixelRGB = this.getPixelColor(this.width - 1, 0)
      this.tableColor.push(pixelRGB.toString().split(',').join(', '))
      if (colorBlock)
        colorBlock.style.background = `rgba(${this.tableColor[this.tableColor.length - 1]})`
      colorBlock = document.getElementById('color-3')
      pixelHEX = this.rgb2hex(pixelRGB[0], pixelRGB[1], pixelRGB[2])
      this.tableColor.push(pixelHEX)
      pixelRGB = this.getPixelColor(this.width - 1, this.height - 1)
      this.tableColor.push(pixelRGB.toString().split(',').join(', '))
      if (colorBlock)
        colorBlock.style.background = `rgba(${this.tableColor[this.tableColor.length - 1]})`
      colorBlock = document.getElementById('color-4')
      pixelHEX = this.rgb2hex(pixelRGB[0], pixelRGB[1], pixelRGB[2])
      this.tableColor.push(pixelHEX)
      pixelRGB = this.getPixelColor(0, this.height - 1)
      this.tableColor.push(pixelRGB.toString().split(',').join(', '))
      if (colorBlock)
        colorBlock.style.background = `rgba(${this.tableColor[this.tableColor.length - 1]})`
      pixelHEX = this.rgb2hex(pixelRGB[0], pixelRGB[1], pixelRGB[2])
      this.tableColor.push(pixelHEX)
    },
    getPixelColor(x, y) {
      if (this.context != 0)
        return this.context.getImageData(x, y, 1, 1).data
    },
    loadPicture(files) {
      let objectURL = ''
      this.cancel = true
      this.loadProcess = true
      this.loading = false
      this.loadFile = false
      let nameFile = ''
      if (files.target) {
        let file = files.target.value.split('\\')
        nameFile = file[file.length - 1]
      }
      let url = files
      if (files.target) {
        url = files.target.files[0]
        files.target.value = ''
      }
      if (files.url) {
        url = files.url
        objectURL = files.url
        this.historyMenu = !this.historyMenu
      } else {
        objectURL = window.URL.createObjectURL(url);
        if (this.historyNameFile.find(t => t === nameFile) == undefined) {
          this.historyNameFile.push(nameFile)
          this.history.push({ url: objectURL, index: this.count++})
        }
      }
      let image = new Image();
      image.src = objectURL;

      image.onload = () => {
        let canvas = document.createElement("canvas");
        let cord = document.createElement("div");
        canvas.classList.add('canvas')
        cord.classList.add('cord')
        let container = document.querySelector('.box-content')
        let block = document.querySelector('.block')
        canvas.width = image.width;
        canvas.height = image.height;
        this.width = image.width
        this.height = image.height;
        cord.innerHTML = `${this.width} × ${this.height}`

        if (this.width > window.innerWidth || this.height > window.innerHeight) {
          alert('Картинка оказалась больше вашего экрана, уменьшите масштаб страницы')
        }

        let context = canvas.getContext("2d");
        context.drawImage(image, 0, 0, image.width, image.height);
        this.canvas = context
        this.context = context
        container.querySelector('.image').appendChild(cord);
        container.querySelector('.image').appendChild(canvas);

        let mas = []
        let pixels = context.getImageData(0, 0, image.width, image.height).data
        for (let i = 0; i < pixels.length; i = i + 4) {
          let pixel = `${pixels[i]}, ${pixels[i + 1]}, ${pixels[i + 2]}, ${pixels[i + 3]}`
          mas.push(pixel)
        }

        function getLongestChain(arr) {
          let main = {};
          let temp = {};

          for (let i = 0; i < arr.length; i++) {
            if (arr[i] === temp.name) {
              temp.count++;
            } else {
              if (temp.count > (main.count || 0)) {
                main = cloneObject(temp);
              }
              temp = ChainData(arr[i], i);
            }
          }
          return main;
        }
        function ChainData(name, index) {
          return {
            name: name,
            count: 1,
            index: index,
          }
        }
        function cloneObject(obj) {
          let x = {};
          for (let key in obj) {
            x[key] = obj[key];
          }
          return x;
        }

        this.maxColorRGB = getLongestChain(mas).name
        let maxColor = document.getElementById('max-color')
        if (maxColor)
          maxColor.style.background = `rgba(${this.maxColorRGB})`
        let pixelMaxHEX = this.maxColorRGB.split(', ')
        this.maxColorHEX = this.rgb2hex(Number(pixelMaxHEX[0]), Number(pixelMaxHEX[1]), Number(pixelMaxHEX[2]))
        this.tableProcess()

        this.loadProcess = false

        canvas.addEventListener('mousemove', (e) => {
          canvas.style.cursor = 'crosshair'
          if (!this.stop) {
            this.pixel = `(${e.offsetX}, ${e.offsetY})`
            let pixelID = this.getPixelColor(e.offsetX, e.offsetY)
            this.offsetX = e.offsetX
            this.offsetY = e.offsetY
            this.pixelColorRGB = pixelID
            this.pixelColorHEX = this.rgb2hex(this.pixelColorRGB[0], this.pixelColorRGB[1], this.pixelColorRGB[2])
            block.style.background = `rgba(${pixelID[0]}, ${pixelID[1]}, ${pixelID[2]}, ${pixelID[3]})`
          }
        })
        canvas.addEventListener('click', (e) => {
          e.stopPropagation()
          let pixelID = this.getPixelColor(e.offsetX, e.offsetY)
          this.offsetX = e.offsetX
          this.offsetY = e.offsetY
          this.pixel = `(${e.offsetX}, ${e.offsetY})`
          this.pixelColorRGB = pixelID
          this.pixelColorHEX = this.rgb2hex(this.pixelColorRGB[0], this.pixelColorRGB[1], this.pixelColorRGB[2])
          block.style.background = `rgba(${pixelID[0]}, ${pixelID[1]}, ${pixelID[2]}, ${pixelID[3]})`
          this.stop = true
        })

        console.clear()
      }
    },
    search(e) {
      let input = e.target.querySelector('input')
      let statistic = document.querySelector('.statistic-info-done')
      let block = statistic.querySelector('.color')
      let pixels = input.value.split(' ').join('').split(',')
      input.value = ''
      this.x = pixels[0]
      this.y = pixels[1]
      if (this.x >= 0 && this.y >= 0 && this.x <= this.width && this.y <= this.height && pixels.length === 2) {
        let x = Number(this.x)
        let y = Number(this.y)
        if (x === this.width)
          x--
        if (y === this.height)
          y--
        this.searchColorRGB = this.getPixelColor(x, y)
        this.searchColorHEX = this.rgb2hex(this.searchColorRGB[0], this.searchColorRGB[1], this.searchColorRGB[2])
        block.style.background = `rgba(${this.searchColorRGB[0]}, ${this.searchColorRGB[1]}, ${this.searchColorRGB[2]}, ${this.searchColorRGB[3]})`
        setTimeout(() => {
          statistic.scrollIntoView({
            behavior: "smooth",
            block: "start"
          });
        }, 100)
        this.searchColorRGB = this.searchColorRGB.toString().split(',').join(', ')
        this.searchProcess = true
      } else {
        this.searchProcess = false
      }
    }
  },
  mounted() {
    console.clear()
  }
}
</script>

<style lang="scss" scoped>
.load-process {
  span {
    margin: 1vw auto 0;
    display: block;
    border: 16px solid #f3f3f3;
    border-top: 16px solid black;
    border-radius: 50%;
    width: 120px;
    height: 120px;
    animation: spin 2s linear infinite;
  }

  @keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
  }
}
.statistic-table {
  margin: 0 0 1vw;
  td strong {
    cursor: pointer;
  }
  h3 {
    margin: 0 0 1vw;
  }
  tr td:first-child {
    font-weight: 600;
  }
}
.history-menu {
  position: fixed;
  top: 0;
  left: 0;
  z-index: 3;
  background-color: rgba(0, 0, 0, 0.8);
  width: 100%;
  height: 100%;
  font-weight: 600;
  border: 1px solid black;
  font-size: 1.4vw;
  display: flex;
  align-items: center;
  justify-content: center;
  .history-menu-content {
    background: #fff;
    color: #000;
    max-width: 60vw;
    max-height: 80vh;
    width: 100%;
    padding: 2vw;
    overflow-x: hidden;
    overflow-y: scroll;
    border-radius: 1vw;
    h3 {
      text-align: center;
      margin: 0 0 2vw;
    }
    .history-item {
      display: flex;
      align-items: center;
      gap: 1vw;
      height: 100vh;
      max-height: 25vh;
      padding: 1vw;
      border-radius: 1vw;
      cursor: pointer;
      transition: all 0.3s;
      border: 1px solid white;
      overflow: hidden;
      &:hover {
        transform: translate(0.25vw, 0);
        border: 1px solid black;
      }
      div:first-child {
        width: 15vw;
        display: flex;
        align-items: center;
        justify-content: center;
      }
      img {
        max-height: 100%;
        max-width: 100%;
      }
      &:not(:last-child) {
        margin: 0 0 1vw;
      }
    }
  }
}
.search {
  border-radius: 2vw;
  width: max-content;
  border: 1px solid black;
  display: flex;
  font-size: 1.3vw;
  input, input:focus {
    background: transparent;
    border-radius: 5px;
    padding: 0.5vw 1vw;
    outline: 0;
    font-size: 1.3vw;
    border: 0;
  }
  button {
    border-radius: 0 2vw 2vw 0;
    background: transparent;
    outline: 0;
    border: 0;
    font-size: 1.3vw;
    cursor: pointer;
    font-weight: 600;
    border-left: 1px solid black;
    padding: 1vw;
  }
}
.statistic {
  font-size: 1.3vw;
}
.statistic-head {
  display: flex;
  gap: 1vw;
  flex-wrap: wrap;
  align-items: center;
  margin: 0 0 1vw 0;
}
.color {
  display: block;
  width: 1vw;
  height: 1vw;
  margin: 0 0.5vw;
  &.color-table {
    display: inline-block;
    margin: 0 0.5vw 0 0;
  }
}
.statistic-info-done {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  margin: 0 0 0.5vw;
  span {
    font-weight: 600;
    cursor: pointer;
  }
}
.content-scan {
  flex: 1 1 auto;
  display: flex;
  align-items: center;
  justify-content: center;
}
.cancel, .drop, .box-load-history, .copy {
  font-weight: 600;
  border: 1px solid black;
  padding: 0.5vw;
  font-size: 1.2vw;
  border-radius: 0.5vw;
}
.drop, .copy {
   -webkit-touch-callout: none;
   -webkit-user-select: none;
   -khtml-user-select: none;
   -moz-user-select: none;
   -ms-user-select: none;
   user-select: none;
 }
.form {
  margin: 0 0 1vw;
}
.box-load-history {
  width: max-content;
  margin: 0 auto;
  font-size: 1.5vw;
}
.drop {
  position: fixed;
  bottom: 1vw;
  left: 50%;
  opacity: 0;
  transition: all 0.3s;
  visibility: hidden;
  transform: translate(-50%, 200%);
  &.open {
    opacity: 1;
    visibility: visible;
    transform: translate(-50%, 0);
  }
}
.copy {
  transition: all 0.3s;
  position: fixed;
  bottom: 1vw;
  right: 1vw;
  background: white;
  visibility: hidden;
  z-index: 999;
  opacity: 0;
  transform: translate(0, 100%);
  &.open {
    opacity: 1;
    visibility: visible;
    transform: translate(0, 0);
  }
}
.input__wrapper {
  width: 100%;
  position: relative;
  margin: 15px 0;
  text-align: center;
  .input__file {
    opacity: 0;
    visibility: hidden;
    position: absolute;
  }
  .input__file-button-text {
    line-height: 1;
    margin-top: 1px;
    display: block;
    width: 100%;
    text-align: center;
    strong {
      font-size: 1.1vw;
      font-weight: 500;
    }
  }
  .input__file-button {
    background: transparent;
    padding: 20px;
    width: 50vw;
    height: 30vh;
    border: 1px solid black;
    font-size: 1.5vw;
    font-weight: 700;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    align-items: center;
    justify-content: center;
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
    -webkit-box-pack: start;
    -ms-flex-pack: start;
    justify-content: flex-start;
    border-radius: 1vw;
    cursor: pointer;
    margin: 0 auto;
  }
}
.button {
  margin: 0 0 1vw;
  width: 100%;
  background: transparent;
  border: 1px solid black;
  cursor: pointer;
  display: block;
  font-weight: 600;
  font-size: 1.2vw;
  padding: 0.5vw;
  border-radius: 0.5vw;
}
.buttons {
  &.hide {
    opacity: 0;
    visibility: hidden;
  }
}
.info-content {
  width: 100%;
  display: flex;
  gap: 1vw;
  align-items: center;
  justify-content: center;
}
.cancel {
  width: max-content;
  margin: 1vw auto 0;
  cursor: pointer;
}
.info {
  text-align: center;
  font-size: 1.3vw;
}
.table, .stat-table {
  white-space: nowrap;
  transition: all 0.3s;
  border-collapse: collapse;

  td {
    border-radius: 0.5vw;
    border: 1px solid black;
    padding: 0.5vw;
  }
}
.table {
  margin: 0 0 1vw 0;

  td {
    min-width: 8vw;
  }
}
.table tr td:first-child, .copy {
    font-weight: 600;
}
.block {
  margin: 1vw auto;
  width: 7vw;
  height: 7vw;
  border: 1px solid black;
  border-radius: 0.5vw;
}
.box-content-click-zone {
  padding: 1vw;
  margin: 0 0 1vw 0;
  border: 1px solid black;
  position: relative;
  border-radius: 0.5vw;
  z-index: 1;
  &::before, &::after {
    content: '';
    position: absolute;
    top: 50%;
    z-index: 1;
    left: 50%;
    transform: translate(-50%, -50%);
    background: white;
  }
  &::before {
    width: 105%;
    height: 85%;
  }
  &::after {
    width: 90%;
    height: 105%;
  }
}
.info-content, .cancel, .statistic {
  position: relative;
  z-index: 2;
}
.image {
  font-weight: 600;
  font-size: 1.3vw;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 10px;
  flex: 1 1 auto;
}
</style>