<template>
  <div class="hello" v-loading="scanloding" :class="{listShow:listShow}" :element-loading-text="loadingText"
    element-loading-background="rgba(0, 0, 0, 0.4)">
    <transition name="fade">
      <div class="bg" v-show="deletes||guide!=0"></div>
    </transition>
    <transition name="fade">
      <div class="delete" v-show="deletes">
        <div class="close">
          <span class="cha" @click="deletes = false">×</span>
        </div>
        <form method="post" style="width:240px; margin: 0 auto;">
          <p>确认删除该项任务吗？</p>
          <input type="button" style="margin-right: 10px;" class="button" @click="deletes = false" value="取消" />            
          <input type="button" class="button color" @click="deleteList(deleteId)" value="确定" />
        </form>
      </div>
    </transition>
    <div class="index-login" :class="{loginshow:loginshow}">
      <div class="loginBox" style="">
        <div class="login">
          <!-- <h1>登录</h1> -->
          <!-- <p class="error">登录失败</p> -->
          <div class="close">
            <span class="cha" @click="loginClose">×</span>
          </div>
          <form method="post" style="width:240px; margin: 0 auto;">
            <!--<label for="username">Username</label>:-->
            <div class="username">
              <i class="iconfont icon-denglu-copy"></i>
              <input type="text" id="user" name="username" v-model="username" placeholder="Username" autofocus="autofocus" />
            </div>
            <div class="password">
              <i class="iconfont icon-mima"></i>
            <input type="password" id="password" @keyup.enter="login" v-model="password" placeholder="Password" name="password" />
            </div>
            <input type="button" class="button" @click="login" value="登录" />
          </form>
        </div>
      </div>
    </div>
    <div class="header">
      <span v-if="showUser" class="name"><span>HI,{{this.showUser}}</span> <i class="iconfont icon-tuichu" @click="quit"></i></span>
      <span v-else  class="user" @click="loginShows">
        <!-- <i class="fa fa-user-o"></i> -->
        <i class="iconfont icon-denglu-copy"></i>
      </span>
    </div>
    <div class="logo">
      <!-- <img class="" src="../assets/images/logo.png"> -->
    </div>
    <!-- <el-dialog
      title=""
      :visible.sync="dialogVisible"
      width="1050px"
      :before-close="handleClose">
      <template>
        <el-carousel :interval="4000" height="450px">
          <el-carousel-item v-for="item in 4" :key="item">
            <img width="100%" height="100%" :src="images+'guide'+item+'.png'" alt="">
         </el-carousel-item>
        </el-carousel>
      </template>
    </el-dialog> -->
    <div :class="guide==1?'guide-search':''" class="search2 ">
      <div class="scan">
        <input type="text" v-model="ip_url" class="scanInput" placeholder="请输入IP/URL">
        <el-select v-model="scanSelect" placeholder="请选择">
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
            :disabled="item.disabled">
          </el-option>
        </el-select>
        <span class="shu">|</span>
        <!-- <i class="iconfont icon-jiance" style="font-size:32px;"></i> -->
        <img src="../assets/images/search.png" @click="search" class="search" alt="">
      </div>
      <img class="arrows" v-if="guide==1" src="../assets/images/arrowsLeft.png" width="100px" height="100px" alt="">
      <div class="guide-text" v-if="guide==1" >
        <p>
        这里输入<br>
        要检测的网站地址<br>
        进行搜索
        </p>
        <input type="button" class="button color" @click="nextStep(2)" value="下一步" />
      </div>
    </div>
    
    <div class="message">
      <p style="font-size:20px;">依据ISO27001信息安全管理体系标准 & 信息安全信息系统安全等级保护测评</p>
      <p style="font-size:12px;">According to ISO27001 information security management system standard & Evaluation of information security information system security level protection</p>
      <p style="color:#818db1;font-size:14px;margin-top:20px;">其它信息安全标准 - CCE通用配置枚举 & CVE通用漏洞与纰漏</p>
    </div>
    <div class="history">
      <a @click="getLists()">
        <span></span>
        <span></span>
        <!-- <span></span> -->
        查看列表
      </a>
    </div>
    <transition name="fade">
      <div class="guide-list" v-if="guide!=1 && guide!=0" >
        <img src="../assets/images/list.png" width="1000" alt="">
        <img class="arrows" v-if="guide!=1 && guide!=0" :class="'guide'+guide" src="../assets/images/arrowsRight.png" width="100px" height="100px" alt="">
        <div class="guide-text" :class="'guide'+guide" v-if="guide!=1 && guide!=0" >
          <p  v-if="guide==2" >
            这里查看<br>
            安全扫描报告<br>
            结果
          </p>
          <p v-if="guide==3">
            这里删除<br>
            安全扫描报告<br>
            结果
          </p>
          <p v-if="guide==4" style="margin-top:36px;">
            这里进行<br>
            漏洞利用<br>
            ...
          </p>
          <input v-if="guide!=4" type="button" class="button color" @click="nextStep()" value="下一步" />
          <input v-else type="button" class="button color" @click="guideClose()" value="关闭" />
        </div>
      </div>
    </transition>
    <div class="content" style="margin-top:45px;">
      <div v-if="scanList.length == 0 && guide==0" style="line-height:40px;">
        <div style="margin:0 auto;">暂无数据</div>
      </div>
      <div v-else style="height:100%;">
      <div class="title">
        <ul>
          <li>任务名称</li>
          <li>任务进度</li>
          <li>任务开始时间</li>
          <li>任务结束时间</li>
          <li>操作</li>
        </ul>
      </div>
      <div class="contents">
        <div style="min-height:63px;" class="lists">
          <ul v-loading="loading"  element-loading-text=" 拼命加载中 ~ ">
            <li v-for="item in scanList">
              <ul> 
                <li>{{item.ip_url}}</li>
                <li v-if="item.progress != '100.0'"><span ><i class="fa-li fa fa-spinner fa-spin"></i>{{item.progress+'%'}}</span></li>
                <li v-else><span ><i class="iconfont icon-chenggong"></i>扫描完成</span></li>
                <li>{{item.begin_time}}</li>
                <li>{{item.end_time?item.end_time:'－'}}</li>
                <li v-if="item.progress != '100.0' " >
                  <a  class="disabled"><i class="iconfont icon-yulan"></i></a>
                  <a class="disabled"><i class="iconfont icon-shanchu"></i></a>
                </li>
                <li v-else :class="[item.vul == '1'?'chenggong':(item.vul == '-1'?'shibai':'') ]" >
                  <div v-show="item.vul == '1'?true:(item.vul == '-1'?true:false)" class="biaoqian"><span>{{item.vul == '1'?'成功':(item.vul == '-1'?'失败':'') }}</span></div>
                  <div class="operation">
                    <a target="_blank" :href="item.download"  title="预览"><i class="iconfont icon-yulan"></i></a>
                    <a @click="deletes = true;deleteId=item.id"  title="删除"><i class="iconfont icon-shanchu"></i></a>
                  </div>
                  <button v-if=" item.vul != '0'  " title="利用" v-bind:disabled=" (item.vul !='' && item.vul != '-2') || utilLoading  " class="utilize" @click="utilize(item.id)" ><img v-if=" item.vul =='' || item.vul == '-2' " src="../assets/images/Attention.svg" width="22" height="22" alt=""> <img v-else src="../assets/images/Attention2.svg" width="22" height="22" alt=""> </button>
                  <button v-else disabled="true" class="utilize" ><img class="fa-spin" src="../assets/images/load.svg" width="22" height="22" alt=""></button>
                </li>
              </ul>
            </li>
          </ul>
        </div>
        <div class="block" style="margin-bottom:20px;">
          <el-pagination style="padding-left: 20px;float: left;"
            layout="prev, pager, next"
            @current-change = "handleCurrentChange"
            :page-size="size"
            :current-page="index"
            :total="total">
          </el-pagination>
        </div>
      </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "HelloWorld",
  data() {
    return {
      msg: "Welcome to Your Vue.js App",
      scanUrl: "https://scan.enncloud.cn",
      listShow: false,
      loginshow: false,
      scanloding: false,
      deletes:false,
      utilLoading:false,
      guideText:'',
      guide:0,
      dialogVisible: true,
      images:'src//assets/images/',
      deleteId:'',
      loadingText: " 玩命下发中 ~ ",
      options: [
        {
          value: 0,
          label: "深度检测"
        },
        // {
        //   value: 1,
        //   label: "自定义",
        //   disabled: true
        // }
      ],
      total: 0,
      scanSelect: 0,
      username: "",
      password: "",
      ip_url: "",
      loading: false,
      myTimeout: "",
      size: 5,
      scanList: [],
      index: 1,
      showUser: sessionStorage.getItem("username"),
      uid: sessionStorage.getItem("uid"),
      key: sessionStorage.getItem("key"),
    };
  },
  
  beforeCreate: function() {},
  created: function() {
    // this.getList();
    let _this = this; //此this指向的是vue
    if(localStorage.getItem('guide')){
      this.guide = 0;
    }else{
      this.guide = 1;
    }
    // window.onmousewheel = function (e) {
    //   if(e.wheelDelta<0 && !_this.listShow){
    //     _this.listShow = true;
    //     _this.getList();
    //   }
    // }
  },
  methods: {
    guideClose:function(){
      this.listShow = false;
      this.guide = 0;
      localStorage.setItem('guide',1)
    },
    nextStep:function(num){
      console.log(this.guide,num,'----nextStep')
      if(num == 2){
        this.guide = num;
        this.listShow = true;
      }else{
        this.guide = this.guide+1
      }
    },
    search: function() {
      const _this = this;
      if (sessionStorage.getItem("uid")) {
        if (this.ip_url != "") {
          this.loadingText = " 玩命下发中 ~ ";
          this.scanloding = true;
          console.log(this.key,'------key')
          axios
            .post(this.scanUrl + "/scan", {
              uid: this.uid,
              key: this.key,
              ip_url: this.ip_url
            })
            .then(
              function(data) {
                if (data.data.code == 200) {
                  _this.ip_url = "";
                  _this.loading = true;
                  _this.index = 1;
                  _this.scanloding = false;
                  _this.getList();
                } else if(data.data.code == -1){
                  _this.scanloding = false;
                  _this.$message({
                    message: "登录失败,请重新登录。",
                    type: "warning"
                  });
                  setTimeout(()=>{
                    _this.quit();
                  },1000)
                } else {
                  _this.scanloding = false;
                  _this.$message({
                    message: "下发任务失败",
                    type: "warning"
                  });
                }
              },
              function(error) {}
            );
        }
      } else {
        _this.$message({
          message: "请登录之后进行操作。",
          type: "warning"
        });
      }
    },
    getLists: function() {
      if (sessionStorage.getItem("uid")) {
        this.scanloding = true;
        this.loadingText = " 拼命加载中 ~ ";
        this.getList();
      } else {
        this.$message({
          message: "请登录之后进行操作。",
          type: "warning"
        });
      }
    },
    utilize: function(id) {
      // this.utilLoading = true;
      const _this = this;
      axios.post(this.scanUrl + "/vul", {
          id : id
        })
        .then(function(data) {
          console.log(data,'--------data')
          if (data.data.code == 200) {
            _this.getList();
            // this.utilLoading = false;
          } else {
            _this.$message({
              message: data.data.error,
              type: "error"
            });
          }
        });
    },
    deleteList: function(id) {
      const _this = this;
      axios.post(_this.scanUrl + "/delete", { id: id }).then(function(data) {
        if (data.data.code == 200) {
          _this.getList();
          _this.deletes = false;
        }else{
          _this.$message({
              message: data.data.error,
              type: "error"
            });
        }
      });
    },
    getList: function() {
      const _this = this;
      if (sessionStorage.getItem("uid")) {
        axios
          .post(_this.scanUrl + "/list", {
            uid: _this.uid,
            key: _this.key,
            size: _this.size,
            index: _this.index-1
          })
          .then(function(data) {
            if (data.data.code == 200) {
              _this.scanList = data.data.list;
              _this.scanloding = false;
              _this.listShow = true;
              _this.loading = false;
              _this.total = data.data.total;
              _this.myTimeout = setTimeout(function() {
                _this.getList();
                console.log("wahaha", new Date());
                clearTimeout(_this.myTimeout);
              }, 3000);
            }else if(data.data.code == -1){
              _this.scanloding = false;
              _this.listShow = false;
              _this.$message({
                message: "登录失败,请重新登录。",
                type: "warning"
              });
              setTimeout(()=>{
                _this.quit();
              },1000)
            } else {
              _this.$message({
                message: data.data.error,
                type: "error"
              });
              _this.scanloding = false;
              _this.listShow = false;
            }
          });
      } else {
        _this.$message({
          message: "请登录之后进行操作。",
          type: "warning"
        });
      }
    },
    handleCurrentChange: function(val) {
      console.log(val);
      this.index = val;
      this.loading = true;
      this.getList();
    },
    loginShows: function() {
      this.loginshow = true;
    },
    loginClose: function() {
      this.loginshow = false;
    },
    quit: function() {
      this.showUser = false;
      sessionStorage.setItem("uid", "");
      sessionStorage.setItem("key", "");
      sessionStorage.setItem("username", "");
      window.location.reload();
    },
    login: function() {
      const _this = this;
      const data = { uid: this.username, passwd: this.password };
      axios.post(this.scanUrl + "/login", data).then(userdata => {
        if (userdata.data.code == 200) {
          _this.loginshow = false;
          _this.username = "";
          _this.password = "";
          _this.showUser = userdata.data.cn;
          _this.uid = userdata.data.uid;
          _this.key = userdata.data.key
          sessionStorage.setItem("uid", userdata.data.uid);
          sessionStorage.setItem("username", userdata.data.cn);
          sessionStorage.setItem("key", userdata.data.key);
        } else {
          _this.$message({
            message: "登录失败,请重新登录。",
            type: "warning"
          });
        }
      });
    }
  }
};
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="less">
.fade-enter-active, .fade-leave-active {
          transition: opacity 1s
        }
        .fade-enter, .fade-leave-active {
          opacity: 0
        }
.el-carousel__item h3 {
    color: #475669;
    font-size: 14px;
    opacity: 0.75;
    line-height: 200px;
    margin: 0;
  }
  .guide-list{
    position: absolute;
    z-index:100;
    top:205px;
    left: 50%;
    margin-left: -500px;
    transition: all 1s;
    -moz-transition: all 1s; /* Firefox 4 */
    -webkit-transition: all 1s; /* Safari 和 Chrome */
    -o-transition: all 1s; /* Opera */
    .arrows{
      position: absolute;
      right:85px;
      transition: all 1s;
      -moz-transition: all 1s; /* Firefox 4 */
      -webkit-transition: all 1s; /* Safari 和 Chrome */
      -o-transition: all 1s; /* Opera */
    }
    .guide-text{
      right:180px;
      top:110px;
      transition: all 1s;
      -moz-transition: all 1s; /* Firefox 4 */
      -webkit-transition: all 1s; /* Safari 和 Chrome */
      -o-transition: all 1s; /* Opera */
    }
    .button {
      height: 35px;
      width:120px;
      background: #f3f4f8;
      margin: 0 auto;
      border:0;
      border-radius: 20px;
      color: #9598a1;
      text-align: center;
      font-weight: bold;
      cursor: pointer;
      
    }
    .color{
      color:#ffffff;
      background: -webkit-linear-gradient(
        left,
        #c36aea,
        #407cea
      ); /* Safari 5.1 - 6.0 */
      background: -o-linear-gradient(
        right,
        #c36aea,
        #407cea
      ); /* Opera 11.1 - 12.0 */
      background: -moz-linear-gradient(
        right,
        #c36aea,
        #407cea
      ); /* Firefox 3.6 - 15 */
      background: linear-gradient(to right, #c36aea, #407cea); /* 标准的语法（必须放在最后） */
    }
  }
  .el-carousel__item:nth-child(2n) {
    background-color: #99a9bf;
  }
  
  .el-carousel__item:nth-child(2n+1) {
    background-color: #d3dce6;
  }
.arrows.guide3{
  right: 50px;
}
.arrows.guide4{
  right: 12px;
}
.guide-text.guide3{
  right: 145px;
}
.guide-text.guide4{
  right: 110px;
}

.el-message {
  border-radius: 20px;
  height: 20px;
  text-align: center;
  background: #ffffff;
  border: 0;
}input[type="button"]
,button{
  list-style: none;
  outline: none;
}
.el-message--warning .el-message__content {
  font-size: 12px;
  font-weight: normal;
}
@bg : #253258;
@size : #a0b0dc;
h1,
h2 {
  font-weight: normal;
}
.shibai{
  .biaoqian{
    position: absolute;
    z-index:10;height:110%;
    width:20px;
    background: #fe6778;
    transform: rotate(-50deg);
    top: -20px;
    right: 8px;
    span{
      color:#ffffff;
      font-size:12px;
      transform: rotate(90deg);
      width:30px;
      height: 50px;
      font-weight: bold;
      margin:0 10px -8px 0!important;
      padding:0!important;
    }
  }
}
.chenggong{
  .biaoqian{
    position: absolute;
    z-index:10;height:100%;
    width:20px;
    background: #33b867;
    transform: rotate(-50deg);
    top: -20px;
    right: 8px;
    span{
      color:#ffffff;
      font-size:12px;
      transform: rotate(90deg);
      width:30px;
      height: 50px;
      font-weight: bold;
      margin:0 10px -8px 0!important;
      padding:0!important;
    }
  }
}
.bg{
  width:100%;
  height: 100%;
  position: absolute;
  z-index:8;
  background: rgba(0,0,0,0.5)
}
.hello {
  height: 100%;
}
.logo {
  overflow: hidden;
  height: 120px;
  margin-top: 100px;
  padding-left: 80px;
}
.header {
  overflow: hidden;
  padding-right: 25px;
  padding-top: 15px;
  padding-bottom: 10px;
}
.user {
  display: inline-block;
  border: 1px solid #b5c1e5;
  border-radius: 17px;
  height: 28px;
  width: 28px;
  float: right;
  i.icon-denglu-copy {
    font-size: 16px;
    color: #b5c1e5;
    line-height: 30px;
    cursor: pointer;
  }
}
.operation{
  height: 24px;line-height: 24px;
  display: inline-block;
}
.utilize{
  position: absolute;
  width:50px;
  text-align: left;
  cursor: pointer;
  padding-left:5px;
  background: #ffffff;
  // color:#28314c;
  margin-left:6px;padding-left:8px;
  border:0px;
  font-size:13px;
  border-left:1px solid #ccc;
  height: 22px;line-height: 22px;
  top:16px;
}
.name {
  display: inline-block;
  border-radius: 17px;
  height: 30px;
  width: auto;
  overflow: hidden;
  float: right;
  i.icon-tuichu {
    font-size: 16px;
    color: #b5c1e5;
    float: right;
    line-height: 30px;
    cursor: pointer;
  }
  span {
    color: @size;
    font-size: 14px;
    float: left;
    line-height: 30px;
    margin-right: 10px;
  }
}
.shu {
  float: left;
  color: rgb(42, 65, 118);
  font-weight: 500;
  font-size: 20px;
  line-height: 50px;
}
.guide-search{
  border:1px dashed #ffffff;
  border-radius:8px;
  background: #414f77;
  z-index:100;
  .arrows{
    position: absolute;
    top:110px;
  }
  .button {
    height: 35px;
    width:120px;
    background: #f3f4f8;
    margin: 0 auto;
    border:0;
    border-radius: 20px;
    color: #9598a1;
    text-align: center;
    font-weight: bold;
    cursor: pointer;
    
  }
  .color{
    color:#ffffff;
    background: -webkit-linear-gradient(
      left,
      #c36aea,
      #407cea
    ); /* Safari 5.1 - 6.0 */
    background: -o-linear-gradient(
      right,
      #c36aea,
      #407cea
    ); /* Opera 11.1 - 12.0 */
    background: -moz-linear-gradient(
      right,
      #c36aea,
      #407cea
    ); /* Firefox 3.6 - 15 */
    background: linear-gradient(to right, #c36aea, #407cea); /* 标准的语法（必须放在最后） */
  }
}
.guide-text{
  width:150px;
  height:150px;
  border-radius:80px;
  background: #ffffff;
  position: absolute;
  font-family: Hanzipen SC;
  top:120px;
  text-align: center;line-height: 24px;
  right:105px;
  p{
    margin-top:36px;
    margin-bottom: 10px;
  }
}
.search2{
  position: relative;
  width:700px;
  padding:24px 22px;
  margin: 0 auto;
}
.scan {
  overflow: hidden;
  width: 700px;
  height: 50px;
  border: 0;
  background-color: @bg;
  border-radius: 25px;
  outline: none;
  color: @size;
  line-height: 50px;
  font-size: 14px;
  // margin: 0 auto;
  // margin-top: 30px;
  .scanInput {
    background-color: @bg;
    width: 510px;
    height: 50px;
    line-height: 50px;
    border: 0;
    float: left;
    outline: none;
    color: @size;
    box-sizing: border-box;
    padding: 0 20px;
    font-size: 14px;
  }
  .scanInput::-webkit-input-placeholder {
    color: @size;
  }
  .scanInput:-moz-placeholder {
    color: @size;
  }
  .scanInput::-moz-placeholder {
    color: @size;
  }
  .scanInput:-ms-input-placeholder {
    color: @size;
  }
  .search {
    vertical-align: middle;
    cursor: pointer;
  }
  .user {
    cursor: pointer;
  }
  .el-select {
    float: left;
    width: 110px;
    border: 0;
    .el-input__inner {
      background-color: @bg;
      border: 0;
      color: @size;
      font-weight: bold;
    }

    .el-select__caret {
      color: #407ded;
      font-size: 18px;
    }

    .el-icon-arrow-up:before {
      content: "\E60C";
    }
  }
}
.el-message-box {
  border-radius: 15px;
}
.el-message-box__message {
  color: @bg;
}
.message {
  width: 700px;
  margin: 0 auto;
  margin-top: 20px;
  p {
    color: #bfcbef;
    font-family: "Ya Hei";
    text-align: left;
    margin: 0;
    margin-top: 10px;
  }
}
.el-popper .popper__arrow {
  display: none;
}
.el-popper[x-placement^="bottom"] {
  margin-top: 5px;
}
.el-select-dropdown {
  background-color: @bg;
  color: #ffffff;
  color: #fff;
  border: 0;
  .el-select-dropdown__item {
    color: @size;
  }
  .el-select-dropdown__item.hover,
  .el-select-dropdown__item:hover {
    background-color: @bg;
  }
  .el-select-dropdown__item.selected {
    color: #409eff;
  }
}
.content {
  background-color: #e8ecf7;
  height: 0;
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  min-width: 1100px;
  overflow: hidden;
  .title {
    width: 1100px;
    margin: 0 auto;
    ul {
      overflow: hidden;
      margin: 0;
      padding: 0 20px;
      li {
        list-style: none;
        float: left;
        width: 20%;
        text-align: center;
        line-height: 50px;
        font-size: 14px;
        color: #9298ab;
      }
    }
  }
  .contents {
    width: 1100px;
    margin: 0 auto;
    overflow-y: auto;
    > .lists ul {
      margin: 0;
      padding: 0 20px;
      > li {
        height: 60px;
        line-height: 60px;
        list-style: none;
        position: relative;
        background-color: #ffffff;
        margin-bottom: 20px;
        border-radius: 5px;

        > ul {
          overflow: hidden;
          padding: 0;
          height: 60px;
          border-radius: 5px;
          &:hover {
            box-shadow: 0px 8px 18px -3px #a0b0dc;
            border-left: 4px solid #407ded;
          }
          li {
            float: left;
            width: 20%;
            list-style: none;
            font-size: 14px;
            color: #28314c;
            overflow: hidden;
            span {
              display: inline-block;
              vertical-align: middle;
              position: relative;
              margin-left: 30px;
              .icon-chenggong {
                color: #71d4a9;
              }
              .fa-li {
                left: -2em;
                top: 0.2em;
              }
              i.fa-spinner {
                font-size: 26px;
                margin-top: 10px;
                color: #407ded;
              }
              .icon-chenggong {
                color: #71d4a9;
                font-size: 26px;
                position: absolute;
                left: -34px;
              }
            }
            a {
              cursor: pointer;
              list-style: none;
              outline: none;
              text-decoration: none;
            }
            .icon-yulan,
            .icon-shanchu {
              font-size: 20px;
              margin: 0 5px;
              color: #407ded;
            }
            .disabled {
              i {
                color: @size;
              }
            }
          }
        }
      }
    }
  }
}

.listShow {
  .logo {
    height: 0;
    margin-top: 0px;
    opacity: 0;
    transition: all 1s;
    -moz-transition: all 1s; /* Firefox 4 */
    -webkit-transition: all 1s; /* Safari 和 Chrome */
    -o-transition: all 1s; /* Opera */
  }
  .message {
    opacity: 0;
    transition: all 1s;
    -moz-transition: all 1s; /* Firefox 4 */
    -webkit-transition: all 1s; /* Safari 和 Chrome */
    -o-transition: all 1s; /* Opera */
  }
  .scan {
    margin-top: 0;
    transition: all 1s;
    -moz-transition: all 1s; /* Firefox 4 */
    -webkit-transition: all 1s; /* Safari 和 Chrome */
    -o-transition: all 1s; /* Opera */
  }
}
.listShow {
  .content {
    height: calc(~"100% - 150px");
    transition: all 1s;
    -moz-transition: all 1s; /* Firefox 4 */
    -webkit-transition: all 1s; /* Safari 和 Chrome */
    -o-transition: all 1s; /* Opera */
  }
  .contents {
    height: calc(~"100% - 70px");
    padding-top: 20px;
  }
}
.index-login {
  -webkit-transition: all 1s linear;
  -moz-transition: all 1s linear;
  -ms-transition: all 1s linear;
  -o-transition: all 1s linear;
  transition: all 1s linear;
  z-index: 100;
  opacity: 0;
  display: table;
  width: 100%;
  margin: 0 auto;
  position: fixed;
  height: 100%;
  visibility: hidden;
  background: #303855b3;
}

.loginshow {
  -webkit-transition: all 1s linear;
  -moz-transition: all 1s linear;
  -ms-transition: all 1s linear;
  -o-transition: all 1s linear;
  transition: all 1s linear;
  opacity: 1;
  visibility: visible;
}

.loginBox {
  display: table-cell; /*让元素以表格的单元素格形式渲染*/
  vertical-align: middle; /*使用元素的垂直对齐*/
}

.delete {
  width: 262px;
  height: 146px;
  overflow: hidden;
  background-color: #ffffff;
  border-radius: 10px;
  margin: 0 auto;
  font-size:12px;
  position: absolute;
  z-index:10;
  left: 50%;margin-left:-131px;
  top:50%;margin-top:-73px;
  .close {
    color: #000;
    font-size: 16px;
    display: block;
    width: 100%;
    text-align: right;
    box-sizing: border-box;
    padding: 15px;
    padding-bottom:0;
    margin-bottom: 10px;
    .cha {
      cursor: pointer;
      width: 30px;
      height: 30px;
    }
  }
  p{
    padding-bottom: 15px;
  }
  form .button {
    height: 30px;
    width: 35%;
    background: #f3f4f8;
    margin: 0 auto;
    border:0;
    border-radius: 20px;
    color: #9598a1;
    text-align: center;
    font-weight: bold;
    cursor: pointer;
    
  }
  form .color{
    color:#ffffff;
    background: -webkit-linear-gradient(
      left,
      #c36aea,
      #407cea
    ); /* Safari 5.1 - 6.0 */
    background: -o-linear-gradient(
      right,
      #c36aea,
      #407cea
    ); /* Opera 11.1 - 12.0 */
    background: -moz-linear-gradient(
      right,
      #c36aea,
      #407cea
    ); /* Firefox 3.6 - 15 */
    background: linear-gradient(to right, #c36aea, #407cea); /* 标准的语法（必须放在最后） */
  }
}

.login {
  width: 362px;
  height: 266px;
  overflow: hidden;
  background-color: #ffffff;
  border-radius: 20px;
  margin: 0 auto;
  form input {
    width: calc(~"100% - 40px");
    height: 30px;
    box-sizing: content-box;
    border: 0;
    color: #000000;
    text-align: left;
    padding: 0 5px;
    box-sizing: border-box;
    padding: 0 5px;
    outline: none;
  }
  form input[type="text"],
  form input[type="password"] {
    background: #f3f4f8;
    float: left;
  }
  .password,
  .username {
    i {
      width: 40px;
      float: left;
      line-height: 30px;
      color: #ffffff;
    }
    border-radius: 20px;
    overflow: hidden;
    margin-bottom: 20px;
    background: #bfccef; /* Old browsers */
    background: -moz-linear-gradient(
      left,
      #bfccef 15%,
      #f3f4f8 15%
    ); /* FF3.6-15 */
    background: -webkit-linear-gradient(
      left,
      #bfccef 15%,
      #f3f4f8 15%
    ); /* Chrome10-25,Safari5.1-6 */
    background: linear-gradient(
      to right,
      #bfccef 15%,
      #f3f4f8 15%
    ); /* W3C, IE10+, FF16+, Chrome26+, Opera12+, Safari7+ */
    filter: progid:DXImageTransform.Microsoft.gradient(
        startColorstr="#bfccef",
        endColorstr="#f3f4f8",
        GradientType=1
      ); /* IE6-9 */
  }
  .password {
    margin-bottom: 30px;
  }
  input::-ms-input-placeholder {
    text-align: left;
    color: #bbb;
  }
  input::-webkit-input-placeholder {
    text-align: left;
    color: #bbb;
  }
  form .button {
    height: 30px;
    width: 100%;
    margin: 0 auto;
    border-radius: 20px;
    color: #ffffff;
    text-align: center;
    font-weight: bold;
    cursor: pointer;
    background: -webkit-linear-gradient(
      left,
      #c36aea,
      #407cea
    ); /* Safari 5.1 - 6.0 */
    background: -o-linear-gradient(
      right,
      #c36aea,
      #407cea
    ); /* Opera 11.1 - 12.0 */
    background: -moz-linear-gradient(
      right,
      #c36aea,
      #407cea
    ); /* Firefox 3.6 - 15 */
    background: linear-gradient(to right, #c36aea, #407cea); /* 标准的语法（必须放在最后） */
  }
}
.login h1 {
  color: #ffffff;
}
.close {
  color: #000;
  font-size: 16px;
  display: block;
  width: 100%;
  text-align: right;
  box-sizing: border-box;
  padding: 15px;
  margin-bottom: 10px;
  .cha {
    cursor: pointer;
    width: 30px;
    height: 30px;
  }
}
.history {
  position: fixed;
  cursor: pointer;
  bottom: 50px;
  left: 50%;
  margin-left: -28px;
  // top: 200px;
  a {
    padding-top: 60px;
    font-size: 14px;
    position: relative;
    color: @size;
    text-decoration: none;
  }
  a span {
    position: absolute;
    top: 0;
    left: 50%;
    width: 24px;
    height: 24px;
    margin-left: -12px;
    border-left: 1px solid #fff;
    border-bottom: 1px solid #fff;
    -webkit-transform: rotate(-45deg);
    transform: rotate(-45deg);
    -webkit-animation: sdb07 2s infinite;
    animation: sdb07 2s infinite;
    opacity: 0;
    box-sizing: border-box;
  }
  a span:nth-of-type(1) {
    -webkit-animation-delay: 0s;
    animation-delay: 0s;
  }
  a span:nth-of-type(2) {
    top: 16px;
    -webkit-animation-delay: 0.15s;
    animation-delay: 0.15s;
  }
  a span:nth-of-type(3) {
    top: 32px;
    -webkit-animation-delay: 0.3s;
    animation-delay: 0.3s;
  }
  @-webkit-keyframes sdb07 {
    0% {
      opacity: 0;
    }
    50% {
      opacity: 1;
    }
    100% {
      opacity: 0;
    }
  }
  @keyframes sdb07 {
    0% {
      opacity: 0;
    }
    50% {
      opacity: 1;
    }
    100% {
      opacity: 0;
    }
  }
}
</style>
