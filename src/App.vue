<template>
  <header>
    <p>测测你的身价是多少？</p>
  </header>
  <section class="questions" v-if="currentQuestion">
    <Question 
      :question="currentQuestion" 
      :index="currentQuestionIndex"
      @select="selectOption"
    />
    <div class="action text-center" 
      :class="{ active: enableNextQuestion }"
      @click="nextQuestion"
    >
      <i class="icono-checkCircle"></i>
    </div>
  </section>
  <section class="share text-center" v-else>
    <button>
      <i class="icono-share"></i>
    </button>
    <p>这个程序员需要 ￥{{ totalPrice }} 才能招的起！</p>
    <p v-if="inWechat" style="color: red;">点击右上角分享到微信</p>
    <p v-else>
      <a :href="weiboShareUrl" target="_blank">分享到微博</a>
    </p>
    <p class="promo">
      下载
      <a href="https://gold.xitu.io/app">掘金</a>
      应用，挖掘更多的技术干货
    </p>
  </section>
  <footer>
    <p>￥{{ showPrice }}</p>
  </footer>
</template>


<script>
import { ref, computed } from 'vue'
import Question from './components/Question.vue'
import questionsData from './questions.json'
import { onMounted } from 'vue'

export default {
  components: {
    Question
  },
  setup() {
    const currentQuestionIndex = ref(0)
    const totalPrice = ref(0)
    const showPrice = ref(0)
    const questions = ref(questionsData)

    onMounted(() => {
      document.title = '测一测你的身价是多少？',
      console.log('Initial state:', {
        currentQuestionIndex: currentQuestionIndex.value,
        currentQuestion: currentQuestion.value,
        questions: questions.value
      })
    })

    const currentQuestion = computed(() => 
      currentQuestionIndex.value < questions.value.length ? questions.value[currentQuestionIndex.value] : null
    )

    const enableNextQuestion = computed(() => 
      currentQuestion.value && currentQuestion.value.selected !== null
    )

    const inWechat = computed(() => 
      window.navigator.userAgent.toLowerCase().indexOf('micromessenger') !== -1
    )

    const weiboShareUrl = computed(() => {
      const appUrl = 'http://xitu.github.io/coder-price/'
      const desc = `${appUrl}：作为一个程序猿我的身价是￥${totalPrice.value}，来测测你的身价是多少！@稀土圈`
      const url = 'https://xitu.io'
      return `http://service.weibo.com/share/share.php?title=${encodeURIComponent(desc)}&url=${encodeURIComponent(url)}`
    })

    const selectOption = (index) => {
      console.log('selectOption called with index:', index)
      if (currentQuestion.value) {
        currentQuestion.value.selected = index
        console.log('Current question updated:', currentQuestion.value)
      }
    }

    const nextQuestion = () => {
      if (enableNextQuestion.value) {
        currentQuestionIndex.value += 1
        const newPrice = questions.value.reduce((sum, q) => 
          sum + (q.selected !== null ? q.options[q.selected].price : 0), 0
        )
        
        // 优化金额变化速度和平滑度
        const duration = 500 // 总持续时间（毫秒）
        const steps = 20 // 变化的步数
        const stepDuration = duration / steps
        const startPrice = showPrice.value
        const priceChange = newPrice - startPrice

        let step = 0
        const interval = setInterval(() => {
          step++
          if (step <= steps) {
            showPrice.value = Math.round(startPrice + (priceChange * step / steps))
          } else {
            clearInterval(interval)
            showPrice.value = newPrice // 确保最终显示精确的新价格
          }
        }, stepDuration)

        totalPrice.value = newPrice

        if (currentQuestionIndex.value === questions.value.length) {
          document.title = `作为一个程序猿我的身价是￥${totalPrice.value}，来测测你的身价是多少！`
        }
      }
    }


    return {
      currentQuestionIndex,
      currentQuestion,
      totalPrice,
      showPrice,
      enableNextQuestion,
      inWechat,
      weiboShareUrl,
      selectOption,
      nextQuestion
    }
  }
}
</script>

<style lang="stylus">
*, *:before, *:after
    -webkit-box-sizing: border-box;
    -moz-box-sizing: border-box;
    box-sizing: border-box;
    outline: none;
body, html
    margin: 0;
    border: 0;
    padding: 0;
    height: 100%;
    max-height: 100%;
    position: relative;
body
    font-family: "Hiragino Sans GB", Helvetica, Arial, sans-serif;
header, footer
    position: absolute;
    z-index: 10;
    background: rgb(19 19 56);
    color: #fff;
    left: 0;
    right: 0;
    text-align: center;
header
    top: 0;
footer
    bottom: 0;
.share, .questions
    position: absolute;
    top: 60px;
    right: 0;
    bottom: 60px;
    left: 0;
.questions
    position: absolute;
    top: 60px;
    right: 0;
    bottom: 60px;
    left: 0;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
.questions .question
    width: 100%;
    max-width: 400px;
    margin: 0 auto;
    padding: 20px;
.questions .action
    position: absolute;
    bottom: 10px;
    left: 0;
    right: 0;
.questions .action.active
    cursor: pointer;
.questions .action [class*=icono-]
    transition: all 0.3s;
.questions .action.active [class*=icono-]
    color: rgb(19 19 56);
.share
    padding-top: 30px;
.share .promo
    position: absolute;
    bottom: 10px;
    left: 0;
    right: 0;
button
    -webkit-appearance: none;
    appearance: none;
    padding: 10px;
    border: 2px solid red;
    border-radius: 50%;
    background: none;
    color: rgb(19 19 56);
button [class*=icono-]
    color: rgb(19 19 56);
.share button
    margin: 20px;

// UTILS
.text-center
    text-align: center;
.pointer
    cursor: pointer;
</style>
