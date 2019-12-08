<template>
  <div style="height:100%;">
    <div class="jy-type" v-if="nowFlag">
      <div class="jy-type-item">
        <div class="jy-type-item-name" @click="gotoStation('1')">司机端</div>
      </div>
      <div class="jy-type-item">
        <div class="jy-type-item-name" @click="gotoStation('2')">油站端</div>
      </div>
      <div class="jy-type-item">
        <div class="jy-type-item-name" @click="gotoStation('3')">公司端</div>
      </div>
    </div>
    <div class="jy-login" v-else>
      <div class="jy-log">
        <div class="jy-log-h2">登录</div>
        <div class="jy-log-block"></div>
        <span class="jy-log-span" v-if="logType == 'pw'">当前为密码登录方式</span>
        <span class="jy-log-span" v-else>
          当前为验证码登录方式
          <!-- <span v-if="!updatePassword" class="jy-log-span" style="display:block;">(首次登录必须用验证码登录奥~)</span> -->
        </span>
      </div>
      <div class="login_form">
        <!-- <input type="text" class="qxs-ic_user qxs-icon" placeholder="请输入手机号" v-model="userName">
      <input type="text" class="qxs-ic_check qxs-icon" placeholder="请输入验证码" v-model="checkNumber" /><span class="jy-getcode" @click="getCheckCode">{{checkCodeContent}}</span>
      <input type="text" class="qxs-ic_password qxs-icon" placeholder="请输入密码" v-model="password"> -->
        <input type="text" class="qxs-icon" :placeholder="nameholder" v-model="userName">
        <input type="text" class="qxs-icon" id="password" placeholder="请输入密码" v-model="password" v-if="logType == 'pw'" />
        <div style="position:relative;" v-else>
          <input type="text" class="qxs-icon" placeholder="验证码" v-model="checkNumber" /><span class="jy-getcode" @click="getCheckCode">{{checkCodeContent}}</span>
        </div>
        <el-button class="login_btn" @click.native="login" type="primary" :loading="isBtnLoading">登 录</el-button>
        <div v-if="station != '3'">
          <div class="jy-log-change" v-if="logType == 'pw'" @click="changeLogType">使用验证码登录</div>
          <div class="jy-log-change" v-else @click="changeLogType">使用密码登录</div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
//  import { userLogin } from '../../api/api';
function changePassword(that,id,origin) {
  var str = ""; //str变量用于存储密码
  //获取input元素
  // debugger;
  var input = document.getElementById(id);
  //获取span元素
  // var span = document.getElementById("span")[0];
  input.oninput = function(evt) {
    var val = this.value; //取到输入框的值
    if (val.length > str.length) {
      //输入值
      str += val.charAt(val.length - 1);
    } else {
      //回删值
      str = str.substr(0, val.length);
    }
    //将输入框除最后一位的字符替换成*
    this.value =
      val.substr(0, val.length - 1).replace(/./g, "*") +
      val.charAt(val.length - 1);
    /* 取当前输入框长度，用于判断是否正在输入
    停止输入时，一秒后将最后一个字符变成*号 */
    var len = this.value.length;
    setTimeout(() => {
      /*这时的this指向在延迟器触发时输入框的状态，
    而不是延迟器创建时的状态*/
      if (this.value.length == len) {
        //一秒后输入框的值长度不变，将所有字符替换为*
        this.value = this.value.replace(/./g, "*");
      }
    }, 1000);
    that[origin] = str;
    // span.innerHTML = str;
  };
}
import * as utils from '../../utils';
export default {
  data() {
    return {
      userName: '',
      password: '',
      password2:'',
      checkNumber: '',
      checkCodeContent: '获取验证码',
      isBtnLoading: false,
      logType: 'yzm',
      nowFlag: true,
      updatePassword:0,
      nameholder:"请输入手机号"
    }
  },
  created() {
    if (localStorage.getItem('updatePassword')) {
      this.updatePassword = localStorage.getItem('updatePassword');
    }
    document.title = "登录通达加油宝";
    if (window.location.href.indexOf('from=')>-1) {
      var urlUserType = window.location.href.split('?from=')[1].split('#/')[0];
      var tmpUserInfo = localStorage.getItem('userInfo') && JSON.parse(localStorage.getItem('userInfo'));
      // debugger
      if (tmpUserInfo && tmpUserInfo.userType===urlUserType) {
        window.logoutFlag = false;
        this.$router.push({
          name: 'index',
          params: {
            userType: JSON.parse(localStorage.getItem('userInfo')).userType
          }
        });
      }else {
        this.gotoStation(urlUserType,'null');
      }
    }else {
      alert('访问地址配置有误，请联系管理员~')
    }
    this.$nextTick(() => {
      // debugger;
      if(this.logType == 'pw'){
        changePassword(this,'password','password2');
      }
    })
  },
  watch:{
    logType(val){
      // debugger;
      this.$nextTick(() => {
        if(val == 'pw'){
          changePassword(this,'password','password2');
        }
      })
    }
  },
  computed: {
    btnText() {
      if (this.isBtnLoading) return '登录中...';
      return '登录';
    }
  },
  methods: {
    login() {
      if (!this.userName) {
        Toast('请输入用户名');
        return;
      }
      if (this.logType == 'pw') {
        if (!this.password) {
          Toast('请输入密码');
          return;
        }
      } else {
        if (!this.checkNumber) {
          Toast('请输入验证码');
          return;
        }
      }
      // if (!this.userName) {
      //   Toast('请输入用户名');
      //   return;
      // }
      this.isBtnLoading = true;
      utils.Post('login', {
        "userPhone": this.userName,
        "userPwd": this.password2,
        "icodeNumber": this.checkNumber,
        "userType": this.station
      }).then((response) => {
        let data = response.data;
        if (data.code === 0) {
          //验证微信的jssdk
          // this.checkWxCode();
          data.result.userType = this.station;
          localStorage.setItem('updatePassword',1);
          window.logoutFlag = false;
          localStorage.userInfo = JSON.stringify(data.result);
          window.userInfo = data.result;
          if (data.result.updatePassword == 0 && this.station !='3') {
            // Toast('登录成功！');
            this.$router.push({
              name: 'editpw',
              params: {
                notFirstLog: false,
                userType: this.station
              }
            });
          } else {
            Toast('登录成功！');
            this.$router.push({
              name: 'index',
              params: {
                userType: this.station
              }
            });
          }
        } else {
          Toast(data.msg || '登录失败！');
        }
        this.isBtnLoading = false;
      }).catch((error) => {
        Toast('登录失败！');
        this.isBtnLoading = false;
      })
    },
    getCheckCode() {
      var self = this;
      var totalSeconds = 60;
      if (self.checkCodeContent == '获取验证码') {
        utils.Get('getIcodeNumber', {
          userPhone: this.userName
        }).then((response) => {
          let data = response.data;
          if (data.code === 0) {

          } else {
            Toast('获取验证码失败！');
          }
        }).catch((error) => {
          Toast('获取验证码失败！');
        })
      } else {
        return;
      }
      self.checkCodeContent = totalSeconds + 's';
      var handleInterval = window.setInterval(function() {
        totalSeconds--;
        self.checkCodeContent = totalSeconds + 's';
        // console.log(totalSeconds +'s')
        if (totalSeconds == 0) {
          window.clearInterval(handleInterval);
          self.checkCodeContent = '获取验证码';
          // console.log('取消')
        }
      }, 1000);
    },
    changeLogType() {
      if (this.logType && this.logType === 'pw') {
        this.logType = '';
      } else {
        this.logType = 'pw';
      }
      this.userName='';
      this.password='';
      this.checkNumber='';
    },
    gotoStation(type,isnull) {
      var tmpUserInfo = {};
      if (localStorage.getItem('userInfo')) {
        tmpUserInfo = JSON.parse(localStorage.getItem('userInfo'));
      }

      this.nowFlag = false;
      this.station = type;
      if (type === '3') {
        this.logType = 'pw';
        this.nameholder = '请输入登录名';
        if (isnull == 'null') {
          this.userName = '';
        } else {
          this.userName = tmpUserInfo.userName;
        }
      }else {
        this.logType = '';
        this.nameholder = '请输入手机号';
        if (isnull == 'null') {
          this.userName = '';
        } else {
          this.userName = tmpUserInfo.userPhone;
        }
        // this.userName = tmpUserInfo.userPhone;
      }
    },
    checkWxCode() {
      utils.Get('getWechatSign', {
        signUrl: window.location.href.split('#/')[0]
      }).then((response) => {
        let data = response.data;
        if (data.code === 0) {
          var jsApiList = [
            'onMenuShareTimeline',
            'onMenuShareAppMessage',
            'onMenuShareQQ',
            'onMenuShareWeibo',
            'onMenuShareQZone',
            'startRecord',
            'stopRecord',
            'onVoiceRecordEnd',
            'playVoice',
            'pauseVoice',
            'stopVoice',
            'onVoicePlayEnd',
            'uploadVoice',
            'downloadVoice',
            'chooseImage',
            'previewImage',
            'uploadImage',
            'downloadImage',
            'translateVoice',
            'getNetworkType',
            'openLocation',
            'getLocation',
            'hideOptionMenu',
            'showOptionMenu',
            'hideMenuItems',
            'showMenuItems',
            'hideAllNonBaseMenuItem',
            'showAllNonBaseMenuItem',
            'closeWindow',
            'scanQRCode',
            'chooseWXPay',
            'openProductSpecificView',
            'addCard',
            'chooseCard',
            'openCard',
            'previewFile',
            'updateAppMessageShareData',
            'updateTimelineShareData'
          ];
          if (/iphone/.test(navigator.userAgent.toLowerCase())) {
            jsApiList.push('getLocalImgData');
          };
          var config = {
            debug: false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
            appId: data.result.appid, // 必填，公众号的唯一标识
            timestamp: data.result.timestamp, // 必填，生成签名的时间戳
            nonceStr: data.result.nonceStr, // 必填，生成签名的随机串
            signature: data.result.signature, // 必填，签名
            jsApiList: jsApiList // 必填，需要使用的JS接口列表
          };

          // console.log('远端文件地址https://res.wx.qq.com')
          // let url = 'https://res.wx.qq.com/open/js/jweixin-1.2.0.js'; // 远端文件地址
          // var script = document.createElement("script");
          // script.src = url;
          // document.head.appendChild(script);
          // script.onload = () => {
          // debugger;
          if (wx) {
            wx.config(config);
            wx.ready((res) => {
              console.log('微信jssdk验证成功', res);
            });
            wx.error((res1) => {
              console.log('微信jssdk验证错误', res1);
            })
          } else {
            alert('无wx对象!请重试')
          }
          // };
        } else {
          Toast(data.msg || '微信验证权限失败！');
        }
      }).catch((error) => {
        Toast('微信验证权限失败！');
      })
    }
  }
}

</script>
<style>
.jy-log {
  position: relative;
  margin: 58px 0 0 35px;
}

.jy-log-h2 {

  height: 40px;
  font-size: 28px;
  font-family: PingFang-SC;
  font-weight: bold;
  color: rgba(34, 36, 42, 1);
  line-height: 40px;
  padding-left: 2px;
}

.jy-log-span {

  /*height:21px;*/
  font-size: 15px;
  font-family: PingFangSC;
  font-weight: 300;
  color: rgba(102, 102, 102, 1);
  /*line-height:21px;*/
  padding-top: 12px;
  display: inline-block;
}

.jy-log-block {
  width: 59px;
  height: 9px;
  background: rgba(255, 197, 0, 1);
  border-radius: 2px;
  position: absolute;
  top: 32px;
  z-index: -1;
}

.login_form {
  padding-top: 10%;
  padding-left: 10%;
  padding-right: 10%;
}

.login_form input {
  height: 50px;
  margin-top: 14px;
  width: 100%;
}

.login_form ::-moz-placeholder {
  color: #999;
}

.login_form ::-webkit-input-placeholder {
  color: #999;
}

.login_form :-ms-input-placeholder {
  color: #999;
}

input.qxs-ic_check {
  width: 60%;
  border-radius: 6px 0 0 6px;
}

.qxs-ic_user {
  background: url("../../assets/login/ic_user.png") no-repeat;
  background-size: 13px 15px;
  background-position: 3%;
}

.qxs-ic_check {
  background: url("../../assets/login/check.png") no-repeat;
  background-size: 13px 15px;
  background-position: 3%;
}

.jy-type {
  height: 100%;
  background: url('../../assets/bg_1.png') no-repeat top center;
  background-size: cover;
  padding-top: 20px;
}

.jy-type-item {
  padding-top: 50px;
}

.qxs-ic_password {
  background: url("../../assets/login/ic_password.png") no-repeat;
  background-size: 13px 15px;
  background-position: 3%;
  margin-bottom: 20px;
}

.login_logo {
  height: 100%;
  width: 100%;
}


.login_btn.el-button {
  margin-top: 66px;
  /*width: 279px;*/
  width: 100%;
  height: 48px;
  background: linear-gradient(282deg, rgba(255, 193, 0, 1) 0%, rgba(255, 215, 0, 1) 100%);
  border-radius: 24px;
  border-color: #fff;
}

.login_btn.el-button span {
  /*width:42px;*/
  height: 25px;
  font-size: 18px;
  font-family: PingFangSC;
  font-weight: 500;
  color: rgba(58, 52, 32, 1);
  line-height: 25px;
}

.jy-getcode {
  /*display: inline-block;
  width: 40%;
  height: 55px;
  margin-top: 14px;
  line-height: 55px;
  text-align: center;
  vertical-align: top;
  border-top: solid 1px lavender;
  border-bottom: solid 1px lavender;
  border-right: solid 1px lavender;
  border-radius: 0 6px 6px 0;*/
  position: absolute;
  top: 28px;
  right: 0px;
  width: 80px;
  height: 22px;
  font-size: 16px;
  font-family: PingFangSC;
  font-weight: 300;
  color: rgba(255, 197, 0, 1);
  line-height: 22px;
  text-align: center;
}

.jy-log-change {
  width: 98px;
  height: 20px;
  font-size: 14px;
  font-family: PingFangSC;
  font-weight: 300;
  color: rgba(153, 153, 153, 1);
  line-height: 20px;
  margin: 16px auto 0;
}

.jy-type-item-name {
  width: 108px;
  height: 108px;
  margin: 0px auto 0;
  line-height: 100px;
  text-align: center;
  border-radius: 54px;
  background: rgba(255, 255, 255, .15);
  border: 3px solid rgba(255, 255, 255, 1);

  font-size: 18px;
  font-family: PingFangSC;
  /*font-weight:600;*/
  color: rgba(255, 255, 255, 1);
  /*border:3px solid rgba(0,0,0,1);*/
}

</style>
