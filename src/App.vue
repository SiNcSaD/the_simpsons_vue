<template>
  <div>
    <div>
      <img src="/static/Logo.jpg" alt="">
    </div>
    <!-- <h1>The Simpsons Character Classification Application</h1> -->
    <div class="uk-container uk-container-large uk-text-center uk-margin">

      <div v-if="!image">
        <div class="uk-placeholder uk-text-center">
          <h2>Select an image</h2>
        </div>
        <div uk-form-custom>
          <input type="file" @change="onFileChange" multiple>
          <button class="uk-button uk-button-secondary" type="button" tabindex="-1">Upload</button>
        </div>
      </div>

      <div v-else class="uk-child-width-1-2@m uk-child-width-1-1@s uk-flex-middle uk-margin-remove" uk-grid>

        <div class="uk-padding-remove">
          <img id="img_src" :src="image">
        </div>

        <div v-show="isPredicted" class="uk-card uk-card-default uk-card-body">
          <!-- 年齡標籤，Male or Female -->
          <div v-if="character_gender === 'Male'">
            <div class="uk-card-badge uk-label">Male</div>
          </div>
          <div v-else>
            <div class="uk-card-badge uk-label uk-label-danger">Female</div>
          </div>
          <h2 class="character-Name">{{ character_name }}</h2>
          <h2 class="character-Eng-Name">{{ character_eng_name }}</h2>
          <h4 class="uk-text-left character-description">10歲的霸子是荷馬和美枝唯一的兒子，花枝和奶嘴的哥哥。霸子最引人注目的特點就是他淘氣、不服管教、不尊重權威的性格。</h4>
        </div>
      </div>
        <!-- <div class="uk-flex uk-flex-center">
          <button class="uk-button uk-button-secondary" @click="removeImage">Remove</button>
        </div> -->

    </div>

    <div class="uk-section uk-section-secondary bottom_info">
      <div class="uk-container uk-text-center">
        <h3>國立台北科技大學 - 電機工程系 - 高效能計算與深度學習研究室</h3>
        <h3>National Taipei University of Technology - Department of Electrical Engineering</h3>
      </div>
    </div>
  </div>
</template>

<script>
import * as tf from '@tensorflow/tfjs'
import Character from './character'
const MODEL_PATH = 'static/web_tfjs_model/model.json'

const Characters = [
  new Character('荷熊·辛普森', 'Grampa Simpson', 'Male', '11111111111'),
  new Character('阿普·納哈薩皮馬佩蒂隆', 'Apu Nahasapeemapetilon', 'Male', '11111111111'),
  new Character('霸子·辛普森', 'Bart Simpson', 'Male', '11111111111'),
  new Character('郭董', 'Charles Montgomery Burns', 'Male', '11111111111'),
  new Character('孔龍金', 'Chief Wiggum', 'Male', '11111111111'),
  new Character('宅神', 'Comic Book Guy', 'Male', '11111111111'),
  new Character('陳趾鹹', 'Edna Krabappel-Flanders', 'Female', '11111111111'),
  new Character('荷馬·傑伊·辛普森', 'Homer Simpson', 'Male', '11111111111'),
  new Character('李掏', 'Kent Brockman', 'Male', '11111111111'),
  new Character('小丑阿基', 'Krusty the Clown', 'Male', '11111111111'),
  new Character('藍尼·雷公', 'Lenny Leonard', 'Male', '11111111111'),
  new Character('莉莎·辛普森', 'Lisa Simpson', 'Female', '11111111111'),
  new Character('美枝·辛普森', 'Marge Simpson', 'Female', '11111111111'),
  new Character('郝融冰', 'Mayor Quimby', 'Male', '11111111111'),
  new Character('蘇呆子', 'Milhouse Van Houten', 'Male', '11111111111'),
  new Character('莫少蔥', 'Moe Szyslak', 'Male', '11111111111'),
  new Character('魯肉王', 'Ned Flanders', 'Male', '11111111111'),
  new Character('阿浮', 'Nelson Muntz', 'Male', '11111111111'),
  new Character('西蒙·喜金捏', 'Principal Skinner', 'Male', '11111111111'),
  new Character('包添丁', 'Sideshow Bob', 'Male', '11111111111')
]

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
      character_name: '',
      character_eng_name: '',
      percent: '',
      costTime: '',
      character_gender: 'Male',
      isPredicted: false
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
        var idx = predication.argMax(1).dataSync()[0]
        const max = predication.max().dataSync()[0]
        this.percent = max.toFixed(5) * 100
        this.character_name = Characters[idx].chtName
        this.character_eng_name = Characters[idx].engName
        this.costTime = (end - start) / 1000 + ' Sec'
        this.character_gender = Characters[idx].gender
        this.isPredicted = true
      })
    },
    removeImage () {
      this.image = ''
      this.character_name = ''
      this.costTime = ''
      this.percent = ''
      this.character_gender = ''
      this.isPredicted = false
    }
  }
}
</script>

<style>
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
.uk-placeholder {
  padding: 50px 30px;
  border: 3px dashed #e5e5e5;
}
.bottom_info h3 {
  font-family: SimSun, Microsoft JhengHei;
}
#img_src {
  padding: 0px 20px;
}
.character-Name {
  display:inline-block;
  font-weight: bold;
  font-family: Microsoft JhengHei, SimSun;
}
.character-Eng-Name {
  margin-left: 50px;
  display: inline-block;
  font-weight: bold;
  font-family: Arial, Helvetica, sans-serif;
}
.character-description {
  font-family: SimSun, Microsoft JhengHei;
}
</style>
