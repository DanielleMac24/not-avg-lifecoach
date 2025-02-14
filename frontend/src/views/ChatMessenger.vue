<template>
  <div id="chat-bot">
    <div id="chat-header">
      <h5 id="chat-header-text">Not Your Average Life Coach</h5>
      <select name="lang" v-model="chosenLang" class="lang-opt">
        <option value="en" selected>English</option>
        <option value="de">German</option>
        <option value="zh">Chinese</option>
        <option value="ru">Russian</option>
        <option value="fr">French</option>
        <option value="es">Spanish</option>
        <option value="hi">Hindu</option>
      </select>
      <button
        :disabled="userMessages.length === 0 || typingEnabled === false"
        @click="goToChatAnalysisRoute"
        class="button-link"
      >
        Analyze
      </button>
    </div>
    <div class="chat-messages" v-chat-scroll>
      <div class="filler"></div>
      <div v-for="(message, index) in conversation" :key="message.index">
        <Bot :conversation="conversation" :index="index" />
        <User :conversation="conversation" :index="index" />
      </div>
    </div>
    <div class="user-input">
      <input
        :disabled="!typingEnabled"
        ref="textinput"
        class="text-input"
        type="text"
        @keyup.enter="sendMessage"
        v-model="userMessage"
        placeholder="Type a message..."
      />
      <button @click="sendMessage" class="send-message">
        <img id="send-icon" src="../assets/sendIcon.png" />
      </button>
      <button @click="wikiMessage" class="wiki-message">
        <img id="wiki-icon" src="../assets/wiki.png" />
      </button>
    </div>
  </div>
</template>

<script>
import Vue from "vue";
import { makeHandshake, postMessage, getBotReply } from "@/services/axios.js";
import VueChatScroll from "vue-chat-scroll";
import FormSelectPlugin from "bootstrap-vue";
import Bot from "@/components/Bot.vue";
import User from "../components/User.vue";
import translate from "translate";
import wikipedia from "wikipedia";
Vue.use(FormSelectPlugin);
Vue.use(VueChatScroll);

translate.engine = "libre";

export default {
  components: {
    Bot,
    User,
  },
  created() {
    this.nlpHandshake();
    this.initialMessage();
    this.$nextTick(() => {
      this.$refs["textinput"].focus();
    });
  },
  data() {
    return {
      chosenLang: "en",
      nlpRestToken: "",
      userMessage: "",
      reply: "",
      allConvoData: undefined,
      botMessages: [],
      userMessages: [],
      botMessageCount: -1,
      conversation: [],
      typingEnabled: true,
      error: "",
    };
  },

  name: "ChatBot",
  props: {
    msg: String,
  },
  methods: {
    goToChatAnalysisRoute() {
      this.$store.commit("setAllConvoData", this.allConvoData);
      this.$store.commit("setConversation", this.conversation);
      this.$router.push("chat-analysis");
    },
    initialMessage() {
      this.conversation.push({
        chatStyle: "bot",
        text: "Hello, I am your Motivational Lifecoach, ask me anything!",
      });
    },
    nlpHandshake() {
      makeHandshake()
        .then((dataId) => {
          this.nlpRestToken = dataId;
        })
        .catch((error) => {
          this.error = "handshake api call is unsuccessful";
        });
    },
    sendMessage() {
      if (this.userMessage != "") {
        this.userMessages.push(this.userMessage);
        this.conversation.push({
          chatStyle: "user",
          text: this.userMessage,
        });
        postMessage(this.userMessage, this.nlpRestToken)
          .then(() => {
            this.typingEnabled = false;
            setTimeout(() => {
              this.typingEnabled = true;
              this.$nextTick(() => {
                this.$refs["textinput"].focus();
              });
              this.getReply();
            }, Math.random() * 1500 + 500);
          })
          .catch((error) => {
            console.log(error);
          })
          .finally(() => {
            this.userMessage = "";
          });
      }
    },
    wikiMessage() {
      if (this.userMessage != "") {
        this.userMessages.push(this.userMessage);
        this.conversation.push({
          chatStyle: "user",
          text: this.userMessage,
        });
        wikipedia
          .page(this.userMessage)
          .then((data) => {
            data.intro().then((data) => {
              translate(data, this.chosenLang).then((data) => {
                this.conversation.push({
                  chatStyle: "bot",
                  text: data,
                });
              })
            });
          })
          .then((this.userMessage = ""))
          .catch((error) => {
            this.conversation.push({
              chatStyle: "bot",
              text: "Could not find this article on Wikipedia!",
            });
          });
      }
    },
    getReply() {
      getBotReply(this.nlpRestToken)
        .then((response) => {
          this.reply = response.data.activities[(this.botMessageCount += 2)];
          this.allConvoData = response.data;
          this.botMessages.push(this.reply.text);
          translate(this.reply.text, this.chosenLang).then((data) => {
            this.conversation.push({
              chatStyle: "bot",
              text: data,
            });
          });
        })
        .catch((error) => {
          console.log(error);
        })
        .finally(() => {});
    },
    updateMessage(currentMessage) {
      this.userMessage = currentMessage;
    },
  },
};
</script>

<style lang="scss">
@import "@/scss/theme.scss";

@media only screen and (max-width: 600px) {
  #chat-bot {
    width: 100vw !important;
    height: 100vh !important;
    margin: 0 !important;
  }
}

.button-link {
  height: fit-content;
  background-color: $accent;
  border-radius: 8px;
  border-width: 1px;
  border-color: $accent;
  text-shadow: 1px 1px 1px $accentDark;
  color: $light;
  padding: 3px 10px;
  transition-duration: 0.4s;
  text-align: center;
  font-size: 16px;
  margin-left: auto;
  margin-right: 10px;
}
.lang-opt {
  background-color: whitesmoke;
  background-clip: padding-box;
  border: 0.25rem solid turquoise;
  border-radius: 8px;
  margin-right: 10px;
  color: purple;
  display: block;
  font-size: 1rem;
  height: fit-content;
  line-height: 1;
  padding: 0.15rem 0.65rem;
  width: 30%;
  transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
  transition-property: border-color, box-shadow;
  transition-duration: 0.15s, 0.15s;
  transition-timing-function: ease-in-out, ease-in-out;
  transition-delay: 0s, 0s;
}
.button-link:hover {
  background-color: $accentDark;
  color: #ffffff;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
#chat-header-text {
  margin-left: auto;
  margin-right: auto;
}
#chat-header {
  margin: 0em 0em auto 0em;
  background-color: $primary;
  justify-content: center;
  align-items: center;
  height: fit-content;
  display: flex;
  box-shadow: 0px 2px 5px 0px $neutral;
  z-index: 100;
}
#chat-header h5 {
  color: $light;
  margin: 3% 2%;
}
.send-message {
  background-color: $light;
  z-index: 1;
  border: none;
  width: 3em;
  border-radius: 40px;
}
.user-input {
  background-color: $light;
  border-top: $neutral solid 2px;
  margin: 0em 0em 0em 0em;
  display: flex;
}
.text-input {
  padding: 1em;
  border-right: none;
  border-bottom: none;
  border-left: none;
  border-top: none;
  width: -webkit-fill-available;
  width: -moz-available;
}
.filler {
  height: -webkit-fill-available;
  height: -moz-available;
  height: inherit;
}
.chat-messages {
  display: flex;
  flex-direction: column;
  overflow: hidden;
  overflow-y: scroll;
  height: -webkit-fill-available;
  height: -moz-available;
  height: 100%;
}
.chat-message {
  display: flex;
  width: 50%;
  padding: 0.2em;
  margin-top: 0.5em;
  margin-bottom: 0.5em;
}
.wiki-message {
  background-color: $light;
  z-index: 1;
  border: none;
  padding-right: 3em;
  padding-left: 0em;
  width: 3em;
  border-radius: 40px;
}
.botMessage {
  background-color: $tertiary;
  margin-right: auto;
  margin-left: 5px;
  border-radius: 5px;
  box-shadow: 0px 2px 5px $neutral;
  padding-left: 15px;
  padding-top: 15px;
  padding-bottom: 15px;
  padding-right: 15px;
}
.botPic {
  width: 3em;
  height: 3em;
}
.bot-flexbox {
  display: flex;
  justify-content: center;
  align-items: center;
}
.user-flexbox {
  display: flex;
  justify-content: center;
  align-items: center;
}
.userMessage {
  background-color: $secondary;
  margin-left: auto;
  margin-right: 5px;
  border-radius: 5px;
  margin-bottom: 1em;
  box-shadow: 0px 2px 5px $neutral;
  padding-left: 15px;
  padding-top: 15px;
  padding-bottom: 15px;
  padding-right: 15px;
}
.userPic {
  width: 3em;
  height: 3em;
}
header .filler {
  flex: 0 10000 100%;
}
#send-icon {
  width: 1.75rem;
}
#wiki-icon {
  width: 3em;
}
#chat-bot {
  padding: 0px;
  background-color: $light;
  border: $accent solid 4px;
  width: 450px;
  height: 80vh;
  display: flex;
  flex-direction: column;
  margin: 30px auto;
  border-radius: 5px;
}
textarea:focus,
input:focus,
button:focus {
  outline: none;
}
</style>
