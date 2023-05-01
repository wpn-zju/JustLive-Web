<template>
  <div v-loading="loading" element-loading-background="rgba(243, 246, 248, 0.8)" class="search-container">
    <div class="search-bar">
      <el-select class="search-bar-select" size="middle" v-model="value" placeholder="选择平台">
        <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value">
        </el-option>
      </el-select>
      <el-input size="middle" class="search-bar-input" v-model="keyWord" @keydown.enter.native="toSearch"></el-input>
      <el-button class="search-bar-btn" type="primary" @click="toSearch">搜索</el-button>
    </div>
    <div class="search-result">
      <div class="search-result-list">
        <el-row class="search-result-row" :gutter="30">
          <el-col class="search-result-col" :span="8" v-for="(owner, index) in resultList" :key="index">
            <el-card @click.native="toRoom(owner.platform, owner.roomId)" class="search-result-card" shadow="hover">
              <div class="search-result-card-left">
                <div class="search-result-card-avatar">
                  <img class="search-head-pic" :src=owner.headPic />
                </div>
              </div>
              <div class="search-result-card-right">
                <div class="result-room-name">
                  {{ owner.roomName }}
                </div>
                <div class="result-name">
                  {{ getPlatform(owner.platform) }} · {{ owner.nickName }}
                </div>
                <div>
                  <div :class="isLive(owner.isLive) ? 'info-isLive' : 'info-notLive'">{{ isLive(owner.isLive) ? "直播中" : "未开播" }}</div>
                  <div class="result-follows">
                  {{ handleOnline(owner.followers) }} 已关注
                  </div>
                </div>
              </div>
            </el-card>
          </el-col>
        </el-row>
      </div>
    </div>
  </div>
</template>

<script>
import {getSearch} from "@/api/liveList";

export default {
  name: "Search",
  data(){
    return {
      keyWord: '',
      options: [{
        value: 'all',
        label: '所有平台'
      }, {
        value: 'douyu',
        label: '斗鱼'
      }, {
        value: 'bilibili',
        label: '哔哩哔哩'
      }, {
        value: 'huya',
        label: '虎牙'
      }, {
        value: 'cc',
        label: '网易CC'
      }, {
        value: 'egame',
        label: '企鹅电竞'
      }],
      value: 'all',
      selectType: '0',
      resultList: [],
      loading: false,
      userInfo: {},
    }
  },
  methods: {
    init(){
      this.$emit("inSearch")
      if (localStorage.getItem('userInfo')) {
        this.userInfo = JSON.parse(localStorage.getItem('userInfo'))
      }
      this.keyWord = this.$route.query.keyWord
      this.toSearch()
      let li = document.getElementsByClassName("result-li")[0]
      li.style.color = '#007ACC'
    },
    isLive(isLive){
      if (isLive == "0"){
        return false
      }else {
        return true
      }
    },
    toRoom(platform, roomId){
      this.$router.push({ name: 'liveRoom', query:{ platform : platform, roomId : roomId } });
    },
    handleOnline(online){
      let num = online.toString().trim()
      if (num.length > 4){
        let numCut = num.substring(0, num.length-4)
        let afterPoint = num.substring(num.length-4,num.length-3)
        return numCut+'.'+afterPoint+'万'
      }else {
        return num+'人'
      }
    },
    toSearch(){
      if(this.keyWord.trim()!=''){
        // if (this.userInfo.uid == undefined) {
        //   this.$message({
        //     message: '搜索功能需登录使用',
        //     type: 'warning'
        //   });
        //   return;
        // }
        this.$router.push({ name: 'search', query:{ keyWord : this.keyWord } })
        this.resultList = []
        this.loading = true
        getSearch(this.value, this.keyWord, this.userInfo.uid)
            .then(response => {
              if (response.data.code == 200) {
                this.resultList = response.data.data.sort((ownera, ownerb) => ownerb.isLive - ownera.isLive)
              } else {
                this.$message({
                  message: '请求过多',
                  type: 'warning'
                });
              }
              this.loading = false
            })
      }
    },
    getPlatform(platForm){
      if (platForm == 'bilibili'){
        return '哔哩哔哩'
      }
      if (platForm == 'douyu'){
        return '斗鱼'
      }
      if (platForm == 'huya'){
        return '虎牙'
      }
      if (platForm == 'cc'){
        return '网易CC'
      }
    },
  },
  mounted() {
    this.init()
  },
  activated() {
    console.log('active')
    this.$emit("inSearch")
  },
  beforeDestroy(){
    this.$emit("exitSearch")
  },
}
</script>

<style scoped>
.search-container{
  position: relative;
  height: 100%;
  width: 100%;
  overflow-x: hidden;
}
.search-container::-webkit-scrollbar {
  width: 8px;
  /*height: 4px;*/
}
.search-container::-webkit-scrollbar-thumb {
  border-radius: 10px;
  background: #8e8e8e;
}
.search-bar{
  width: 100%;
  padding: 20px;
  height: auto;
  overflow: hidden;
}
.search-bar-select{
  margin-left: 30%;
  width: 110px;
}
.search-bar-input{
  margin-left: 10px;
  width: 300px;
}
.search-bar-btn{
  margin-left: 10px;
}
.search-result{
  width: 100%;
}
.result-ul{
  position: absolute;
  left: 40%;
  list-style: none;
  height: 25px;
  margin-block: 5px;
}
.result-li{
  font-weight: bold;
  cursor: pointer;
  position: relative;
  margin-right: 20px;
  text-align: center;
  width: auto;
  display: inline;
  float:left;
  height: 100%;
}
.result-li:hover{
  color: #007ACC;
}
.under-result-li{
  position: absolute;
  bottom: 0px;
  height: 4px;
  width: 33px;
  background: #007ACC;
  transition: all 0.3s;
  transform: translateX(0px);
}
.search-result-list{
  width: 100%;
  top: 10px;
  bottom: 10px;
}
.search-result-row {
  width: 100%;
  padding: 20px;
}
.search-result-col{
  width: 390px;
  height: 130px;
  margin-bottom: 20px;
  margin-left: 20px;
}
.search-result-card-left{
  width: 90px;
  height: 90px;
  display: inline-block;
  vertical-align: top;
}
.search-result-card-right{
  width: 220px;
  height: 90px;
  padding-left: 10px;
  display: inline-block;
  vertical-align: top;
}
.search-result-card-avatar{
  top: 10px;
  left: 10px;
  bottom: 10px;
}
.search-head-pic{
  width: 100%;
  height: 100%;
  border-radius: 10px;
}
.search-result-card{
  height: auto;
  width: 100%;
  transition: all 0.2s;
}
.search-result-card:hover{
  cursor: pointer;
  transform: scale(1.1);
}
.info-isLive{
  margin-top: 6px;
  margin-right: 5px;
  padding: 1px 4px 4px 4px;
  height: 18px;
  width: auto;
  background-color: #c10f0f;
  border-radius: 2px;
  font-size: 13px;
  font-weight: 600;
  text-align: center;
  color: #F3F6F8;
  display: inline-block;
  vertical-align: center;
}
.info-notLive{
  margin-top: 6px;
  margin-right: 5px;
  padding: 1px 4px 4px 4px;
  height: 18px;
  width: auto;
  background-color: #979797;
  border-radius: 2px;
  font-size: 13px;
  font-weight: 600;
  text-align: center;
  color: #F3F6F8;
  display: inline-block;
  vertical-align: center;
}
.result-follows{
  margin-top: 6px;
  margin-left: 5px;
  width: auto;
  max-width: fit-content;
  font-weight: normal;
  font-size: 13px;
  display: inline-block;
  vertical-align: center;
}
.result-name{
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
  width: auto;
}
.result-room-name{
  margin-bottom: 6px;
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
  width: auto;
}
</style>
