<template>
  <div class="hello">
    <div class="discuss">
      <ul class="video-sms-list">
        <li v-for="item in chatItem" class="chatLi">
          <div class="video-sms-pane">
              <div class="video-sms-text">
                  <span class="uniqueId">{{item.uniqueId}}:</span>
                  <span v-for="textEmoji in item.elems">
                    {{textEmoji.content.text}}
                    <img :src="textEmoji.content.index | emojiFilter" alt="" />
                  </span>
              </div>
          </div>
        </li>        
      </ul>
    </div>
    <!-- 聊天输入 -->
    <div class="chat-main">
      <div class="chat_btn" v-show="chatBtnShow">
        <button class="chat_input" @click="chatClick"></button>
        <button class="chat_like"></button>
      </div>
      <div class="chat_box" v-show="chatBoxShow">
        <div class="chat_box_btn">
          <input type="text" ref="chatInput"/>
          <span @click="emojiClick">表情</span>
          <button @click="onSendMsg">发送</button>
        </div>       
        <!-- 表情 -->
        <div class="emoji" v-show="emojiShow">
          <ul>
            <li v-for="item in emojiData" @click="selectEmoji(item[0])"><img :id="item[0]" :src="item[1]" alt="" /></li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
 
export default {
  name: 'hello',
  data () {
    return {
      onGroupSystemNotifys :{
          
      },
      loginInfo: {
        'sdkAppID':'', //用户所属应用id,必填
        'accountType':'', //用户所属应用帐号类型，必填
        'identifier': '', //当前用户ID,必须是否字符串类型，选填
        'userSig': '', //当前用户身份凭证，必须是字符串类型，选填
        'headurl': ''//当前用户默认头像，选填
      },
      listeners: {
        "onConnNotify": function(){console.log(1)}, //选填
        "jsonpCallback": function(){console.log(2)}, //IE9(含)以下浏览器用到的jsonp回调函数,移动端可不填，pc端必填
        "onBigGroupMsgNotify": (res)=>{this.onBigGroupMsg(res)},//onBigGroupMsgNotify,//监听新消息(大群)事件，必填
        "onMsgNotify": function(){console.log(3)},//监听新消息(私聊(包括普通消息和全员推送消息)，普通群(非直播聊天室)消息)事件，必填
        "onGroupSystemNotifys": function(){console.log(4)}, //监听（多终端同步）群系统消息事件，必填
        "onGroupInfoChangeNotify": function(){console.log(5)}//监听群资料变化事件，选填
      },
      options: {
        'isAccessFormalEnv': true,//是否访问正式环境，默认访问正式，选填
        'isLogOn': true
      },
      chatBtnShow:true,
      chatBoxShow:false,
      identifier:"",
      chatItem:[],
      avChatRoomId:'@TGS#a43BMC5EB',//聊天室群号
      emojiData:[],
      emojiShow:false,
      focusStatus:false
    }
  },
  mounted: function(){
    this.sdkLogin();
    this.initEmojiUL();
  },
  watch:{
    
  },
  methods:{
    //登录
    sdkLogin() {
        var loginInfo=this.loginInfo;
        var listeners=this.listeners;
        var options=this.options;
        var avChatRoomId=this.avChatRoomId;
        var _self=this;
        //web sdk 登录
        webim.login(loginInfo, listeners, options,
            function(identifierNick) {
                //identifierNick为登录用户昵称(没有设置时，为帐号)，无登录态时为空
                // console.log(loginInfo)
                webim.Log.info('webim登录成功');
                _self.applyJoinBigGroup(avChatRoomId); //加入大群
                // hideDiscussForm(); //隐藏评论表单
                // initEmotionUL(); //初始化表情
            },
            function(err) {
                alert(err.ErrorInfo);
            }
        )
    },   
    applyJoinBigGroup(groupId) {
        var options = {
            'GroupId': groupId //群id
        };
        webim.applyJoinBigGroup(
            options,
            function(resp) {
                //JoinedSuccess:加入成功; WaitAdminApproval:等待管理员审批
                if (resp.JoinedStatus && resp.JoinedStatus == 'JoinedSuccess') {
                    webim.Log.info('进群成功');
                    var selToID = groupId;
                } else {
                    alert('进群失败');
                }
            },
            function(err) {
                alert(err.ErrorInfo);
            }
        );
    },
    //表情
    initEmojiUL() {
      this.emojiData=webim.Emotions
    },
    selectEmoji:function(res){
      this.$refs.chatInput.value=this.$refs.chatInput.value+res
      
    },
    //监听群消息
    onBigGroupMsg(msgList) {
            // webim.Log.error('receive a new group msg: ' + msg.getFromAccountNick());                     
            msgList.forEach((v,i)=>{
                this.chatItem.push(v);
                console.log(v);               
            });
            
          //获取chatItem数组最后几个值          
          if(this.chatItem.length>6){
            this.chatItem=this.chatItem.slice(this.chatItem.length-6,this.chatItem.length)
          }
    },
    closeLogin:function(){
      this.loginShow=false;
    },
    independentModeLogin:function() {
        
    },
    chatClick:function() {
        if (!this.loginInfo.identifier) { //未登录
            if (accountMode == 1) { //托管模式
                //将account_type保存到cookie中,有效期是1天
                webim.Tool.setCookie('accountType', loginInfo.accountType, 3600 * 24);
                //调用tls登录服务
                this.tlsLogin();
            } else { //独立模式
                alert('请填写帐号和票据');
                // $('#login_dialog').show();
            }
            return;
        } else {
            this.chatBtnShow=false;
            this.chatBoxShow=true;
            setTimeout(()=>{
              this.$refs.chatInput.focus();
            },30)
            
        }
    },
    //点击表情显示表情框
    emojiClick:function(){
      this.emojiShow=!this.emojiShow;
      if(this.emojiShow){
        this.$refs.chatInput.blur();
      }else{
        this.$refs.chatInput.focus();
      }
    },
    //发送消息
    onSendMsg:function() {
        if (!this.loginInfo.identifier) { //未登录
            if (accountMode == 1) { //托管模式
                //将account_type保存到cookie中,有效期是1天
                webim.Tool.setCookie('accountType', loginInfo.accountType, 3600 * 24);
                //调用tls登录服务
                this.tlsLogin();
            } else { //独立模式
                alert('请填写帐号和票据');
                // $('#login_dialog').show();
            }
            return;
        }
        var selToID = this.avChatRoomId;
        if (!selToID) {
            alert("您还没有进入房间，暂不能聊天");
            this.$refs.chatInput.value="";
            return;
        }
        var msgtosend=this.$refs.chatInput.value;
        // console.log(this.msgtosend);
        var msgLen = webim.Tool.getStrBytes(msgtosend);
        if (msgtosend.length < 1) {
            alert("发送的消息不能为空!");
            return;
        }
        var selType = webim.SESSION_TYPE.GROUP;
        var selSessHeadUrl="";
        if (!sess) {
          var sess = new webim.Session(selType, selToID, selToID, selSessHeadUrl, Math.round(new Date().getTime() / 1000));
        }
        var isSend = true; //是否为自己发送
        var seq = -1; //消息序列，-1表示sdk自动生成，用于去重
        var random = Math.round(Math.random() * 4294967296); //消息随机数，用于去重
        var msgTime = Math.round(new Date().getTime() / 1000); //消息时间戳
        var subType; //消息子类型
        if (selType == webim.SESSION_TYPE.GROUP) {
            //群消息子类型如下：
            //webim.GROUP_MSG_SUB_TYPE.COMMON-普通消息,
            //webim.GROUP_MSG_SUB_TYPE.LOVEMSG-点赞消息，优先级最低
            //webim.GROUP_MSG_SUB_TYPE.TIP-提示消息(不支持发送，用于区分群消息子类型)，
            //webim.GROUP_MSG_SUB_TYPE.REDPACKET-红包消息，优先级最高
            subType = webim.GROUP_MSG_SUB_TYPE.COMMON;

        } else {
            //C2C消息子类型如下：
            //webim.C2C_MSG_SUB_TYPE.COMMON-普通消息,
            subType = webim.C2C_MSG_SUB_TYPE.COMMON;
        }
        var identifier=this.loginInfo.identifier;
        var identifierNick='';
        var msg = new webim.Msg(sess, isSend, seq, random, msgTime, identifier, subType, identifierNick);
        //解析文本和表情
        var expr = /\[[^[\]]{1,3}\]/mg;
        var emotions = msgtosend.match(expr);
        var text_obj, face_obj, tmsg, emotionIndex, emotion, restMsgIndex;
        if (!emotions || emotions.length < 1) {
            text_obj = new webim.Msg.Elem.Text(msgtosend);
            msg.addText(text_obj);
        } else { //有表情

            for (var i = 0; i < emotions.length; i++) {
                tmsg = msgtosend.substring(0, msgtosend.indexOf(emotions[i]));
                if (tmsg) {
                    text_obj = new webim.Msg.Elem.Text(tmsg);
                    msg.addText(text_obj);
                }
                emotionIndex = webim.EmotionDataIndexs[emotions[i]];
                emotion = webim.Emotions[emotionIndex];
                if (emotion) {
                    face_obj = new webim.Msg.Elem.Face(emotionIndex, emotions[i]);
                    msg.addFace(face_obj);
                } else {
                    text_obj = new webim.Msg.Elem.Text(emotions[i]);
                    msg.addText(text_obj);
                }
                restMsgIndex = msgtosend.indexOf(emotions[i]) + emotions[i].length;
                msgtosend = msgtosend.substring(restMsgIndex);
            }
            if (msgtosend) {
                text_obj = new webim.Msg.Elem.Text(msgtosend);
                msg.addText(text_obj);
            }
        }
        var _self=this;
        webim.sendMsg(msg, function(resp) {
            if (selType == webim.SESSION_TYPE.C2C) { //私聊时，在聊天窗口手动添加一条发的消息，群聊时，长轮询接口会返回自己发的消息
                showMsg(msg);
            }
            webim.Log.info("发消息成功");
            _self.chatBtnShow=true;
            _self.chatBoxShow=false;
            _self.$refs.chatInput.value="";
            _self.emojiShow=false;
        }, function(err) {
            webim.Log.error("发消息失败:" + err.ErrorInfo);
            alert("发消息失败:" + err.ErrorInfo);
        });
    }
  },
  filters:{
    emojiFilter:function(res){
        if(res){
          return webim.Emotions[res][1]
        }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  
}

a {
  color: #42b983;
}
.hello{
  width: 100%;
  height: 100%;
  -webkit-position: fixed;
  position: fixed;
  left: 0;
  top: 0;
  z-index: 1;
  overflow-y: auto;
  background: url(../assets/back-img.png) no-repeat center center;
  -webkit-background-size: 100%;
  background-size: 100%;
}
.emoji ul {
  margin: 0;
  padding-left: 0.24rem;
  padding-bottom: 0.3rem;
  overflow: hidden;
  background: #fff;
}
.emoji ul li{
  float: left;
  width:0.8rem;
  height: 0.7rem;
}
.discuss{
  width: 100%;
  position: fixed;
  left: 0;
  overflow: hidden;
  bottom: 2rem;
  overflow-y: scroll;
}
.chatLi{
  margin-top: 0.1rem;

}
.chatLi:first-child{
  display: none;
}
.video-sms-pane div{
  background: rgba(0, 0, 0, 0.6);
  display: inline-block;
  max-width: 5rem;
  padding: 0.05rem;
  min-height: 0.4rem;
  line-height: 0.4rem;
  margin-left: 0.24rem;
  border-radius: 0.06rem;
  text-align: left;
  vertical-align: middle;
  font-size: 0.24rem;
  color:#333;

}
.video-sms-pane div span{
  
  vertical-align: middle;
  color: #fff;
}
.video-sms-pane div .uniqueId{
  color:#ff711a;
}
.chat_btn {
  position: fixed;
  width: 100%;
  height: 1rem;
  left: 0;
  bottom: 0.4rem;
}
.chat_btn .chat_input{
  float: left;
  width: 1rem;
  height: 1rem;
  margin-left: 0.24rem;
  border-radius: 50%;
  background: url(../assets/comment.png) no-repeat center center;
  -webkit-background-size: 100%;
  background-size: 100%;
}
.chat_btn .chat_like{
  float: right;
  width: 1rem;
  height: 1rem;
  margin-right: 0.24rem;
  border-radius: 50%;
  background: url(../assets/like.png) no-repeat center center;
  -webkit-background-size: 100%;
  background-size: 100%;
}
.chat_box{
  position: fixed;
  width: 100%;
  left: 0;
  bottom: 0;
}
.chat_box .chat_box_btn{
  height: 0.8rem;
  padding-top: 0.2rem;
  background: #fff;
  width:100%;
  position: relative;
}
.chat_box input{
  width: 65%;
  font-size: 0.32rem;
  height: 0.6rem;
  margin-left: 0.24rem;
  padding-left: 0.2rem;
  border-radius: 0.08rem;
  float: left;
  padding-right: 0.1rem;
  -webkit-box-sizing: border-box;
  box-sizing: border-box;
}
.chat_box span{
  display: block;
  background:#fff url(../assets/face.png) no-repeat center center;
  -webkit-background-size: 100%;
  background-size: 100%;
  color: #fff;
  text-align: center;
  height: 0.5rem;
  line-height: 0.5rem;
  width: 0.5rem;
  position: absolute;
  top: 0.25rem;
  z-index: 66;
  border-radius: 50%;
  left: 72%;
}
.chat_box button{
  width:15%;
  height: 0.6rem;
  font-size: 0.3rem;
  color: #333;
  margin-right: 0.24rem;
  border-radius: 0.04rem;
  float: right;
}
#login_dialog input{
  width:40%;
  height: 1rem;
  line-height: 1rem;
  font-size: 0.72rem;
}
#login_dialog button{
  width: 3rem;
  height: 1rem;
  font-size: 0.4rem;
}
</style>
