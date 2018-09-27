<template>
  <div class="bg">
    <div>
       <img src="/static/Logo.jpg" alt="LOGO">
    </div>
    <div class="uk-container uk-container-expand">
      <!-- Upload Picture -->
      <div class="uk-child-width-expand@s uk-text-center uk-padding" uk-grid>
        <div>
            <div v-if="!image">
              <div class="uk-placeholder uk-position-relative">
                <h2 class="uk-position-center">Select an image</h2>
              </div>
              <div uk-form-custom>
                <input type="file" @change="onFileChange" multiple>
                <button class="uk-button uk-button-secondary" type="button" tabindex="-1">Upload</button>
              </div>
            </div>
            <div v-else>
              <div class="uk-placeholder uk-position-relative">
                <img class="uk-position-center" id="img_src" :src="image" alt="uploadImage" style="max-height: 300px;">
              </div>
              <div class="uk-flex uk-flex-center">
                <button class="uk-button uk-button-secondary" @click="removeImage">Remove</button>
              </div>
            </div>
        </div>
        <!-- Show Predict Result -->
        <div>
          <div v-if="!character">
            <div class="uk-child-width-1-1 uk-text-center" uk-grid>
              <div class="uk-child-width-1-2 uk-padding-small" uk-grid>
                <div>
                  <img src="/static/who.png" width="80" alt="">
                </div>
                <div>
                  <h3>Character</h3>
                </div>
              </div>
              <div class="uk-child-width-1-2  uk-padding-small" uk-grid>
                <div>
                  <img src="/static/percent.png" width="80" alt="">
                </div>
                <div>
                  <h3>Percent</h3>
                </div>
              </div>
              <div class="uk-child-width-1-2  uk-padding-small" uk-grid>
                <div>
                  <img src="/static/time1.png" width="80" alt="">                  
                </div>
                <div>
                  <h3>%</h3>
                </div>
              </div>
            </div>
          </div>
          <div class="uk-padding" v-else>
            <div class="uk-child-width-1-1 uk-text-center uk-flex-center" uk-grid>
              <div class="uk-child-width-1-2 uk-padding-small" uk-grid>
                <div>
                  <img src="/static/who.png" width="80" alt="">
                </div>
                <div>
                  <h3>{{character}}</h3>
                </div>
              </div>
              <div class="uk-child-width-1-2  uk-padding-small" uk-grid>
                <div>
                  <img src="/static/percent.png" width="80" alt="">
                </div>
                <div>
                  <h3>{{percent}}%</h3>
                </div>
              </div>
              <div class="uk-child-width-1-2  uk-padding-small" uk-grid>
                <div>
                  <img src="/static/time1.png" width="80" alt="">                  
                </div>
                <div>
                  <h3>{{costTime}}</h3>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <!-- Introduction -->
      <div class="uk-text-center">
        <div class="bar uk-button uk-button-secondary uk-width-1-1 uk-light uk-text-center uk-position-relative">
          <h3 class="uk-position-center">Introduction</h3>
        </div>
        <!-- <div class="rect"></div> -->
      </div>
      <div>
        <div class="bar uk-button uk-button-secondary uk-width-1-1 uk-light uk-text-center uk-margin-top uk-position-relative">
          <h3 class="uk-position-center">Visualize CNN</h3>
        </div>
        <!-- <div class="rect"></div> -->
      </div>
    </div>
  </div>
</template>

<script>
import * as tf from '@tensorflow/tfjs'

const MODEL_PATH = 'static/web_tfjs_model/model.json'
const Character = ['荷熊·辛普森', '阿普·納哈薩皮馬佩蒂隆', '霸子·辛普森', '郭董', '維古姆警官', '宅神', '埃德娜·克拉巴佩爾',
  '荷馬·辛普森', '李掏', '小丑庫斯提', '倫尼·倫納德', '莉莎·辛普森', '美枝·辛普森', '郝融冰', '米爾豪斯·范霍滕',
  '莫少蔥', '魯肉王', '阿浮', '西蒙·喜金捏', '包添丁']

const loadModel = async () => {
  console.log('loading model...')
  var model = await tf.loadModel(MODEL_PATH)
  console.log('Warmup the model')
  model.predict(tf.zeros([1, 64, 64, 3])).dispose()
  return model
}

export default {
  name: 'App',
  data () {
    return {
      image: '',
      character: '',
      percent: '',
      costTime: ''
    }
  },
  methods: {
    onFileChange (e) {
      var files = e.target.files || e.dataTransfer.files
      if (!files.length) return
      this.createImage(files[0])
    },
    createImage (file) {
      var reader = new FileReader()
      var vm = this
      reader.onload = e => {
        vm.image = e.target.result
      }
      reader.readAsDataURL(file)

      /*
        Classification Simpson
      */
      var start = 0
      var end = 0
      start = new Date().getTime()
      loadModel().then(model => {
        console.log('Predicting...')

        /**
         * Get element of html <img>
         */
        var imgSrcElement = document.getElementById('img_src')
        // var H = imgSrcElement.height
        // var W = imgSrcElement.width

        /**
         * Loading image and normalization
         */
        var img = tf.fromPixels(imgSrcElement).toFloat()
        var imgNorm = img.div(255.0)
        const predication = tf.tidy(() => {
          /**
          * Image pre-processing and prediction
          */
          var imgResize = tf.image.resizeBilinear(imgNorm, [64, 64])
          return model.predict(imgResize.reshape([1, 64, 64, 3]))
        })
        end = new Date().getTime()
        predication.print()
        var idx = predication.argMax(1).dataSync()[0]
        const max = predication.max().dataSync()[0]
        this.percent = max.toFixed(5) * 100
        this.character = Character[idx]
        this.costTime = (end - start) / 1000 + ' Sec'
      })
    },
    removeImage () {
      this.image = ''
      this.character = ''
      this.costTime = ''
      this.percent = ''
    }
  }
}
</script>

<style>
html, body{
  height: 100%;
}
/* .bg {
  background: url("/static/730131.jpg");
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
  height: 100%;
} */
.move img{
  animation: floating 2s ease infinite;
}
@keyframes floating {
  0%{
    transform: rotate(-30deg) translateY(2px);
  }
  50%{
    transform: rotate(-30deg) translateY(15px);
  }
  100%{
    transform: rotate(-30deg) translateY(2px);
  }
}
.uk-placeholder{
  height: 300px;
  text-align: center;
}
h3{
  margin: 0;
}
.bar{
  height: 50px;
}
/* .rect{
  height: 0;
  width: 0;
  border-color: #222 transparent transparent transparent;
  border-style: solid;
  border-width: 20px;
} */
</style>
