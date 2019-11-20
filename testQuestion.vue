<template>
  <div class="question">
    <div class="startMain" v-if="stratShow">
      <div class="start_button" @click="startTest"></div>
    </div>
    <!--  真正显示数据 -->
    <div
      class="question__list animated"
      v-if="questionListShow"
      ref="nextQuestion"
    >
      <div class="number">{{ nextData.number }}</div>
      <div class="title">
        <div class="title__content">
          <div class="inline" :class="{ large: nextData.titleType === '1' }">
            {{ nextData.title }}
          </div>
        </div>
      </div>
      <div class="options">
        <div
          class="options__item"
          v-for="(item, index) in nextData.options"
          :key="index"
        >
          <div
            class="content"
            :class="{
              active:
                nextData.answer === selectOption && nextData.answer === item.id
            }"
            @click="optionClick(item)"
          >
            <div
              class="inline"
              :class="{
                small: nextData.optionType == '2',
                mini: nextData.optionType == '3'
              }"
            >
              {{ item.name }}
            </div>
          </div>
        </div>
      </div>
    </div>
    <!--  模拟向下掉的效果 -->
    <div
      class="question__list animated"
      v-if="questionListShow"
      ref="currentQuestion"
    >
      <div class="number">{{ currentData.number }}</div>
      <div class="title">
        <div class="title__content">
          <div class="inline" :class="{ large: currentData.titleType === '1' }">
            {{ currentData.title }}
          </div>
        </div>
      </div>
      <div class="options">
        <div
          class="options__item"
          v-for="(item, index) in currentData.options"
          :key="index"
        >
          <div
            class="content"
            :class="{
              active:
                currentData.answer === selectOption &&
                currentData.answer === item.id
            }"
            @click="optionClick(item)"
          >
            <div
              class="inline"
              :class="{
                small: currentData.optionType == '2',
                mini: currentData.optionType == '3'
              }"
            >
              {{ item.name }}
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="posters" v-if="postersShow">
      <div
        class="posters__main"
        :class="
          postersData && postersData.level == '1'
            ? 'levelOne'
            : postersData && postersData.level == '2'
            ? 'levelTwo'
            : 'levelThree'
        "
      >
        <div class="posters__content">
          <div class="top">
            <div class="eName">{{ postersData.eName }}</div>
            <div class="cName">{{ postersData.cName }}</div>
          </div>
          <div class="center">{{ postersData.describe }}</div>
        </div>
        <div class="describe__image"></div>
        <div id="QRCodeNone" class="qrcode" style="display:none;"></div>
        <div id="qrcode" ref="qrcode" class="qrcode"></div>
        <div class="qrcodeTitle">长按识别二维码</div>
      </div>
      <div
        class="posters__main share"
        ref="imageDom"
        :class="
          postersData && postersData.level == '1'
            ? 'levelOne'
            : postersData && postersData.level == '2'
            ? 'levelTwo'
            : 'levelThree'
        "
      >
        <div class="posters__content">
          <div class="top">
            <div class="eName">{{ postersData.eName }}</div>
            <div class="cName">{{ postersData.cName }}</div>
          </div>
          <div class="center">{{ postersData.describe }}</div>
        </div>
        <div class="describe__share"></div>
        <div id="QRCodeNoneShare" class="qrcode" style="display:none;"></div>
        <div id="qrcodeShare" ref="qrcodeShare" class="qrcode"></div>
        <div class="qrcodeTitle">长按识别二维码</div>
      </div>
      <div class="share_btn" @click="saveAndShareClick"></div>
    </div>
    <van-popup
      class="wrong_popup"
      v-model="wrongShow"
      :close-on-click-overlay="false"
    >
      <errorDialog
        v-if="wrongShow"
        :data="wrongData"
        @close="wrongClose"
      ></errorDialog>
    </van-popup>
    <van-popup
      class="popup__custom"
      v-model="shareImgShow"
      :close-on-click-overlay="false"
    >
      <div class="popupShare__main">
        <div class="share-title">长按图片保存到本地</div>
        <van-icon class="share-close" name="close" @click="closeShare" />
        <img
          id="share-img"
          class="share-img"
          :src="imgUrl"
          alt=""
          @touchstart="gotouchstart"
          @touchmove="gotouchmove"
          @touchend="gotouchend"
        />
      </div>
    </van-popup>
  </div>
</template>
<script>
import {
  getQueryString,
  getShiftSize,
  doWxCheck,
  configShare
} from '@/plugins/utils'
import { activityData } from '@/plugins/json'

import errorDialog from '../question/wrong.vue'
import QRCode from 'qrcodejs2'
import html2canvas from 'html2canvas'
import { getStatistical } from '@/plugins/api'
import { getRandomUuid } from '@/plugins/uuid'

export default {
  components: {
    errorDialog
  },
  data() {
    return {
      stratShow: false,
      wrongShow: false,
      currentData: {},
      nextData: {},
      questionListShow: false,
      selectOption: '',
      postersShow: false,
      postersData: {},
      imgUrl: '',
      shareTitleShow: false,
      shareImgShow: false,
      timeOutEvent: 0,
      currentUuid: '',
      statisticalData: {}
    }
  },
  created() {
    if (!this.currentUuid) {
      this.currentUuid = getRandomUuid()
    }
    this.selectOption = ''
    this.stratShow = true
    this.postersShow = false
    this.questionListShow = false
    var answerArr = activityData.answerLevel.one
    this.postersData = answerArr[Math.floor(Math.random() * answerArr.length)]
    getStatistical({ res: '000' })
    console.log(this.postersData, 'this.postersData', this.currentUuid)
  },
  methods: {
    startTest() {
      console.log('开始test')
      this.selectOption = ''
      this.stratShow = false
      this.questionListShow = true
      this.postersShow = false
      this.currentData = activityData.questionData[0]
      this.nextData = activityData.questionData[0]
    },
    //
    wrongClose(number) {
      this.wrongShow = false
      this.nextData = activityData.questionData[number]
      //  下方数据延迟更新
      setTimeout(() => {
        this.currentData = activityData.questionData[number]
        this.$refs.currentQuestion.classList.add('question_none')
      }, 1000)
      this.$refs.currentQuestion.classList.add('fadeOutDown')
      this.$refs.nextQuestion.classList.add('fadeInDown')
    },
    optionClick(data) {
      this.selectOption = data.id
      if (this.$refs.currentQuestion.classList.contains('fadeOutDown')) {
        this.$refs.currentQuestion.classList.remove('fadeOutDown')
      }
      if (this.$refs.nextQuestion.classList.contains('fadeInDown')) {
        this.$refs.nextQuestion.classList.remove('fadeInDown')
      }
      if (this.$refs.currentQuestion.classList.contains('question_none')) {
        this.$refs.currentQuestion.classList.remove('question_none')
      }
      if (this.currentData.number === '10') {
        this.wrongShow = false
        this.selectOption = ''
        this.questionListShow = false
        var answerArr = activityData.answerLevel.one
        this.postersData =
          answerArr[Math.floor(Math.random() * answerArr.length)]
        this.getQrcode()
        this.postersShow = true
        return;
      }
      if (this.selectOption != this.currentData.answer) {
        this.wrongShow = true
        this.selectOption = ''
        this.wrongData = this.currentData
      } else {
        setTimeout(() => {
          //  nextData 数据是真正显示数据      currentData数据   只是模拟向下掉的一个效果
          this.nextData = activityData.questionData[this.nextData.number]
          //  下方数据延迟更新
          setTimeout(() => {
            this.currentData =
              activityData.questionData[this.currentData.number]
            this.$refs.currentQuestion.classList.add('question_none')
          }, 1000)
          this.$refs.currentQuestion.classList.add('fadeOutDown')
          this.$refs.nextQuestion.classList.add('fadeInDown')
          this.selectOption = ''
        }, 1000)
      }
      console.log(this.selectOption, 'this.selectOption')
    },
    getQrcode() {
      // 获取分享页面地址
      var source = getQueryString('source')
      var qrcodeUrl =
        window.location.protocol +
        '//' +
        window.location.host +
        '/activity/question?source=share-pic'
      console.log(qrcodeUrl, 'this.qrcodeUrl')
      this.$nextTick(() => {
        this.createQrcode(qrcodeUrl)
      })
    },
    createQrcode(url) {
      // 生成二维码
      let qrcode = new QRCode(document.getElementById('QRCodeNone'), {
        width: getShiftSize(100),
        height: getShiftSize(100),
        text: url, // 二维码地址
        colorDark: '#000',
        colorLight: '#fff'
      })
      var myCanvas = document.getElementsByTagName('canvas')[0]
      var img = this.convertCanvasToImage(myCanvas)
      document.getElementById('qrcode').append(img)
      //
      let qrcodeShare = new QRCode(document.getElementById('QRCodeNoneShare'), {
        width: getShiftSize(90),
        height: getShiftSize(90),
        text: url, // 二维码地址
        colorDark: '#000',
        colorLight: '#fff'
      })
      var myCanvasShare = document.getElementsByTagName('canvas')[0]
      var imgShare = this.convertCanvasToImage(myCanvasShare)
      document.getElementById('qrcodeShare').append(imgShare)
    },
    // 将canvas返回的图片添加到image里
    convertCanvasToImage(canvas) {
      var image = new Image()
      image.src = canvas.toDataURL('image/png')
      return image
    },
    saveAndShareClick() {
      /**
       * 将页面指定节点内容转为图片
       * 1.拿到想要转换为图片的内容节点DOM；
       * 2.转换，拿到转换后的canvas
       * 3.转换为图片
       */
      html2canvas(this.$refs.imageDom).then(canvas => {
        // 转成图片，生成图片地址
        this.imgUrl = canvas.toDataURL('image/png')
        this.shareImgShow = true
      })
      doWxCheck().then(domain => {
        let _title = '分享测试'
        let _desc = '分享测试desc'
        let _icon = ''
        let _link =
          window.location.protocol +
          '//' +
          window.location.host +
          '/activity/question?source=share-link'
        configShare(_title, _desc, _icon, _link)
      })
    },

    closeShare() {
      this.shareImgShow = false
    },
    gotouchstart() {
      let that = this
      clearTimeout(this.timeOutEvent) // 清除定时器
      this.timeOutEvent = 0
      this.timeOutEvent = setTimeout(function() {
        // 执行长按要执行的内容，
      }, 600) //这里设置定时
    },
    gotouchend() {
      clearTimeout(this.timeOutEvent)
    },
    // 如果手指有移动，则取消所有事件，此时说明用户只是要移动而不是长按
    gotouchmove() {
      clearTimeout(this.timeOutEvent) //清除定时器
    }
  }
}
</script>

<style lang="scss" scoped>
@import '~@/assets/scss/_var.scss';

.question {
  width: 100%;
  font-size: 14px;
  height: 100%;
  .startMain {
    background: url(~@/assets/images/test_bg.png) no-repeat;
    height: 100%;
    width: 100%;
    background-size: 100% 100%;
    .start_button {
      position: absolute;
      bottom: 88px;
      background: url(~@/assets/images/test_btn.png) no-repeat;
      width: 152px;
      height: 49px;
      left: 50%;
      transform: translateX(-50%);
      background-size: 100% 100%;
      cursor: pointer;
    }
  }
  &__list {
    background: url(~@/assets/images/question_list_bg.png) no-repeat;
    height: 100%;
    width: 100%;
    background-size: 100% 100%;
    position: absolute;
    left: 0px;
    top: 0px;
    .number {
      background: #fff;
      color: $blue1;
      border-radius: 50%;
      font-size: 36px;
      font-weight: 500;
      width: 45px;
      height: 45px;
      line-height: 45px;
      position: absolute;
      left: 50%;
      transform: translateX(-50%);
      top: 32px;
      letter-spacing: -1px;
    }
    .title {
      background: url(~@/assets/images/question_title_bg.png) no-repeat;
      height: 120px;
      width: 352px;
      background-size: 100% 100%;
      top: 174px;
      position: absolute;
      left: 50%;
      transform: translateX(-50%);
      &__content {
        height: 63px;
        width: 318px;
        position: absolute;
        left: 14px;
        line-height: 21px;
        top: 15px;
        text-align: left;
        .inline {
          position: absolute;
          top: 50%;
          transform: translateY(-50%);
          color: $blue;
          font-weight: 600;
          font-size: 14px;
          letter-spacing: 0px;
          &.large {
            font-size: 20px;
            font-weight: 500;
          }
        }
      }
    }
    .options {
      height: 240px;
      background: #fff;
      border-radius: 5px;
      width: 336px;
      position: absolute;
      left: 50%;
      transform: translateX(-50%);
      bottom: 20px;
      padding: 12px 0;
      &__item {
        height: 48px;
        width: 318px;
        color: #fff;
        margin: 6px 9px 12px 9px;
        position: relative;
        cursor: pointer;
        .content {
          width: 318px;
          font-size: 12px;
          text-align: left;
          line-height: 17px;
          height: 48px;
          position: absolute;
          background: url(~@/assets/images/question_option_bg.png) no-repeat;
          background-size: 100% 100%;
          &.active {
            height: 58px;
            position: absolute;
            top: -10px;
            background: url(~@/assets/images/question_true_answer.png) no-repeat;
            background-size: 100% 100%;
            .inline {
              margin-top: 5px;
            }
          }
          .inline {
            margin-left: 122px;
            line-height: 28px;
            font-weight: 600;
            font-size: 20px;
            position: absolute;
            top: 50%;
            transform: translateY(-50%);
            &.small {
              font-size: 16px;
              margin-left: 70px;
              line-height: 17px;
            }
            &.mini {
              font-size: 12px;
              margin-left: 20px;
              line-height: 17px;
            }
          }
        }
      }
    }
  }
  .posters {
    height: 100%;
    width: 100%;
    &__main {
      height: 100%;
      width: 100%;
      &.levelOne {
        background: url(~@/assets/images/posters_bg_level1.png) no-repeat;
        background-size: 100% 100%;
      }
      &.levelTwo {
        background: url(~@/assets/images/posters_bg_level2.png) no-repeat;
        background-size: 100% 100%;
      }
      &.levelThree {
        background: url(~@/assets/images/posters_bg_level3.png) no-repeat;
        background-size: 100% 100%;
      }
      .posters__content {
        position: absolute;
        left: 38px;
        top: 86px;
        width: 332px;
        height: 348px;
        .top {
          text-align: left;
          margin-bottom: 30px;
          .eName {
            font-size: 12px;
            font-weight: bold;
            letter-spacing: 2px;
            font-size: 10px;
            height: 12px;
            margin-top: 44px;
            margin-bottom: 8px;
          }
          .cName {
            font-size: 50px;
            font-weight: 600;
            height: 50px;
            line-height: 50px;
          }
        }
        .center {
          font-size: 14px;
          font-weight: 500;
          line-height: 18px;
          letter-spacing: 2px;
          width: 234px;
          white-space: pre;
          text-align: left;
        }
      }

      .describe__image {
        position: absolute;
        bottom: 140px;
        left: 46px;
        width: 154px;
        height: 32px;
        background: url(~@/assets/images/describe__image.png) no-repeat;
        background-size: 100% 100%;
      }
      .qrcode {
        position: absolute;
        bottom: 88px;
        right: 41px;
        width: 100px;
        height: 100px;
      }
      .qrcodeTitle {
        position: absolute;
        bottom: 72px;
        right: 42px;
        width: 100px;
        height: 12px;
        font-size: 8px;
        color: #fff;
        line-height: 12px;
        text-align: center;
      }
      &.share {
        position: absolute;
        right: 10000000000000000000000px;
        .qrcode {
          position: absolute;
          bottom: 88px;
          right: 41px;
          width: 90px;
          height: 90px;
        }
        .qrcodeTitle {
          position: absolute;
          bottom: 62px;
          right: 22px;
          width: 120px;
          height: 12px;
          font-size: 10px;
          color: #fff;
          line-height: 12px;
          text-align: center;
          letter-spacing: 2px;
        }
        .describe__share {
          position: absolute;
          bottom: 120px;
          left: 46px;
          width: 159px;
          height: 33px;
          background: url(~@/assets/images/describe__share.png) no-repeat;
          background-size: 100% 100%;
        }
      }
    }
    .share_btn {
      background: url(~@/assets/images/share_btn.png) no-repeat;
      background-size: 100% 100%;
      position: absolute;
      bottom: 72px;
      width: 152px;
      height: 49px;
      left: 48px;
      cursor: pointer;
    }
  }
}
.wrong_popup {
  background: none;
}
.popup__custom {
  background: none;
  .popupShare__main {
    .share-title {
      font-size: 16px;
      color: #fff;
      width: 200px;
      height: 20px;
      line-height: 20px;
      position: absolute;
      left: 50%;
      top: 10px;
      transform: translateX(-50%);
    }
    .share-close {
      font-size: 26px;
      color: #fff;
      right: 0;
      top: 0px;
      position: absolute;
      cursor: pointer;
    }
    .share-img {
      width: 320px;
      margin-top: 35px;
    }
  }
}
.question_none {
  display: none;
}
</style>
