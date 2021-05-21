<template>
  <v-app>
    <v-app-bar
        app
        color="primary"
        dark
        v-show="!fullscreen"
    >
      <span>
        Ez-loop-slide
      </span>
    </v-app-bar>
    <v-main id="main" :style=background :class="padding">
      <v-carousel
          cycle
          hide-delimiters
          :interval="interval"
          height="100vh"
          show-arrows-on-hover
          v-show="fullscreen"
      >
        <v-carousel-item
            :transition="transition.effect"
            :reverse-transition="transition.effect"
            color="#303030"
            contain
            v-for="(image, i) in images"
            :key="i"
            :src="image.src"
        >
        </v-carousel-item>
      </v-carousel>
      <v-dialog
          v-model="dialog"
          max-width="290"
      >
        <v-card>
          <v-card-title class="headline">
            削除？
          </v-card-title>

          <v-card-text>
            リストから{{ selectedImage.filename }}を削除しますか？
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>

            <v-btn
                color="green darken-1"
                text
                @click="dialog = false; this.dialog = true"
            >
              いいえ
            </v-btn>

            <v-btn
                color="red darken-1"
                text
                @click="deleteImage(selectedImage.index)"
            >
              はい
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-row v-show="!fullscreen" >
        <v-col cols="12"
               class="pa-2,mx-4"
        >

          <v-card class="my-4">
            <v-row class="mx-6"  align="center">
              <v-spacer/>
              <v-btn color="primary" class="ma-2" @click="decrimentInput">
                -
              </v-btn>
              <v-col cols="1" class="pa-0">
                <v-text-field
                    label="周期(秒)"
                    type="number"
                    v-model="input"
                    max="999"
                    min="1"
                />
              </v-col>
              <h2>秒</h2>
              <v-btn color="primary" class="ma-2" @click="incrimentInput">
                +
              </v-btn>
              <v-col cols="2">
                <v-select
                    v-model="transition"
                    :items="transitionList"
                    :item-text="item => item.name"
                    label="Select"
                    persistent-hint
                    return-object
                    single-line
                ></v-select>
              </v-col>
              <v-btn
                  color="green darken-1"
                  dark
                  @click="changeFullscreen()">
                <span class="mr-5">開始</span>
                <v-icon>mdi-open-in-new</v-icon>
              </v-btn>
            </v-row>
          </v-card>
        </v-col>
        <v-col v-for="(image,index) in images"
               :key="index"
               xs="12"
               sm="6"
               md="3"
               class="pa-2"
        >
          <v-card class="ma-6">
            <v-img
                :src="image.src"
                class="white--text align-end"
                gradient="to bottom, rgba(0,0,0,.1), rgba(0,0,0,.5)"
                height="300px"
            >
            </v-img>
            <v-card-actions>
              <v-card-text>{{ image.name| omittedText }}</v-card-text>
              <v-btn icon @click.stop="openDialog(image.name,index)">
                <v-icon>mdi-delete-empty</v-icon>
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
    </v-main>
  </v-app>
</template>

<script>
export default {
  name: 'App',

  components: {},

  data: () => ({
    dialog: false,
    images: [],
    selectedImage: {},
    fullscreen: false,
    background:"background-color:#FFF;",
    imageElement: {},
    padding: "",
    input: 5,
    transitionList:[{name:"フェード",effect:"fade-transition"},{name:"スライド",effect:undefined }],
    transition:{name:"フェード",effect:"fade-transition"}
  }),
  computed: {
    interval: function () {
      return this.input * 1000
    }
  },
  filters: {
    omittedText(text) {
      // 11文字目以降は"…"
      return text.length > 30 ? text.slice(0, 30) + "…" : text;
    },
  },
  mounted() {
    document.ondrop = (e) => {
      e.preventDefault()
      const files = e.dataTransfer.files
      for (let file of files) {
        const reader = new FileReader()
        reader.onload = (e) => {
          console.log(e.target.result)
          this.images.push({src: e.target.result, name: file.name})
        }
        reader.readAsDataURL(file)
      }
    }
    document.ondragover = function (e) {
      e.preventDefault()
      return false
    }
    this.imageElement = document.documentElement
    document.addEventListener('fullscreenchange', () => {
      if (document.fullscreenElement) {
        console.log("entered full-screen mode.");
      } else {
        console.log('Leaving full-screen mode.');
        this.background = "background-color:#FFF;"
        document.body.classList.remove("hide")
        this.fullscreen = false
        this.padding = ""
      }
    });
  },
  methods: {
    openDialog(filename, index) {
      this.selectedImage = {filename: filename, index: index}
      this.dialog = true
    },
    deleteImage(index) {
      this.dialog = false
      this.images.splice(index, 1)
      this.selectedImage = {}
    },
    incrimentInput() {
      if (this.input < 1000) {
        this.input = parseInt(this.input)+1
      }
    },
    decrimentInput() {
      if (this.input > 1) {
        this.input = parseInt(this.input)-1
      }
    },
    changeFullscreen() {
      if (this.images.length >= 1) {
        this.fullscreen = true
        this.imageElement.src = this.images[0].src
        this.imageElement.requestFullscreen(); // フルスクリーンを表示する
        this.background = "background-color:#000;"
        console.log(document.body.classList)
        document.body.classList.add("hide")
        this.padding = "pa-0"
      }
    }
  },
};
</script>

<style>
.hide::-webkit-scrollbar {
  display: none;
}
.hide {
  cursor: none;
}

.no-padding {
  padding: 32px 0px 0px;
}
</style>
