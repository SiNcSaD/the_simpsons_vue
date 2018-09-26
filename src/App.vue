<template>
  <div class="simpson"> 
    <!-- <div>
       <img src="/static/Logo.jpg" alt="LOGO">
    </div> -->
    <div class="uk-container uk-container-expand">
      <!-- Upload Picture -->
      <div class="uk-child-width-expand@s uk-text-center" uk-grid>
        <div>
          <div class="uk-padding">
            <div v-if="!image">
              <div class="uk-placeholder uk-position-relative">
                <h2 class="uk-position-center">Select an image</h2>
              </div>
              <div uk-form-custom>
                <input type="file" @change="onFileChange" multiple>
                <button class="uk-button uk-button-secondary" type="button" tabindex="-1">Upload</button>
              </div>
              <progress id="js-progressbar" class="uk-progress" value="0" max="100" hidden></progress>
            </div>
            <div v-else>
              <div class="uk-placeholder">
                <img id="img_src" :src="image" alt="uploadImage" style="height: 300px;">
              </div>
              <div class="uk-flex uk-flex-center">
                <button class="uk-button uk-button-secondary" @click="created">Predict</button>
                <div class="uk-padding-small uk-padding-remove-vertical"></div>
                <button class="uk-button uk-button-secondary" @click="removeImage">Remove</button>
              </div>
            </div>
          </div>
        </div>
        <!-- Show execute time -->
        <div class="uk-padding uk-position-relative">
          <div class="move uk-position-center">
            <img src="/static/time.png" width="80" alt="">
            <h2>Execute time</h2>
            <h2>2.71S</h2>
          </div>
        </div>
        <!-- Show Predict Result -->
        <div class="uk-padding">
          <div v-if="!character">
            <table class="uk-table uk-table-hover uk-table-divider">
                <thead>
                    <tr>
                        <th>Character</th>
                        <th>Percent</th>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <td>Simpson</td>
                        <td>100%</td>
                    </tr>
                </tbody>
            </table>
          </div>
          <div v-else>
            <table class="uk-table uk-table-hover uk-table-divider">
                <thead>
                    <tr>
                        <th>Character</th>
                        <th>Percent</th>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <td>{{character}}</td>
                        <td>100%</td>
                    </tr>
                </tbody>
            </table>
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
      percent: ''
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
    },
    removeImage () {
      this.image = ''
      this.character = ''
    },
    created () {
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
        predication.print()
        var idx = predication.argMax(1).dataSync()[0]
        var max = predication.argMax(1)

        console.log(Character[idx])
        console.log(max)
        this.character = Character[idx]

        console.log('Complete!')
      })
    }
  }
}
</script>

<style>
.simpson{
  background-image: url("/static/730131.jpg");
}
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
