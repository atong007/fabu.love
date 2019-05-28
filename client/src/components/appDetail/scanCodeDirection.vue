

<template>
    <div class="scanviewapp-wrapper">
      <!--右侧二维码-->
      <div class="scanview-mobilewrapper" v-show="!isPhone">
        <img class="mobieImg" src='../../assets/ic_mobilphone.png'>
        <vue-qr class="qrcodeImg" :text="downloadUrl" :logoSrc="logoUrl" size="500" margin="5" logoMargin="3" whiteMargin="true" :callback="downloadQrcode"></vue-qr>
        <p class="codetips">请扫描二维码下载APP</p>
      </div>
    </div>
</template>

<script type="text/ecmascript-6">
  import * as AppResourceApi from '../../api/moudle/appResourceApi'
  import VueQr from 'vue-qr'
  import PhoneWrapper from './phoneWrapper.vue'

  export default {
    components: {
      VueQr, PhoneWrapper
    },
    data() {
      return {
        versionArr: [],
        appVersionInfo: {},
        appBaseData: null,
        downloadUrl: '',
        logoUrl: '',
        platformStr: '',
        pwd: '',
        isPhone: false
      }
    },
    computed: {
      isIos() {
        var u = navigator.userAgent
        var isiOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/)
        return isiOS
      },
      isAndroid() {
        var u = navigator.userAgent
        var isAndroid = !!(u.match(/(Android)\s+([\d.]+)/))
        return isAndroid
      }
    },
    mounted() {
      // 判断是否是手机设备
      if (this.isIos || this.isAndroid) {
        this.isPhone = true
      } else {
        this.isPhone = false
      }
      this.checkShortUrls(this.$route.params.id)
    },
    created() {
      this.$nextTick(() => {
      })

    },
    methods: {
      getTableBackground(index) {
        if (index % 2 === 0) {
          return `backgroundColor: rgb(244, 245, 247)`
        } else {
          return `backgroundColor: white`
        }
      },
      checkShortUrls(shortUrls) {
        var shortUrlArray = shortUrls.split('#')
        var shortUrl = shortUrlArray[this.isIos ? 0 : 1]

        if (!this.isPhone) {
          this.getAppInfo(shortUrl)
          return
        }

        this.$router.push({
          name: 'AppPreView',
          path: '/',
          params: { 'id': shortUrl }
        })

      },
      getAppInfo(shortUrl) {
        AppResourceApi.getAppInfoByShortUrl(shortUrl).then((res) => {
          if (res.data.version === null) {
              this.$message.error('未检测到版本信息')
              return
          }
          this.appVersionInfo = res.data.version
          this.appBaseData = res.data.app
          let releaseDate = new Date(this.appVersionInfo.uploadAt)
          this.platformStr = res.data.app.platform
          this.appVersionInfo.creatDateStr = `${releaseDate.getFullYear()}-${releaseDate.getMonth() + 1}-${releaseDate.getDate()}`
          if (this.appBaseData.installPwd === 1) {
            this.installWithPwd = true
          } else {
            this.installWithPwd = false
          }
          // 获取logo地址
          this.logoUrl = this.getIconUrl()
          this.downloadUrl = `${window.origin}${this.$route.fullPath}`
        }, reject => {
          this.$message.error('服务器错误')
        })
      },
      getIconUrl() {
        return `${this.axios.defaults.baseURL}${this.appBaseData.icon}`
      },
      downloadQrcode(dataUrl, id) {
        const a = document.createElement('a')
        a.href = dataUrl
        a.download = 'downloadQrcode.png'
        a.click()
      }
    }
  }
</script>

<style lang="scss">
  @import "../../common/scss/base";
  body{
    background-color: white;
  }
  /*网页样式*/
  .scanviewapp-wrapper {
    position: absolute;
    background-color: white;
    width: 100%;
    height: 100%;
    background-image: url("../../assets/bg_picture.png");
    background-size: cover;
  }
  .scanview-mobilewrapper {
    display: block;
    width: 25%;
    height: 100%;
    vertical-align: top;
    position: relative;
    top: auto;
    margin-left: auto;
    margin-right: auto;
    text-align: center;
  }

  .scanview-mobilewrapper > img {
    margin-top: 120px;
    width: 450px;
    position: absolute;
    left: 0px;
    height: auto;
  }

  .scanviewapp-wrapper .title {
    color: #354052;
    font-weight: bold;
    font-size: 26px;
    height: 37px;
    line-height: 37px;
    margin-top: 33px;
  }
  .scanviewapp-wrapper .desc {
    margin-right: 12px;
  }
  .scanview-mobilewrapper .qrcodeImg img{
    position: absolute;
    width: 250px;
    height: 250px;
    left: 36px;
    margin-top: 280px;
  }

  .scanview-mobilewrapper .codetips {
    color: #354052;
    font-size: 14px;
    text-align: center;
    line-height: 20px;
    height: 20px;
    left: 85px;
    top: 540px;
    position: absolute;
  }
  .scanview-mobilewrapper .platform {
    color: #354052;
    opacity: 0.5;
    font-size: 14px;
    text-align: center;
    line-height: 20px;
    height: 20px;
    left: 60px;
    top: 400px;
    position: absolute;
  }
</style>
