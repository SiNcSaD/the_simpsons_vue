<template>
  <div id="app">
    <h1>NTUTEE</h1>
    <!-- <img id="img_src" src="static/bart_simpson/pic_0059.jpg"> -->
    <!-- <img id="img_src" src="static/kent_brockman/pic_0001.jpg"> -->
    <img id="img_src" src="static/lisa_simpson/pic_0105.jpg">
  </div>
</template>

<script>
import * as tf from '@tensorflow/tfjs'

const MODEL_PATH = 'static/web_tfjs_model/model.json'

const loadModel = async () => {
  console.log('loading model...')
  var model = await tf.loadModel(MODEL_PATH)
  console.log('Warmup the model')
  model.predict(tf.zeros([1, 64, 64, 3])).dispose()
  return model
}

export default {
  name: 'App',
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
      predication.argMax(1).print()

      console.log('Complete!')
    })
  }
}
</script>

<style>
</style>
