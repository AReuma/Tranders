<template>
  <form @submit.prevent="onSubmit" >
      <div id='full'>
        <div id='left'>
          <div id='image'>
              <div>
                  <div style="border-radius: 50%; overflow: hidden; margin-top: 20px">
                      <v-img width="44" height="44" src="@/assets/profile.jpg"/>
                  </div>
              </div>
          </div>
        </div>
        
        <div id='center'>
          <div id='name'>
            <div id='name_value'>{{userInfo.name}}</div>
          </div>
          <div id='chatList'>
            <div id='chatroom' v-for="chatroom in chatrooms" :key="chatroom.roomNo">
              <button @click="goLink()" style="width:100%">
                <router-link id='chatroom_link' 
                  :style="isBackground ? 'background-color: #f2f3f6;' : ''"
                  :to="{
                    name: 'ChattingReadView',
                    params: {roomNo: chatroom.roomNo.toString()}}"

                  >
                <div>
                    <div style="border-radius: 50%; overflow: hidden; margin-right: 12px; width: 40px; height: 40px">
                        <v-img src="@/assets/profile.jpg"/>
                    </div>
                </div>
                <div style="display: flex; ">
                  <div>
                    <div v-if="login.memberNo==chatroom.member2.memberNo" style="display: flex; align-items: center; height: 20px;">
                      <span  style="font-weight: bold; font-size: 13px;">{{chatroom.member1.name}}</span>&nbsp;<span style="font-size: 12px;">{{chatroom.member1.region}}</span>
                    </div>
                    <div v-if="login.memberNo==chatroom.member1.memberNo" style="display: flex; align-items: center; height: 20px;">
                      <span  style="font-weight: bold; font-size: 13px;">{{chatroom.member2.name}}</span>&nbsp;<span style="font-size: 12px;">{{chatroom.member2.region}}</span>
                    </div>
                      <div v-if="lastMessage.roomNo==chatroom.roomNo" style="display: flex;
                      -webkit-box-align: center;
                      align-items: center;
                      height: 20px;
                      font-size: 13px;
                      color: var(--seed-scale-color-gray-700);">{{lastMessage.message}}
                      </div>
                      <div v-else style="display: flex;
                      -webkit-box-align: center;
                      align-items: center;
                      height: 20px;
                      font-size: 13px;
                      color: var(--seed-scale-color-gray-700);">{{chatroom.lastMessage}}
                      <!-- {{new_data.slice(-1)[0].content.message}} -->
                      </div>
                    </div>
                  </div>
                  <div v-if="chatroom.productBoard" style="display: flex; margin-left: auto;">
                      <v-img style="margin-right: 4px;
                      border: 1px solid var(--profile-image-border);
                      border-radius: 4px;
                      width: 40px;
                      height: 40px;
                      object-fit: cover;" :src="require(`@/assets/pImage/${chatroom.productBoard.productImage}`)"/>
                  </div>
                </router-link></button>
            </div>
          </div>
        </div>
        
        <div id='right'>
          <div id='right1'>

              <div>
                  <div style="border-radius: 50%; overflow: hidden; margin-right: 12px; width: 40px; height: 40px">
                      <v-img src="@/assets/profile.jpg"/>
                  </div>
              </div>
              <span v-if="login.memberNo==chatroom.member2.memberNo">{{chatroom.member1.name}} {{chatroom.member1.temperature}}°C</span>
              <span v-if="login.memberNo==chatroom.member1.memberNo">{{chatroom.member2.name}} {{chatroom.member2.temperature}}°C</span>

            <div id="right_button"  v-if="chatroom.productBoard.process==='판매중'">
            <v-layout>
              <v-dialog persisten max-width="400">
                  <template v-slot:activator="{ on }">
                      <v-btn id='my_button' depressed v-on="on">약속잡기</v-btn>
                  </template>
                  <template v-slot:default="dialog">
                      <v-card>
                          <v-card-title class="headline">
                              약속 설정
                          </v-card-title>
                          <v-card-text>
                              <v-menu
                                ref="menu"
                                v-model="menu"
                                :close-on-content-click="false"
                                :return-value.sync="date"
                                transition="scale-transition"
                                offset-y
                                min-width="auto"
                              >
                                <template v-slot:activator="{ on, attrs }">
                                  <v-text-field
                                    v-model="date"
                                    label="날짜 선택"
                                    prepend-icon="mdi-calendar"
                                    readonly
                                    v-bind="attrs"
                                    v-on="on"
                                  ></v-text-field>
                                </template>
                                <v-date-picker
                                  v-model="date"
                                  no-title
                                  scrollable
                                >
                                  <v-spacer></v-spacer>
                                  <v-btn
                                    text
                                    color="primary"
                                    @click="$refs.menu.save(date)"
                                  >
                                    선택
                                  </v-btn>
                                  <v-btn
                                    text
                                    color="primary"
                                    @click="menu = false"
                                  >
                                    닫기
                                  </v-btn>
                                </v-date-picker>
                              </v-menu>
                              <v-menu
                                ref="menu2"
                                v-model="menu2"
                                :close-on-content-click="false"
                                :nudge-right="40"
                                :return-value.sync="time"
                                transition="scale-transition"
                                offset-y
                                max-width="290px"
                                min-width="290px"
                              >
                                <template v-slot:activator="{ on, attrs }">
                                  <v-text-field
                                    v-model="time"
                                    label="시간 선택"
                                    prepend-icon="mdi-clock-time-four-outline"
                                    readonly
                                    v-bind="attrs"
                                    v-on="on"
                                  ></v-text-field>
                                </template>
                                <v-time-picker
                                  v-if="menu2"
                                  v-model="time"
                                  full-width
                                  @click:minute="$refs.menu2.save(time)"
                                ></v-time-picker>
                              </v-menu>
                          </v-card-text>
                          <v-card-actions>
                              <v-spacer></v-spacer>
                              <v-btn color="blue" text @click.native="onAppoint($event)">
                                  완료
                              </v-btn>
                              <v-btn text @click="dialog.value=false">
                                  닫기
                              </v-btn>
                          </v-card-actions>
                      </v-card>
                  </template>
              </v-dialog>
          </v-layout>
          <!-- <v-layout v-if="login.memberNo!=chatroom.productBoard.member.memberNo"> -->
            <v-layout>
              <v-dialog persisten max-width="400">
                  <template v-slot:activator="{ on }">
                      <v-btn id='my_button' depressed style="margin-left:15px" v-on="on">송금하기</v-btn>
                  </template>
                  <template v-slot:default="dialog">
                                <v-card>
                                    <v-card-title class="headline">
                                        Pay
                                    </v-card-title>
                                    <v-card-text>
                                      <div id="pay_box">결제금액: {{chatroom.productBoard.price}} 원</div>
                                    </v-card-text>
                                    <v-card-text >
                                      <div id="pay_box" >
                                        <div>페이머니: <span :style="chatroom.productBoard.price>userInfo.money ? 'color:red' : ''">{{userInfo.money}} </span>원</div>
                                        <div style="display: inline-flex;
  margin-left: auto;"><v-btn style="
                                          
    border-radius: 24px;
    min-width: 80px;
    height: 36px;
    padding: 0px 9px;
    margin: 0px 5px
    letter-spacing: -0.1px;
    position: relative;
    overflow: hidden;
    color: rgb(25, 28, 32);
    background-color: rgb(255, 235, 0);" 
                                        @click="onCharge()">
                                            충전
                                        </v-btn></div></div>
                                        <br/>
                                        * 결제 버튼 클릭 시 바로 결제가 진행됩니다.
                                    </v-card-text>
                                    <v-card-actions>
                                        <v-spacer></v-spacer>
                                        
                                        <v-btn text color="blue" @click="onPay()">
                                            결제
                                        </v-btn>
                                        <v-btn text @click="dialog.value=false">
                                            취소
                                        </v-btn>
                                    </v-card-actions>
                                </v-card>
                            </template>
              </v-dialog>
          </v-layout>
          </div>
          </div>
          <div v-if="chatroom.productBoard" id='product' style="position:relative;">
            <div>
              <v-img style="margin-right: 12px;
              border: 1px solid var(--profile-image-border);
              border-radius: 4px;
              width: 40px;
              height: 40px;
              object-fit: cover;" :src="require(`@/assets/pImage/${chatroom.productBoard.productImage}`)"/>
            </div>
            <div style="flex-direction: column;
    -webkit-box-pack: center;
    justify-content: center;
    -webkit-box-flex: 1;
    flex-grow: 1;
    font-size: 14px;
    line-height: 150%;
    letter-spacing: -0.02em;">
              <div>{{chatroom.productBoard.title}}</div>
              <div style="font-weight: bold;">{{chatroom.productBoard.price}}원</div>
            </div>
            <div>{{chatroom.productBoard.process}}</div>
          </div>
          <div v-if="chatroom.appointDate" id='notice'>
            <div id='notice_box'>
              <div>
              <span style="font-weight: 700; margin-right: 4px;">알림</span>
              <span>{{chatroom.appointDate}}&nbsp;{{chatroom.appointTime}} 거래 약속을 만들었어요.</span>
              </div>
              <!-- <v-btn class='success' depressed @click="onReminder()"><b>알림 받기</b></v-btn>  -->
            </div>
          </div>
          <div id='chatView' :style="chatroom.appointDate ? 'height:500px': 'height:585px' && chatroom.productBoard ? 'height:581px':'height:656px' ">
            <!-- <div v-if="!new_data || (Array.isArray(new_data) && new_data.length === 0)">
                <div colspan="4">
                    {{chatroom.roomNo}} 채팅방에 입장하였습니다. 
                </div>
            </div> -->
            <!-- <div v-else> -->
            <div>
              <div v-for="msg in new_data" :key="msg.messageNo">
                <div style="display: flex; justify-content: flex-end;" v-if="login.memberNo==msg.content.memberNo">
                  <div id='message_date' v-if="chatroom.roomNo==msg.content.roomNo">{{msg.content.now}}</div>
                  <div id='message_greenBox' v-if="chatroom.roomNo==msg.content.roomNo && msg.content.image"><v-img width="200px" height="200" :src="require(`@/assets/chatting/${msg.content.message}`)"/></div>
                  <div id='message_greenBox' v-if="chatroom.roomNo==msg.content.roomNo && !msg.content.image"><b>{{msg.content.message}}</b></div>
                </div>
                <div style="display: flex;" v-else>
                  <div id='message_box' v-if="chatroom.roomNo==msg.content.roomNo"><b>{{msg.content.message}}</b></div>
                  <div id='message_box' v-if="chatroom.roomNo==msg.content.roomNo && msg.content.image"><v-img width="200px" height="200" :src="msg.content.message"/></div>
                  <div id='message_date' v-if="chatroom.roomNo==msg.content.roomNo && !msg.content.image">{{msg.content.now}}</div>
                </div>
              </div>
              <div style="display: flex; justify-content: flex-end;" v-for="msg in newMessage" :key="msg.messageNo">
                <div id='message_date'>{{msg.now}}</div>
                <div v-if="msg.image" id='message_greenBox'><v-img width="200px" height="200" :src="msg.message"/></div>
                <div v-else id='message_greenBox'><b>{{msg.message}}</b></div>
              </div>
            </div>
            
          </div> 
          <div v-if="priview" style="display: flex; position:absolute; bottom: 178px; background-color: rgba(128, 128, 128, 0.5); margin-left: 16px">
            <v-img style="margin-left:305px;" width="200px" height="200" :src="priview"/>
            <v-btn @click="deletePriview()" icon style="margin-right:237px;"><v-icon large color="white">mdi-alpha-x-circle-outline</v-icon></v-btn>
          </div>
          <div id='submit'>
            <div id='submit_form'>
              <textarea @keyup.enter="onSubmit" v-model="message" placeholder="메시지를 입력해주세요"></textarea>
              <div style="display: flex; 
              -webkit-box-pack: justify;
              justify-content: space-between;
              margin: 8px 10px;">
                <div style="display: flex;
                -webkit-box-align: center;
                align-items: center;
                ">
                      <label for="files">
                          <v-icon large style="margin-top:4px">mdi-camera</v-icon>
                      </label>
                      <input type="file" id="files" ref="files" v-on:change="handleFileUpload()"/>
                  </div>
                  
                <v-btn id='my_button' depressed type="submit">전송</v-btn>
              </div>
            </div>
          </div>
        </div>
      </div>
  </form>

</template>

<script>
import axios from 'axios'
import Vue from 'vue'
import cookies from "vue-cookies";
Vue.use(cookies)

export default {
  name: "ChattingRead",
  props: {
      userInfo: {
          type: Object,
          required: true
      },
      chatroom: {
          type: Object,
          require: true
      },
      chatrooms: {
          type: Array
      }
    },
  data() {
    return {
      login: {
        memberNo: cookies.get("memberNo"),
        id: cookies.get('id'),
        name: cookies.get('name'),
        access_token: cookies.get('access_token'),
      },
      
      message: '',
      lastMessage: '',
      now: ('0' + (new Date()).getHours()).slice(-2) +':'+('0' + (new Date()).getMinutes()).slice(-2),
      new_data: [],
      newMessage: [],
      date: (new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substr(0, 10),
      menu: false,
      time: '00:00',
      menu2: false,
      priview: '',
      isBackground: false,

      // realTime: false,
    }
  },
  created() {
      this.getNewData();


  },
  methods: {
    async getNewData() {
      await axios.get('http://127.0.0.1:5000/kafka-data')
        .then((res) => {
          console.log(res.data)
            this.new_data = res.data;
        }).catch((error) => {
            console.error(error);
        });
    },
    goChatroom(roomNo) {
      this.$router.push({name: 'ChattingReadView',
                          params: {roomNo: roomNo.toString()}})
    },
    handleFileUpload () {
      this.files = this.$refs.files.files
      this.priview = URL.createObjectURL(this.files[0])
    },
    deletePriview() {
      this.priview = ''
    },
    onSubmit() {
        const { roomNo } = this.chatroom
        const { memberNo } = this.login
        const { message, now } = this
        this.message=''
        if(this.priview){
          let formData = new FormData()
          formData.append('fileList', this.files[0])
          axios.post('http://localhost:7777/chatting/uploadImg', formData, {
                headers: {
                    'Content-Type': 'multipart/form-data',
                }
            })
          .then (()=> {
            this.$emit('onSubmit', { roomNo, memberNo, message:this.files[0].name, now, image:true })
          })
          this.newMessage.push({message:this.priview, now, image:true})
          this.priview=''
          this.lastMessage={roomNo, message:'사진을 전송 했습니다'}
          // this.realTime=true
        }
        if(message!='\n' && message!=''){
          this.$emit('onSubmit', { roomNo, memberNo, message, now })
          this.newMessage.push({message, now})
          this.message=''
          this.lastMessage={roomNo, message}
          // this.realTime=true
        }
    },
    onAppoint() {
      const { date, time } = this
      this.$emit('onAppoint', { roomNo: this.chatroom.roomNo, date, time })
      history.go(0);
    },
    onReminder() {
      const { access_token } = this.login
      const { date, time } = this
      this.$emit('onReminder', { access_token, date, time })
    },
    onCharge(){
      window.open('http://kko.to/LJwi9Wf7n')
      this.userInfo.money += this.chatroom.productBoard.price
      this.$emit('onCharge', {id:this.login.id, money: this.userInfo.money})
    },
    onPay(){
      if(this.userInfo.money>=this.chatroom.productBoard.price) {
        const { productNo } = this.chatroom.productBoard
        this.userInfo.money -= this.chatroom.productBoard.price
        this.$emit('onPay', {id:this.login.id, money: this.userInfo.money, memberId:this.chatroom.productBoard.member.id, productNo})
      }
    },
    goLink() {
      this.isBackground=!this.isBackground
      console.log(this.isBackground)
    }
  },
  // beforeUpdate() {

  //   console.log(this.realTime);
  //     if(this.realTime) {

  //       this.$router.go()
  //     }
  // }
}
</script>

<style scoped>
/* #top{
  border-bottom: 1px solid #bcbcbc;
} */
#logo{
  display: flex;
  align-items: center;
  width: 1200px;
  padding: 0px 8px;
}
#full{
  display: flex;
  border-left: 1px solid #e9ecef;
  border-right: 1px solid #e9ecef;
  width: 1200px;
  margin: 0 auto;
  line-height: 24px;
  background: #fff;
}
#left{
	width: 80px;
	float: left;	
}
#image{
  background-color: #ededed;
	width: 80px;
	height: 900px;
  display: flex; 
  justify-content: center;
}
#center{
	width: 310px;
	float: left;
  border: 1px solid #e9ecef;
  border-top-style: none;
  border-bottom-style: none;
}
#name{
    border-bottom: 1px solid #e9ecef;
    position: relative;
    display: flex;
    height: 72px;
    min-height: 72px;
    padding: 0px 20px;
    align-items: center;
}
#name_value{
    font-weight: bold;
    font-size: 16px;
    line-height: 150%;
}
#chatList{
	width: 310px;
	height: 825px;
}

#chatroom_link{
  display: flex;
  padding: 16px;
  height: 72px;
  border-bottom: 1px solid #e9ecef;
  -webkit-box-align: center;
  align-items: center;
  position: relative;
  overflow: hidden;
  background-position: center center;
  contain: content;
  color: inherit;
  cursor: pointer;
  text-decoration: none;
}
#chatroom_link:hover{
  background-color: #f2f3f6;
}
#right{
	width: 810px;
	float: left;
  border: 1px solid #e9ecef;
          border-top-style: none;
          border-left-style: none;
}
#right1{
	width: 810px;
	height: 72px;
  min-height: 72px;
  display: flex; 
  border-bottom: 1px solid #e9ecef;
    /* -webkit-box-pack: justify; */
    /* justify-content: space-between; */
    /* -webkit-box-align: center; */
    align-items: center;
    /* height: 64px; */
    /* min-height: 64px; */
    padding: 0px 20px;
}
#right_button{
  display: inline-flex;
  margin-left: auto;
    /* -webkit-box-pack: center;
    justify-content: center;
    -webkit-box-align: center;
    align-items: center;
    border-radius: 4px; */
}
#pay_box{
  border: 2px solid #e9ecef;;
  border-radius: 8px;
  height: 70px;
  display: flex;
  align-items: center;
  padding: 0px 15px
}
#product{

	height: 72px;
  display: flex;
    -webkit-box-align: center;
    align-items: center;
    -webkit-box-pack: justify;
    justify-content: space-between;
    padding: 0px 20px;
    font-size: 14px;
    border-bottom: 1px solid #e9ecef;
}
#notice{
  display: flex;
      -webkit-box-align: center;
    align-items: center;
    -webkit-box-pack: center;
    justify-content: center;
      padding: 20px 20px 0px 20px;
    

}
#notice_box{
    display: flex;
  padding: 20px;
  border-radius: 10px;
  font-size: 14px;
  letter-spacing: -0.02em;
  /* margin: 8px 0px; */
  background-color: #E6F3E6;
  color: #086e5b;
  width: 100%;
      align-items: center;
    justify-content: space-between;

}
#chatView{
	width: 810px;
	/* height: 555px; */
  overflow-y:auto; 
  /* overflow-x:hidden;  */
  /* overflow: hidden auto; */
  padding: 20px 20px 0px 20px;
      /* border-bottom: 1px solid #bcbcbc; */
}
#submit{
	/* height: 125px; */


}
#submit_form{

  /* display: flex; */
    /* flex-direction: column; */
    /* position: relative; */
    margin: 16px;
    border: 2px solid #e9ecef;
    border-radius: 8px;
    /* -webkit-box-pack: justify; */
    /* justify-content: space-between; */
}
#message_greenBox{
	/* height: 40px; */
  color: white;
  background-color: #086e5b;
  border-radius: 20px 2px 20px 20px;
  display: inline-flex;
  margin: 0px;
  padding: 10px 14px;
  max-width: 484px;
  max-height: 484px;
  white-space: pre-wrap;
  font-size: 14px;
  line-height: 150%;
  letter-spacing: -0.02em;
}
#message_box{
display: inline-flex;
    margin: 0px;
    padding: 10px 14px;
    max-width: 484px;
    word-break: break-word;
    white-space: pre-wrap;
    font-size: 14px;
    line-height: 150%;
    letter-spacing: -0.02em;
    border-radius: 2px 20px 20px;
    background-color: #eaebee;;
    /* color: #212124; */
}
#message_date{
  color: #868b94;
  font-size: 12px;
  /* line-height: 150%; */
  letter-spacing: -0.02em;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  padding: 0px 4px;
}
textarea{
  margin: 12px 12px 0px;
  width: calc(100% - 24px);
  height: 63px;
  /* line-height: 150%; */
  padding: 0px;
  /* resize: none; */
  font-size: 14px;
  border: none;
  outline: none;

}
#files {
    visibility: hidden;
}
#my_button{
  letter-spacing: -2%;
  background-color: white;
  color: #212124;
  height: 40px;
  font-weight: 700;
  padding: 5px 10px;
  border-radius: 3px;
  border: 1px solid #d1d3d8;
}
</style>