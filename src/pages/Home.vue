<template>
  <div class="contain">
    <pageLoaded :pageLoading="pageLoading"></pageLoaded>
    <div class="menu">
      <div @click="cardCenter" class="cardCenter">
        <div class="cardCenternotice" v-if="carnotice"></div>
        <div class="personhead"></div>
        <div class="personbody"></div>
      </div>
      <div v-show="userID" class="mycount">￥{{mycount}}</div>
      <p class="great" ref="great"><span class="laba"></span><span class="kuang"><span class="greatcont" ref="greatcont">{{great}}</span></span></p>
      <HomeButton name="我要点餐" bgcolor="yellow" funname="order"  @order="order"></HomeButton>
      <div class="separ"></div>
      <HomeButton name="已点订单" bgcolor="skyblue" funname="finished"  @finished="finished" id="main"></HomeButton>
    </div>
    <myAlert title="温馨提示~" des="请输入你的手机号"  :isshow="isshow" @hiddleAlert="hiddleAlert" >
    </myAlert>
    <div v-transfer-dom>
      <Alert v-model="alertShow" title="出错啦~" button-text="确定" content="数据拉取失败"  hide-on-blur></Alert>
    </div>
  </div>
</template>

<script>
  import HomeButton from '@/components/HomeButton.vue'
  import {getLocalStorage, setLocalStorage} from '../utils/storage'
  import myAlert from "@/components/myAlert.vue"
  import pageLoaded from '@/components/preload-view.vue'
  import {Alert,TransferDomDirective as TransferDom} from 'vux'

  export default {
    directives: {
      TransferDom
    },
      data() {
          return {
            great:'',
            isshow:false,
            userID:'',
            pageLoading: {
                show: true
            },
            alertShow:false,
            timer:'',
            carnotice: false,
            mycount:0// 账户余额
          }
      },
      mounted() {
        let self = this;
          this.$http.get(this.$store.state.baseUrl + '/greet').then(res => {
             let data = res.data;
             if(data.error == 0){
               self.great = data.data.greet;
               if(data.data.carcount > 0) self.carnotice = true;
               self.pageLoading["show"] = false;
               self.dealanno(); //  处理喇叭提示字超长
             }else {
               self.alertShow = true;
               self.pageLoading["show"] = false;
             }
          }).catch(err =>{
            self.alertShow = true;
            self.pageLoading["show"] = false;
              console.log(err)
          })
        this.getmycount();//请求账户余额

      },
    beforeDestroy() {
      clearInterval(this.timer);
      localStorage.removeItem("btnindex");
    },
      components: {
          HomeButton,
          myAlert,
          pageLoaded,
          Alert
      },
    methods: {
        getmycount() { // 请求账户余额接口
          let self = this;
          this.userID = getLocalStorage("userID");
          if(this.userID){ // 请求账户接口
            this.$http.get(this.$store.state.baseUrl+'/mycount?userId='+this.userID).then(res=>{
                console.log(res);
              if(res.data.error == -1){
                  alert('账户余额加载失败');
              }else {
                  self.mycount = res.data.data.mycount;
              }
            }).catch(err=>{
              alert('账户余额加载失败');
            })
          }
        },
      dealanno() {
        let self = this;
        let great = this.$refs.great;
        let kuang = great.clientWidth - 40;
        let greatWidth = 0.75*(document.documentElement.clientWidth/320)*20*(this.great.length);
        let dert = greatWidth - kuang;
        if(dert >= 5) {
            self.timer = setInterval(function () {
              document.querySelector('.greatcont').style.transition = "left 3s linear 0s";
              document.querySelector('.greatcont').style.left = -dert +'px';
              setTimeout(function () {
                document.querySelector('.greatcont').style.transition = "";
                document.querySelector('.greatcont').style.left = 0 + 'px';
              },2000,false);
            },3500,false)
        }
      },
      cardCenter() {
        this.isshow = this.getIshow();
        if(!this.isshow) {
            this.$router.push('/cardCenter')
        }
      },
      finished() {
          setLocalStorage("btnindex",2)
          this.isshow = this.getIshow();
          if(!this.isshow) {
              this.$router.push("/orderList")
          }
      },
      order() {
        console.log(this.userID)
        this.isshow = this.getIshow();
        setLocalStorage("btnindex",1)
        if(!this.isshow) {
          this.$router.push("/orderMain");
        }
      },
      getIshow() {
          this.userID = getLocalStorage("userID");
          return !this.userID
      },
      hiddleAlert() {
          this.isshow = false;
      }
    }
  }
</script>

<style scoped>
  .test {
    font-size: 0.8rem;
    white-space: nowrap;
  }
  .contain {
    overflow: hidden;
    width: 100%;
    height: 28rem;
    display: flex;
    display: -webkit-flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background:  url("../../static/bgHome.jpg") no-repeat;
    background-size: 30% 15%;
  }
  .menu {
    width: 80%;
    border: 1px dashed gray;
    height: 20rem;
    display: flex;
    display: -webkit-flex;
    flex-direction: column;
    align-items: center;
    background-color: #FFFFCC;
    border-left: 3px solid #C0C0C0;
    margin-top: 1rem;
    overflow: hidden;
    position: relative;
  }
  .great {
    font-size: 0.8rem;
    color: 	#C0C0C0;
    margin-top: 2rem;
    margin-bottom: 2.6rem;
    text-align: center;
    overflow: hidden;
    white-space: nowrap;
    width: 80%;
  }
  .laba {
    background: url("../../static/laba.jpg");
    width: 30px;
    height: 30px;
    display: inline-block;
    background-size: cover;
    transform: rotate(-160deg);
    margin-right: 10px;
    animation: labarotate 1.5s ease 1s infinite;
  }
  .separ {
    width: 100%;
    height: 2rem;
  }
  .greatcont {
    display: inline-block;
    position: relative;
    left: 0;
    top: 0;
  }
  .kuang {
    display: inline-block;
    overflow: hidden;
  }
  @keyframes labarotate {
    0% {
      transform: rotate(-160deg) scale(1);
    }
    25% {
      transform: rotate(-140deg) scale(1.1);

    }
    50% {
      transform: rotate(-160deg) scale(1);
    }
    75% {
      transform: rotate(-160deg);
    }
    100% {
      transform: rotate(-160deg);
    }
  }
  .cardCenter {
    position: absolute;
    text-align: center;
    height: 20px;
    overflow: hidden;
    border-bottom: 1px solid #646464;
    right: 60px;
    top: 14px;
  }
  .personhead {
    border-radius: 50%;
    width: 10px;
    height: 10px;
    border: 1px solid #646464;
    margin: 0 auto;
    margin-bottom: 1px;
  }
  .personbody {
    border-radius: 50%;
    width: 16px;
    height: 16px;
    border: 1px solid #646464;
  }
  .cardCenternotice {
    position: absolute;
    width: 7px;
    height: 7px;
    background-color: red;
    border-radius: 50%;
    right: 0;
    top: 1px;
  }
  .mycount {
    position: absolute;
    top: 16px;
    right: 17px;
    font-size: 14px;
    color: #A0A0A0;
  }
</style>
