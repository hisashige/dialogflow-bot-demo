<template>
  <v-app>
    <v-app-bar
      color="light-blue lighten-1"
      dense
      dark
      fixed
      elevation="0"
    >
      <img src="../public/img/hamburger.png" alt="ロゴ" height="35px" class="mr-4">
      <v-toolbar-title class="font-weight-black">SoftwareDesignバーガー</v-toolbar-title>
    </v-app-bar>
    <v-main class="d-flex flex-column mainContainer">
        <v-sheet color="grey lighten-4" width="100vw">
          <v-container class="messageContainer px-8 pt-12" :style="`height: calc(100vh - ${computeInputHeight});`">
            <transition-group tag="div" name="slide" class="messageWrapper">
              <div v-for="item in dispMessages" :key="item.id" >
                <div class="d-flex justify-end">
                  <v-card
                    v-if="item.userMessage"
                    class="mt-5 mb-5 rightBalloon"
                    max-width="400"
                    elevation="0"
                  >
                    <v-card-text class="font-weight-black">
                      {{ item.userMessage }}
                    </v-card-text>
                  </v-card>
                </div>
                <div class="d-flex justify-start">
                  <v-avatar
                    color="transparent"
                    size="50"
                    class="mt-5 mr-5"
                  >
                    <img
                      src="../public/img/dezigon-icon.png"
                      alt="アイコン"
                      width="50px"
                    >
                  </v-avatar>
                  <v-card
                    v-if="item.botMessage"
                    class="mt-5 mb-5 leftBalloon"
                    max-width="400"
                    color="light-blue lighten-1"
                    elevation="0"
                    dark
                  >
                    <v-card-text class="font-weight-black">
                      {{ item.botMessage }}
                    </v-card-text>
                  </v-card>
                </div>
              </div>
            </transition-group>
          </v-container>
        </v-sheet>
        <v-sheet color="white" width="100vw" :height="computeInputHeight" class="inputContainer" elevation="10">
          <v-container class="pt-6">
            <v-row no-gutters>
              <v-col
                cols="12"
                md="10"
              >
              <v-text-field
                v-model="inputMessage"
                label="ご注文をどうぞ"
                outlined
              ></v-text-field>
            </v-col>
            <v-col
              cols="12"
              md="2"
              :class="{'alignRight': ['xs', 'sm'].includes($vuetify.breakpoint.name)}"
            >
              <v-btn
                class="ml-4 font-weight-black"
                color="light-blue lighten-2"
                @click="sendMessage()"
                x-large
                depressed
                dark
              >注文する</v-btn>
            </v-col>
            </v-row>
          </v-container>
        </v-sheet>
        <img src="../public/img/dezigon.png" alt="dezigon" height="300px" class="charaImage" v-if="['md', 'lg', 'xl'].includes($vuetify.breakpoint.name)" />
    </v-main>
  </v-app>
</template>

<script>
import { v4 as uuidv4 } from 'uuid';
import axios from 'axios'

export default {
  name: 'App',
  data: function() {
    return {
      talkId: 1, // カウント用の入力文字列と回答文字列セットのID
      inputMessage: '', // テキストフォームからの入力文章
      dispMessages: [{ // 会話メッセージ表示用データ
        id: 0, // 入力文字列と回答文字列セットのID
        userMessage: '', // 表示用のユーザーの入力文字列
        // ボットからの回答文字列
        botMessage: 'いらっしゃいませ！ご注文をどうぞ〜'
      }],
      sessionId: '' // Dialogflowの会話セッションを維持するID
    };
  },
  created() {
    this.sessionId = uuidv4()
  },
  computed: {
    computeInputHeight() {
      // 入力部分の高さをウインドウのサイズに合わせて変更する
      if (['xs', 'sm'].includes(this.$vuetify.breakpoint.name)) {
        return '180px'
      } else {
        return '120px'
      }
    }
  },
  watch: {
    dispMessages() {
      this.$nextTick(() => {
        this.scrollToEnd()
      })
    }
  },
  methods: {
    sendMessage() {
      // APIを呼び出し、Dialogflowでの自然言語処理結果から返却メッセージを取得する
      axios
      .get('http://localhost:3000/detectIntent', {
        params: {
          sessionId: this.sessionId,
          inputMessage: this.inputMessage
        }
      })
      .then(response => {
        const responseData = response.data
        if (responseData) {
          // 画面上のメッセージ一覧に結果を表示する
          this.dispMessages.push({
            id: this.talkId,
            userMessage: this.inputMessage,
            botMessage: responseData.fulfillmentText
          })
          this.inputMessage = ''
          this.talkId++
        }
      })
    },
    scrollToEnd () {
      // チャットを常に最下部に移動する
      const messageWrapper = document.getElementsByClassName('messageWrapper')
      if (messageWrapper) {
        messageWrapper[0].scrollIntoView({block: "end", inline: "nearest", behavior: "smooth"})
      }
    }
  }
};
</script>

<style>
.mainContainer {
  width: 100vw;
  height: 100vh;
}
.messageContainer {
  max-width: 800px !important;
  overflow-y: scroll;
  /* scrollbar非表示 */
  -ms-overflow-style: none;
  scrollbar-width: none;
}
.messageContainer::-webkit-scrollbar {
  /* scrollbar非表示 */
  display:none;
}
.alignRight {
  text-align: right;
}
.charaImage {
  position: fixed;
  left: 10px;
  bottom: 120px;
}
/* 吹き出し部分 */
.leftBalloon, .rightBalloon {
  position: relative;
}
.leftBalloon::before, .rightBalloon::before {
  position: absolute;
  content: '';
  display: inline-block;
  top: 25px;
  width: 13px;
  height: 13px;
  background-size: contain;
}
.leftBalloon::before {
  background-image: url('../public/img/left-balloon.svg');
  left: -12px;
}
.rightBalloon::before {
  background-image: url('../public/img/right-balloon.svg');
  right: -12px;
}
/* アニメーション */
.slide-enter-active, .slide-leave-active {
  transform: translate(0px, 0px); 
  transition: transform 800ms cubic-bezier(0, 0, 0.2, 1) 0ms, opacity 800ms;
  opacity: 1;
}
.slide-enter, .slide-leave-to {
  transform: translateY(20px);
  opacity: 0;
}
</style>
