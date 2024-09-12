<template>
  <header>
    <p>这个程序员啥时候才能财务自由?</p>
  </header>
  <section class="questions" v-if="currentQuestion">
    <Question 
      :question="currentQuestion" 
      :index="currentQuestionIndex"
      @select="selectOption"
    />
  </section>
  <section class="share text-center" v-else>
    <button>
      <i class="icono-share"></i>
    </button>
    <p>这个程序员 <b>{{ totalPrice }}</b> 岁才能财务自由！</p>
    <div class="age-comment" v-html="ageCommentHtml"></div>
    <p v-if="inWechat" style="color: red;">点击右上角分享到微信</p>
    <p v-else>
      <a :href="weiboShareUrl" target="_blank">分享到微博</a>
    </p>
    <p class="promo">
      10x 程序员都在用
      <a href="https://www.marscode.cn/">豆包MarsCode</a>
      刷面试算法题
    </p>
  </section>
  <footer>
    <p>可能是<b>{{ showPrice }}岁？</b></p>
  </footer>
</template>


<script>
import { ref, computed } from 'vue'
import MarkdownIt from 'markdown-it'
import Question from './components/Question.vue'
import questionsData from './questions.json'
import { onMounted } from 'vue'

const md = new MarkdownIt()

export default {
  components: {
    Question
  },
  setup() {
    const currentQuestionIndex = ref(0)
    const totalPrice = ref(0)
    const getRandomInt = (min, max) => {
      return Math.floor(Math.random() * (max - min + 1)) + min;
    }
    const showPrice = ref(getRandomInt(20, 40))
    const questions = ref(questionsData)

    onMounted(() => {
      document.title = '这个程序员啥时候才能财务自由?',
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
      const desc = `${appUrl}：作为一个程序猿，我会在${totalPrice.value}财务自由，来测测你的是什么时候！`
      const url = 'https://www.marscode.cn/'
      return `http://service.weibo.com/share/share.php?title=${encodeURIComponent(desc)}&url=${encodeURIComponent(url)}`
    })

    const selectOption = (index) => {
      console.log('selectOption called with index:', index)
      if (currentQuestion.value) {
        currentQuestion.value.selected = index
        console.log('Current question updated:', currentQuestion.value)
      }

      // 立即进入下一题
      nextQuestion()
    }

    const nextQuestion = () => {
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
        document.title = `作为一个程序猿，我会在${totalPrice.value}岁财务自由，来测测你的！`
      }
    }

    const getAgeComment = (age) => {
      if (age >= 0 && age <= 18) return `
让我解读来你的超能力：

- 时间管理大师：Elon Musk 还在用 Basic 写 Blastar 游戏，你已经在纳斯达克敲钟了。是不是偷偷把时间复杂度优化到了O(1)？
- 投资界的巴菲特：你的投资眼光堪比 AI 量化算法，连比特币都对你俯首称臣。华尔街应该给你颁发一个"最佳神童"奖
- Work Life Balance 大师：周末加班？不存在的！你的人生信条是"Work smart, not hard`
      if (age >= 19 && age <= 28) return `
你是怎样的奇才？

- 超速成长：你的代码能力恐怕比编译器编译的速度还快！扎克伯格还在学习用 PHP 写"Hello World"的时候，你已经在构建下一个独角兽公司了。
- 时间管理大师：你一定掌握了什么神秘的时间管理术。难道你每天都在使用小叮当的时间静止器，把24小时变成240小时？
- 投资界的巴菲特：你的投资眼光非常独到，比特币个位数的时候就早早买入。华尔街应该给你颁发一个"最佳预言家"奖杯。`
      if (age >= 29 && age <= 38) return `
让我来解读一下你的"成功密码"：

- **编程语言**：C++？不怕头发掉光的勇士！财务自由后，假发随便买。
- **Bug处理**：先Google？聪明！Stack Overflow 大神永远滴神，除非你问了个"蠢问题"。
- **加班态度**：周末加班？用时间换钱，再用钱买时间，这操作，秀啊！
- **投资策略**：A股？代码能力强，赌性更是杠杠的！愿你的投资曲线如学习曲线般陡峭（向上）。`
      if (age >= 39 && age <= 48) return `
你用 C++ 和 Java 编织梦想，用 Tab 键缩进人生。每一个 bug 都是通向财务自由的垫脚石，每一次加班都是为自由储蓄的小金库。看来“早起的鸟儿有虫吃”在你这里变成了“熬夜的程序员赚更多”！

PS: 如果你不小心把这条分享发到了工作群，别担心，就说你在测试新的大模型。毕竟，作为一个即将财务自由的程序员，你怎么可能会犯这种低级错误呢？😉`
      if (age >= 49 && age <= 58) return `
你的人生就像调试代码：充满挑战，但值得。在这条路上，你可能会遇到一些小 bug，比如忘买礼物导致的“家庭系统崩溃”，或熬夜 Coding 导致的“身体运行错误”。别担心，这都是升级到"人生2.0"的必经之路！

送你一段伪代码：

  while age < financial_freedom_age:
      code_hard()
      save_money()
      avoid_unnecessary_expenses()
      if random.choice([True, False]):
          learn_new_skills()
      sleep(6)  # 谁说程序员就一定要熬夜呢？

  print("恭喜你实现财务自由！现在你可以真正地'Hello World'了！")
`
      if (age >= 59 && age <= 68) return `
哇哦，看来你是打算在退休前的最后一刻才实现财务自由啊！不愧是程序员界的"慢热型选手"，用大半辈子的时间来酝酿这一刻的辉煌。你的财务自由之路，就像是写了一辈子的 if-else，终于在生命的尾声迎来了 return true 的那一刻！

PS: 如果你真的在这个年龄段实现财务自由了，别忘了请我吃顿大餐！我们可以一起讨论如何用大模型预测股市，或者研究一下养老院里的 Wi-Fi 信号怎么才能覆盖到每个躺椅！毕竟，咱们可是要做最酷的退休程序员！
`
      if (age >= 69 && age <= 78) return `
哇哦，看来你是打算在退休前的最后一刻还在疯狂敲代码啊！不愧是行走的编程百科全书，连财务自由都要在古稀之年才能实现。

想象一下，当你终于财务自由那天：

- 你的 IDE 里积累了几十年的 TODO 注释
- 你可能已经见证了 Python 4.0 的发布
`
      if (age >= 79 && age <= 88) return `
恭喜你，你已经踏上了通往财务自由的光明大道！

* **好消息**：你终于可以告别996，迎接997（99岁退休，7天不间断coding）的美好生活了！
* **坏消息**：在你实现财务自由的那一天，你可能需要先确认一下你的老花镜度数是否够用。      
`
      if (age >= 89 && age <= 98) return `
你的编程生涯堪称一部史诗级的"慢动作"电影。当你终于达到财务自由的那一刻，想必比编译一个巨型遗留 C++ 项目还要令人激动！

以下是你的独家成就解锁：

- "代码与退休赛跑"奖杯
- "活到老，码到老"勋章
- "我的代码比我的皱纹还要深奥"称号      
`
      return `
别担心，等你财务自由的时候，键盘说不定都已经进化成脑机接口了。不过话说回来，你这么执着于代码，该不会是想在99岁的时候创造出"长生不老.exe"吧？

你的编程生涯亮点：

- 代码注释可能比《战争与和平》还要长
- Git 提交历史堪比地质年代表
- 可能是唯一一个见证过 "JavaScript 统治世界" 预言从诞生到实现的人
`
    }

    const ageCommentHtml = computed(() => md.render(getAgeComment(totalPrice.value)))

    return {
      currentQuestionIndex,
      currentQuestion,
      totalPrice,
      showPrice,
      enableNextQuestion,
      inWechat,
      weiboShareUrl,
      selectOption,
      nextQuestion,
      ageCommentHtml
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
    border: 2px solid rgb(19 19 56);
    border-radius: 50%;
    background: none;
    color: rgb(19 19 56);
button [class*=icono-]
    color: rgb(19 19 56);
.share button
    margin: 20px;

.age-comment
  text-align: center
  padding: 10px
  background-color: #f9f9f9
  border-radius: 5px
  margin-top: 20px
  
  h1
    font-size: 1.2em
    color: #333
    margin-bottom: 10px
  
  ul
    padding-left: 20px
    
    li
      margin-bottom: 5px
      color: #666

// UTILS
.text-center
    text-align: center;
.pointer
    cursor: pointer;  
</style>
