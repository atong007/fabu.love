<template>
  <div class="applist-wrapper">
    <!--内容头部-->
    <div class="applist-header">
      <div class="left" style="position: relative">
        <div
          style="width: 120px;height: 16px;background-color: #6477F2;position: absolute;top: 30px;left: 12px;border-radius: 10px;filter: blur(10px);z-index: -1"></div>
        <el-button class="uploadWrapper button-style-main"><i class="icon-ic_upload"></i>上传应用</el-button>
        <input
          alt=""
          title="上传应用"
          ref="referenceUpload"
          accept=".ipa, .apk"
          @change="referenceUpload"
          type="file"
          style="position: absolute;top: 0px;left: 0px;width: 144px;height: 48px;opacity: 0;cursor:pointer;">
      </div>


      <!--++++++++++++++++-->
      <!-- 添加by Atong007 -->
      <!--++++++++++++++++-->
      <div class="left">
          <!--生成下载二维码-->
          <div class="scan-code-list">
            <el-popover
              ref="popover"
              placement="bottom-start"
              width="144"
              left="400"
              @show="popovershow"
              @hide="popoverhide"
              trigger="click"
              :disabled="!this.isAppList || originDataList.length === 0"
              :visible-arrow="false">
              <ul>
                <li class="leftWrapper-item" v-for="(item, index) in this.appList" :key="index" @click="selectApp(item)">
                  <p>
                    {{item.appName}}
                  </p>
                </li>
              </ul>
            </el-popover>
            <el-button class="scanCodeBtn" v-popover:popover @click="clickTeamBtn">{{this.selectedAppName}}  <i class="el-icon-arrow-down" ref="arrow"></i></el-button>

            <el-button class="flagBtn" @click="clickFlagBtn" v-show="!isAppList"></el-button>
          </div>
        </div>
        <!--++++++++++++++++-->
        <!--    添加结束     -->
        <!--++++++++++++++++-->



      <div class="applist-header-right">
        <div class="platform-wrapper">
          <div class="platform-ios" :class="getActiveClass('ios')" @click="clickIosPlatform">
            <span class="platformImg icon-ic_ios"></span>
          </div>
          <div class="platform-android" :class="getActiveClass('android')" @click="clickAndroidPlatform">
            <span class="platformImg icon-ic_andr"></span>
          </div>
        </div>
        <div class="search-wrapper">
          <i class="el-icon-search"></i>
          <input class="applist-header-search" v-model="queryText" type="text" placeholder="输入名称搜索">
        </div>
      </div>
    </div>
    <!--应用列表-->
    <collectionView
      :dataArr="this.dataList"
      @gotoAppDetail="gotoAppDetail"
    >
    </collectionView>

    <uploadApp v-if="this.showUploadView" :teamId="this.currentTeam._id" :appFile="this.file"
               @closeUpload="closeUploadMethod"
               @uploadSuccess="uploadSuccessMethod"></uploadApp>

    <emptyView v-if="this.showEmpty"></emptyView>

  </div>
</template>

<script type="text/ecmascript-6">
  import * as AppResourceApi from '../../api/moudle/appResourceApi'
  import UploadApp from './uploadApp.vue'
  import { getUserTeam } from '../../mgr/userMgr'
  import CollectionView from '../base/collectionView.vue'
  import EmptyView from './emptyView.vue'

  export default {
    name: 'Apps',
    data() {
      return {
        currentPlatform: '',
        dataList: [],
        originDataList: [],
        queryText: '',
        showUploadView: false,
        file: FileList,
        currentPage: 0,
        currentTeam: {},
        showEmpty: false,

        selectedAppName: '生成二维码',
        appName: '',
        isAppList: true,
        appList: []
      }
    },
    components: {
      UploadApp, CollectionView, EmptyView
    },
    computed: {},
    methods: {
      loadAppList() {
        AppResourceApi.getAppList(this.currentTeam._id)
          .then(response => {
            this.dataList = []
            this.dataList = response.data.reverse()
            this.originDataList = this.dataList
            if (this.dataList.length === 0) {
                this.showEmpty = true
            } else {
              this.showEmpty = false
            }

            console.log(this.dataList)


            // ----------------------------------------
            // 添加by Atong007
            // ----------------------------------------

            var newAppList = []
            this.dataList.forEach((item) => {
              var newAppItem = {}
              var isNewApp = true
              newAppList.every(function(element, index) {
                if (element.appName === item.appName || element.appName === item.bundleName) {
                  isNewApp = false
                  if (item.platform === 'ios') {
                    element.shortUrls = item.shortUrl + '#' + element.shortUrls
                  } else {
                    element.shortUrls += '#' + item.shortUrl
                  }
                }
                return isNewApp !== false
              })
              if (isNewApp) {
                if (item.appName !== undefined) {
                  newAppItem.appName = item.appName
                } else {
                  newAppItem.appName = item.bundleName
                }
                newAppItem.shortUrls = item.shortUrl
                newAppItem.platform = item.platform
                newAppList.push(newAppItem)
              }
            })
            console.log(newAppList)
            this.appList = newAppList

            // ----------------------------------------
            // 添加by Atong007
            // ----------------------------------------


          }, reject => {
            this.$message.error(reject)
            this.showEmpty = true
          })
      },
      loadMore() {
      },
      referenceUpload(e) {
        this.file = e.target.files
        if (e.target.files.length > 0) {
          this.showUploadView = true
        }
      },
      closeUploadMethod() {
        this.showUploadView = false
        this.$refs.referenceUpload.value = ''
      },
      uploadSuccessMethod() {
        this.showUploadView = false
        this.$refs.referenceUpload.value = ''
        this.loadAppList()
      },
      clickIosPlatform() {
          if (this.currentPlatform === 'ios') {
            this.currentPlatform = ''
          } else {
            this.currentPlatform = 'ios'
          }
      },
      clickAndroidPlatform() {
        if (this.currentPlatform === 'android') {
          this.currentPlatform = ''
        } else {
          this.currentPlatform = 'android'
        }
      },
      getActiveClass(flag) {
        if (flag === this.currentPlatform) {
          return 'platformActive'
        }
      },
      gotoAppDetail(item) {
        this.$router.push({
          name: 'AppDetail',
          params: {appId: item._id}
        })
        this.bus.$emit('appdetail')
      },
      appItemHovered() {
      },
      appItemUnhovered() {
      },
      clickEditorBtn() {
      },
      clickPreview() {
      },
      delectApp(item) {
        this.$confirm('确认删除？')
          .then(_ => {
            AppResourceApi.delectApp(this.currentTeam._id, item._id).then((res) => {
              this.loadAppList()
            }, reject => {
              this.$message.error(reject)
            })
          })
          .catch(_ => {})
      },

      // ----------------------------------------
      // 添加by Atong007
      // ----------------------------------------

      selectApp(item) {
        console.log('++++++++selectApp+++++++++++')
        console.log(item)

        // 模拟点击，取消弹框
        document.querySelector('#app').click()

        this.selectedAppName = item.appName

        // 生成下载二维码
        this.generateScanCode(item)
        // 刷新app列表
        // this.bus.$emit('refreshList')
      },
      clickTeamBtn() {
        if (this.isAppList) {
        } else {
        }
      },
      clickFlagBtn() {
        this.$router.push('/apps')
      },
      popovershow() {
        this.$refs.arrow.style.transform = `rotate(-180deg)`
      },
      popoverhide() {
        this.$refs.arrow.style.transform = `rotate(0deg)`
      },

      generateScanCode(item) {
        const {href} = this.$router.resolve({
          name: 'ScancodeDirection',
          path: '/',
          params: { 'id': item.shortUrls }
        })
        window.open(href, '_blank')
      }
      // ----------------------------------------
      // 结束添加
      // ----------------------------------------

    },
    destroyed() {
      this.bus.$off('refreshList')
    },
    mounted() {
      this.currentTeam = getUserTeam()
      this.bus.$emit('applist')
      this.bus.$on('refreshList', () => {
        this.currentTeam = getUserTeam()
        this.loadAppList()
      })
      this.$watch('queryText', () => {
        let newArr = []
        this.dataList.forEach((item) => {
          if (item.appName.search(this.queryText) !== -1) {
            newArr.push(item)
          }
        })
        this.dataList = newArr

        if (this.queryText.length === 0) {
          this.dataList = this.originDataList
        }
      })

      this.loadAppList()
    },
    created () {
      this.$nextTick(() => {
      })
    },
    watch: {
      currentPlatform(val) {
        this.dataList = this.originDataList
        if (val === '') {
          if (this.dataList.length === 0) {
            this.showEmpty = true
          } else {
            this.showEmpty = false
          }
          return
        }
        let newArr = []
        this.dataList.forEach((item) => {
          if (item.platform === val) {
            newArr.push(item)
          }
        })
        this.dataList = newArr
        if (this.dataList.length === 0) {
          this.showEmpty = true
        } else {
          this.showEmpty = false
        }
      }
    }
  }
</script>

<style lang="scss">
  @import "../../common/scss/base";

  .applist-wrapper {
    padding-left: 20px;
    padding-right: 20px;
    min-width: 660px;
  }
  .applist-header {
    height: 75px;
    padding-top: 25px;
  }
  .applist-header .uploadWrapper {
    width: 144px;
    float: left;
  }
  .applist-header .uploadWrapper i {
    margin-right: 15px;
  }
  .applist-header-right {
    float: right;
  }
  .applist-header .search-wrapper {
    width: 312px;
    height: 48px;
    border-radius: 24px;
    border: solid 1px $mainColor;
    position: relative;
    display: inline-block;
  }
  .el-icon-search {
    position: absolute;
    left: 15px;
    top: 17px;
    color: $mainColor;
  }

  .applist-header-search {
    position: absolute;
    left: 45px;
    top: 0px;
    width: 235px;
    height: 50px;
    background-color: rgba(0, 0, 0, 0);
    outline: 0;
    color: $mainColor;
  }

  .applist-header-search::-webkit-input-placeholder {
    color: $mainColor;
  }

  .applist-header-search:-moz-placeholder {
    color: $mainColor;
  }

  .applist-header-search:-ms-input-placeholder {
    color: $mainColor;
  }

  .applist-header .platform-wrapper {
    display: inline-block;
    width: 144px;
    height: 48px;
    border-radius: 24px;
    overflow: hidden;
    font-size: 0px;
    border: solid 1px $mainColor;
    margin-right: 22px;
    vertical-align: top;
  }

  .applist-header .platform-wrapper .platform-ios {
    display: inline-block;
    width: 72px;
    height: 100%;
    text-align: center;
    border-right: solid 1px $mainColor;
    box-sizing: border-box;
    vertical-align: top;
  }
  .applist-header .platform-wrapper .platform-android {
    display: inline-block;
    width: 72px;
    height: 100%;
    text-align: center;
  }
  .platformImg {
    line-height: 48px;
    font-size: 26px;
  }

  .platformActive {
    background-color: #e0e4fc;
  }

  .scanCodeBtn {
    background: #6477F2;
    color: #FFF;
    height: 48px;
    border-radius: 24px;
    margin-left: 20px;
  }

</style>
