<template>
  <div class="sidebar-single-enter-animate page-top" v-if="isShowTopImg" id="page-top">
    <div class="top-mask" :style="setBackgroundUrl"></div>
    <div class="top-image" id="top-image" v-if="isShowHeadLine">
      <h1>{{headLine}}</h1>
    </div>
    <slot name="top1"></slot>
    <slot name="top2"></slot>
    <slot name="top3"></slot>
    <slot name="top4"></slot>
    <slot name="top5"></slot>
    <div class="page-record-control" v-if="isShowHeadLine">
      <div class="page-top-record" id="page-top-record">
        <div class="page-record-bot-common page-record-top">
          <div class="page-record-top-left page-record-single-common">
            <span class="aurora-iconfont-common aurora-page-word"></span>&nbsp;
            <span class="page-record-single-desc">总字数</span>
            <span>{{contentLength}}</span>
          </div>
          <div class="page-record-top-right page-record-single-common">
            <span class="aurora-iconfont-common aurora-page-time"></span>&nbsp;
            <span class="page-record-single-desc">时长</span>
            <span>{{getSugTime}}</span>
          </div>
        </div>
        <div class="page-record-bot-common page-record-center">
          <div class="page-record-center-left page-record-single-common">
            <span class="aurora-iconfont-common aurora-page-comment"></span>&nbsp;
            <span class="page-record-single-desc">评论数</span>
            <span :id="pathName" class="waline-comment-count" />
            <!--<span>{{$store.state.commentCount}}</span>-->
          </div>
          <div class="page-record-center-right page-record-single-common">
            <span class="aurora-iconfont-common aurora-page-read"></span>&nbsp;
            <span class="page-record-single-desc">总阅读数</span>
            <span :id="pathName" class="waline-visitor-count" />
            <!--<span>{{$store.state.readCount}}</span>-->
          </div>
        </div>
        <div v-if="tagArr.length !== 0" class="page-record-bot-common page-record-bot">
          <div class="page-record-bot-tag" id="page-record-bot-tag">
            <span class="aurora-iconfont-common aurora-page-tag"></span>&nbsp;
            <router-link v-for="(item,index) in tagArr" :to="goTag(item)">
              <span class="home-page-tag-span page-record-tag-span">{{item}}</span>
            </router-link>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {useThemeLocaleData} from "../../composables";
import WordCount from "crisp-word-count";

const network = require('../../public/js/network.js')
export default {
  name: "TopImage",
  data() {
    return {
      animeImg: '',
      pageMap: '',
      contentLength: 0,
      tagArr: [],
      topBackgroundUrl: 'https://picture.cco.vin/pic/rmimg',
      pathName: '',
      sugCountPerMin: 230,
      document: {}
    }
  },
  props: {
    isShowTopImg: {
      type: Boolean,
      default() {
        return false
      }
    },
    isShowHeadLine: {
      type: Boolean,
      default() {
        return false
      }
    },
    headLine: {
      type: String,
      default() {
        return "aurora";
      }
    },
    themeProperty: {
      type: Object
    }
  },
  created() {
    if (this.themeProperty.sugCountPerMin !== undefined) {
      this.sugCountPerMin = this.themeProperty.sugCountPerMin
    }
    this.setPathName(this.$route.path)
    this.$router.beforeEach((to,from,next) => {
      this.setPathName(to.path)
      next()
    })
  },
  mounted() {
    this.document = document
    this.pathName = window.location.pathname
    setTimeout(() =>{
      this.getPageMap()
    },500)
  },
  computed: {
    goTag() {
      return (item) => {
        return '/tag?tag=' + item
      }
    },
    setBackgroundUrl() {
      let path = this.$route.path
      let customTop = this.themeProperty.customTopImg

      let imgPath = ""
      if (customTop === undefined) {
        return this.getRandomBg()
      }

      if (customTop.custom === undefined || !customTop.custom) {
        //用户自定义
        return this.getRandomBg()
      }

      //用户自定义 使用用户的图片
      if (path.search("link") !== -1) {
        let arr = this.themeProperty.customTopImg.friend
        imgPath = this.getCustomTopImgPath(arr)
      }else if (path.search("tag") !== -1) {
        let arr = this.themeProperty.customTopImg.tag
        imgPath = this.getCustomTopImgPath(arr)
      }
      else if (path.search("mood") !== -1) {

        let arr = this.themeProperty.customTopImg.mood
        imgPath = this.getCustomTopImgPath(arr)
      }else {
        let arr = this.themeProperty.customTopImg.page
        imgPath = this.getCustomTopImgPath(arr)
      }

      return "background-image: url(" + imgPath + ");"
    },
    getWordLength() {
      let content = this.pageMap.content + ""
      if (content.length === undefined || content.length === null) {
        this.length = 0
      }else {
        this.length = content.length
      }
      return this.length
    },
    getSugTime() {

      return  Math.floor(this.contentLength / this.sugCountPerMin) === 0 ? 1 : Math.ceil(this.contentLength / this.sugCountPerMin);
    },
    getRandom() {
      return this.getRandomInt(0,99999)
    }
  },
  methods: {
    setPathName(pathName) {
      this.pathName = pathName
    },
    getRandomBg() {
      //用户没有自定义图片，使用随机图片
      let num1 = this.getRandomInt(-9999,999)
      let num2 = this.getRandomInt(0,300)
      let num3 = this.getRandomInt(0,30)
      let num = num2 / num3 * num1 + num2

      const themeLocale = useThemeLocaleData()

      let homePageImgApi = themeLocale.value.homePageImgApi

      if (homePageImgApi === undefined) {
        homePageImgApi = this.$store.state.defaultHomePageImgApi
      }

      let path = homePageImgApi + "?time=" + num
      return "background-image: url(" + path + ");"
    },
    getCustomTopImgPath(pathArr) {
      if (pathArr.length === 0) {
        try {
          let path = pathArr[0]
          return path
        }catch (e) {
          return pathArr
        }
      }
      //随机选择一个
      let path = pathArr[this.getRandomInt(0,pathArr.length - 1)]
      return path
    },
    getPageMap() {
      let allPageMap = this.$store.state.allPageMap
      for (let i = 0; i < allPageMap.length; i++) {
        if (this.pathName === allPageMap[i].articleUrl) {
          this.pageMap = allPageMap[i]
          if (this.pageMap.categories.length === 0) {
            this.tagArr = this.pageMap.tag
          }else {
            this.tagArr = this.pageMap.categories
          }
        }
      }
    },
    getRandomInt(min, max) {
      min = Math.ceil(min);
      max = Math.floor(max);
      return Math.floor(Math.random() * (max - min)) + min; //不含最大值，含最小值
    },
    countNum() {
      new Promise((resolve,reject) => {
        let allContent = ''
        try {
          allContent = this.document.querySelector(".medium-zoom-content").innerText;
        }catch (e) {
          return
        }
        let allCodeElement = this.document.querySelectorAll(".medium-zoom-content div[class*=language-]");
        for (let i = 0; i < allCodeElement.length; i++) {
          let codeContent = allCodeElement[i].innerText;
          let indexOf = allContent.indexOf(codeContent);
          if (indexOf === -1) {
            continue
          }
          let startContent = allContent.substr(0,indexOf)
          let endContent = allContent.substr((indexOf + codeContent.length), allContent.length)
          startContent = startContent.replace("\n","")
          endContent = endContent.replace("\n","")
          allContent = startContent + endContent
          allContent = allContent.replace("\n","")
        }
        resolve(allContent)
      }).then((allContent) => {
        const string_summary = WordCount.stringSummary(allContent)
        new Promise((resolve,reject) => {
          let chineseContentNum = 0
          for (let i = 0; i < allContent.length; i++) {
            let c = allContent.charAt(i);
            //基本汉字
            if (c.match(/[\u4e00-\u9fa5]/)) {
              chineseContentNum++;
            }
            //基本汉字补充
            else if (c.match(/[\u9FA6-\u9fcb]/)){
              chineseContentNum++;
            }
          }
          resolve(chineseContentNum)
        }).then((chineseContentNum) => {
          this.contentLength = chineseContentNum + string_summary.spaces
        })
      })
    }
  },
  watch: {
    headLine(newValue,oldValue) {
      setTimeout(() => {
        this.pathName = window.location.pathname
        this.getPageMap()
      },500)
      if (this.isShowHeadLine) {
        setTimeout(() => {
          this.countNum()
        },1000)
      }
    },
    pathName() {
      if (this.isShowHeadLine) {
        setTimeout(() => {
          this.countNum()
        },1000)
      }
    }
  }
}
</script>

<style scoped>

</style>