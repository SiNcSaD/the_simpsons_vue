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
          <div v-if="predict_character.gender === 'Male'">
            <div class="uk-card-badge uk-label">Male</div>
          </div>
          <div v-else>
            <div class="uk-card-badge uk-label uk-label-danger">Female</div>
          </div>
          <h2 class="character-Name">{{ predict_character.chtName }}</h2>
          <h2 class="character-Eng-Name">{{ predict_character.engName }}</h2>
          <h4 class="uk-text-left character-description">{{ predict_character.description }}</h4>
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
  new Character('荷熊·辛普森', 'Grampa Simpson', 'Male', '荷熊過去曾經是一個空軍飛官，退役後到酒廳端盤子並意外發掘出他的作曲天分和人生第三位妻子，並在人生的某個階段中以魔鬼筋肉人的身分稱霸擂台，是郭董的偶像。 個性慈祥，年輕時很照顧單親的荷馬，但老了之後有嚴重的癡呆症。絕大多數的時間他住在內糊療養院。荷熊在神智清醒的時候常常大罵荷馬是個不孝子，但是某些集數裡荷馬還是會展現對荷熊的愛。'),
  new Character('阿普·納哈薩皮馬佩蒂隆', 'Apu Nahasapeemapetilon', 'Male', '印度非法移民（後通過考試，轉為正式），信奉印度教，喜歡把信奉的神明雕塑擺在店和家裡，素食主義者。在印度加爾各答某大學畢業後，來斯普林菲爾德讀研究生，並取得計算機專業的博士學位，後來加盟了一家Kwilk-E-Mart便利店，經常被搶劫，在印度老家有個年僅10歲的包辦婚姻的妻子，後來又娶了另一個包辦婚姻的妻子Manjula，生了八胞胎。總體來說還是不錯的一個人，但經常篡改食品的保質期。'),
  new Character('霸子·辛普森', 'Bart Simpson', 'Male', '10歲的霸子是荷馬和美枝唯一的兒子，莉莎和奶嘴的哥哥。霸子最引人注目的特點就是他淘氣、不服管教、不尊重權威的性格。莉莎剛出生時，霸子妒嫉她吸引到的注意力，但當他發現她說出第一個字是「霸子」時，他就喜歡上她了。 霸子第一次上學時，他最初學習的熱情就在他發現老師根本不在意他時消散了。一天午休時，霸子遇到蘇呆子，並開始用搞笑的姿勢和粗魯的字眼來娛樂呆子和其他孩子。喜金捏校長警告他：「你剛剛開始上學，而你選擇的發展道路會影響你一生！你想說什麼？」就在這關鍵的時刻，霸子回答：「Eat my shorts！」。'),
  new Character('郭董', 'Charles Montgomery Burns', 'Male', '郭董是個完全的壞蛋，幾乎沒有任何可取之處。「貪婪」，作為老闆，他壓榨員工，比如取消了所有員工的聖誕節獎金。「無自尊」，郭董對比自己有錢的人採取巴結逢迎的態度。「冷酷」，史密瑟斯是他的副手、顧問、知己與暗慕者，對他既忠誠又諂媚。在核電站即將融毀的關鍵時刻，郭董霸占了史密瑟斯的防護服。無自知之明－內糊沒人認為郭董相貌好看（史密瑟斯除外）。郭董卻說自己「有錢、英俊」。「恃強凌弱」，史密瑟斯勸郭董不要和內糊小學較勁。郭董表示和小學較勁就像是從嬰兒的嘴裡奪取糖果。接著他看到了遠處一個拿糖果的嬰兒，就要立刻去搶。'),
  new Character('孔龍金', 'Chief Wiggum', 'Male', '春田鎮警察局局長。他是拉爾夫·維古姆的父親和薩拉·維古姆的丈夫。喜劇看點為無能、不負責、懶惰和暴食的一名警察。'),
  new Character('宅神', 'Comic Book Guy', 'Male', '宅神的真名是朱斜橫。他經營的漫畫店常常賣一些質量很差、價格很高的商品。他是個狂熱的漫畫收集者。他的口頭禪是：「Worst episode ever!」在辛普森家庭大電影中他以為他快死時，他說：「我這輩子除了收集漫畫以外什麼也沒做，我現在想說的是：『這輩子活的有價值！』」。'),
  new Character('陳趾鹹', 'Edna Krabappel-Flanders', 'Female', '她是霸子·辛普森內糊小學四年級教師和魯肉王的現任妻子。2013年10月，美國配音員逝世後，節目製作人宣布該角色於節目引退。'),
  new Character('荷馬·傑伊·辛普森', 'Homer Simpson', 'Male', '荷馬出生於1956年5月12日，在上內糊高中的時候，荷馬與美枝相識並陷入愛河。1981年，美枝懷上了霸子，當時荷馬在一處小高爾夫球場工作。兩人在橫跨州界的一所小教堂中舉行了婚禮。在1985年和1986年，荷馬在「Be Sharps」理髮店四重唱中擔當主唱與歌曲作者，甚至還得過一次艾美獎。荷馬是一個時常犯蠢、懶惰與暴怒的人。儘管過著藍領般的日子，他常常展現出低智商的行為荷馬卻有著許許多多不平凡的事跡。雖然他貪食、懶惰、常惹事故且非常愚蠢，但卻偶爾能展現出自身的才智與真實價值，譬如對自己家人的熱愛及保護。'),
  new Character('李掏', 'Kent Brockman', 'Male', '在劇集「Krusty Gets Busted」中首次出現。他是一個脾氣暴躁，以自我中心的新聞主播。'),
  new Character('小丑阿基', 'Krusty the Clown', 'Male', '人稱希斯‧萊傑。他是霸子和花枝最喜歡的電視節目的主持人，還是「醜八怪漢堡」連鎖店的老闆，分店除了遍布整個內糊地區外，甚至在中國也有分店，是霸子甚至是荷馬和鎮上的居民很常光顧的店，但店內很常爆出衛生問題。阿基的父親是一名猶太人。他反對阿基成為小丑，導致父子長期不相往來。阿基的私生活亂七八糟。他在海灣戰爭期間結識了一名美軍女兵，兩人育有一女，蘇菲。'),
  new Character('藍尼·雷公', 'Lenny Leonard', 'Male', '內糊核電站工人。是莫那·盧到最好的朋友，也是荷馬·辛普森的朋友。他獲得了物理學碩士學位，卻經常以藍領的形象出現。儘管經常出現在內糊第一教堂中，他卻是一名佛教徒。'),
  new Character('莉莎·辛普森', 'Lisa Simpson', 'Female', '8歲的莉莎·辛普森是荷馬和美枝的第二個孩子。IQ高達156，會吹奏上低音薩克斯，是內糊小學二年級的小學生。一位素食主義者並皈依為佛教徒並投身於各種政治活動中，包括西藏獨立運動。'),
  new Character('美枝·辛普森', 'Marge Simpson', 'Female', '荷馬·辛普森的妻子，霸子、莉莎和奶嘴的母親，是一位善良且極具耐心的角色。她最引人注目的特點便是她以一種罕見的高度佇立的藍色頭髮。'),
  new Character('郝融冰', 'Mayor Quimby', 'Male', '民主黨一員，也是內糊市市長。有許多私生子。'),
  new Character('蘇呆子', 'Milhouse Van Houten', 'Male', '霸子的最好的朋友。對莉莎一見鍾情，卻被莉莎堅決拒絕。個性深深受到母親的不穩重與父親的懦弱影響。'),
  new Character('莫少蔥', 'Moe Szyslak', 'Male', '人稱老莫，是內糊的一間酒吧「老莫酒吧」的經營者及服務生。陰險、自私、好色、說謊和喜歡背叛。他為了錢讓荷馬和職業拳擊手同台競技，全不顧荷馬的死活。他的酒吧衛生條件極差，甚至出過人命。而他寧願通過賄賂來讓酒吧得過且過，也不肯花力氣整頓。老莫偶爾也會表現出一些優點，如老莫給受到流氓圍攻的荷馬解圍。對奶嘴·辛普森表現出了關懷和愛護。'),
  new Character('魯肉王', 'Ned Flanders', 'Male', '辛普森家的鄰居，時常被荷馬所憎惡。魯肉是一名虔誠的基督徒，他是內糊居民中最友善且最富同情心的人，他通常亦被看作是內糊社會的支柱。在政治上，魯肉王想聯合天主教徒戰勝他們共同的敵人——共產黨和大減價。'),
  new Character('阿浮', 'Nelson Muntz', 'Male', '霸子的同班同學，也是內糊小學裡的壞孩子之一。他的母親是妓女，父親去向不明，還有一個姐姐，但是可能死了。口頭禪是「Haw-haw!」，用以表示對某人的嘲笑。在欺負別人時，他經常說「Stop beating yourself!」'),
  new Character('西蒙·喜金捏', 'Principal Skinner', 'Male', '內糊小學校長。經常要奮力維持這破碎的學校，為經費不充足學校而爭取更多的教學資源，還要時常與他的「敵人」——霸子·辛普森作對。他參加過美國陸軍，還在越南戰爭中被越共俘虜，因此度過了18個月的戰俘生活。已經成年的他仍與自己的母親艾格尼絲·喜金捏一起生活，且對其母親言聽計從。'),
  new Character('包添丁', 'Sideshow Bob', 'Male', '一位邪惡天才，常精心設計復仇計畫卻被霸子制止。經過多年來不斷的失敗，包添丁步向精神失常，變成一位基因科學家並修改了自身多處基因，但因愧疚感跳水自殺，卻因為有鰓而沒有死。當完成24年來的心願成功殺死霸子後，卻因為懷念追殺霸子的生活，而不斷復活霸子好將他不斷殺死。')
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
      percent: '',
      costTime: '',
      isPredicted: false,
      predict_character: new Character('', '', '', '')
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
        this.costTime = (end - start) / 1000 + ' Sec'
        this.predict_character = new Character(Characters[idx].chtName, Characters[idx].engName, Characters[idx].gender, Characters[idx].description)
        this.isPredicted = true
      })
    },
    removeImage () {
      this.image = ''
      this.costTime = ''
      this.percent = ''
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
