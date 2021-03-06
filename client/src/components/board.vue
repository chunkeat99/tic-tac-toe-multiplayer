<template>
  <div>
    <h1>Room Name: {{roomID}}</h1>
    <h3 v-if="!isStart&&countDown==0">Waiting for players...</h3>
    <div class="playersList">
      <player
        v-for="player in players"
        :key="player.id"
        :player="player"
        :currentTurn="currentTurn"
        :socket="socket"
      />
    </div>

    <div id="board" v-show="isStart">
      <cell
        v-for="cell in cells"
        :key="cell.id"
        :img="cell.img"
        v-on:change="move(cell.id)"
        :canClick="cell.canClick"
        :size="size"
        :id="cell.mark"
      />

      <transition name="overlay">
        <div v-if="winnerImg!=''" class="overlay">
          <img :src="winnerImg" />
        </div>
      </transition>
    </div>

    <transition name="countDown" mode="out-in">
      <h1 v-if="countDown>0" :key="countDown" id="countDown">{{countDown}}</h1>
    </transition>

    <div class="chatBox">
      <transition name="msgInput">
        <input v-show="showMsg" type="text" v-model="msg" @keyup.enter="sendMsg()" />
      </transition>
      <img src="msgbtn.svg" @click="sendMsg()" />
    </div>

    <button
      v-show="(!isStart||winnerImg!='')&&countDown==0"
      v-on:click="$bus.emit('back')"
      class="btn bbtn"
    >BACK</button>
    <button v-show="winnerImg!=''" class="btn bbtn" v-on:click="playAgain">PLAY AGAIN</button>
  </div>
</template>

<script>
import cell from "./cell.vue";
import player from "./player.vue";

export default {
  name: "board",
  props: ["socket"],

  data() {
    return {
      roomID: "",
      players: [],
      cells: [],
      currentTurn: "",
      isStart: false,
      winnerImg: null,
      size: null,
      countDown: 0,
      msg: "",
      showMsg: false
    };
  },

  created() {
    this.socket.on("chat", (id, msg) => {
      this.$bus.emit("chat", id, msg);
    });
    this.socket.on("waiting", time => {
      this.countDown = time;
    });
    this.socket.on("update", data => {
      this.roomID = data.roomID;
      this.cells = data.cells;
      this.currentTurn = data.currentTurn;
      this.winnerImg = data.winner.img;
      this.players = data.players;
      this.isStart = data.isStart;
      this.size = data.size;
    });
  },

  methods: {
    move: function(id) {
      this.socket.emit("move", id);
    },
    playAgain: function() {
      this.socket.emit("playAgain");
    },
    sendMsg: function() {
      if (!this.showMsg) this.showMsg = true;
      else if (this.showMsg && this.msg == "") this.showMsg = false;
      if (this.msg == "") return;
      this.socket.emit("chat", this.msg);
      this.msg = "";
    }
  },

  components: {
    cell,
    player
  }
};
</script>

<style scoped>
.chatBox {
  display: flex;
  flex-flow: row wrap;
  align-items: center;
  height: 40px;
}
.chatBox input {
  margin: 0 10px;
  text-align: center;
  height: 100%;
  padding: 21px 13px;
  display: inline-block;
  border: none;
  border-radius: 20px;
  box-sizing: border-box;
  box-shadow: 0 3px 4px rgba(0, 0, 0, 0.8);
  outline: none;
  font-weight: bold;
  font-size: large;
}

.chatBox img {
  display: inline-block;
  height: 100%;
  border-radius: 50%;
  border: 3px solid #f5f5f5;
  box-shadow: 0 3px 4px rgba(0, 0, 0, 0.8);
  transition: 0.1s ease-in-out;
  margin: auto;
}
.chatBox img:hover {
  transform: scale(1.1);
}

#countDown {
  font-size: 200px;
  margin: 0;
  user-select: none;
}

.overlay-enter-active {
  transition: all 0.3s ease-in-out;
  transition-delay: 1s;
}
.overlay-enter {
  opacity: 0;
  position: absolute;
  transform: scale(0);
  transform: rotate(90deg);
}
.msgInput-enter-active,
.msgInput-leave-active {
  transition: all 0.3s ease-in-out;
}
.msgInput-enter,
.msgInput-leave-to {
  opacity: 0;
  transform: translateX(-50px);
}

.countDown-enter-active {
  transition: all 0.35s ease-in-out;
}
.countDown-enter {
  opacity: 0;
  transform: rotate3d(0, 1, 0, -180deg);
}
.bbtn {
  width: 40%;
  margin: 20px 10px;
  padding: 0;
}
.playersList {
  display: flex;
  align-items: center;
  justify-content: center;
  margin: auto;
  margin-bottom: 20px;
  height: 100px;
}
.overlay img {
  object-fit: cover;
  width: 100%;
  height: 100%;
  opacity: 0.7;
}
.overlay {
  position: relative;
  top: -100%;
  background-color: rgba(0, 0, 0, 0.2);
  z-index: 1;
}

#board {
  background: rgba(0, 0, 0, 0.1);
  margin: auto;
  margin-bottom: 10px;
  width: 350px;
  height: 350px;

  display: flex;
  flex-wrap: wrap;

  border-style: solid;
  border-width: 1px;
  border-color: black;
  overflow: hidden;
}

h3 {
  overflow: hidden;
  background: linear-gradient(90deg, #1a535c, #4ecdc4, #1a535c);
  background-repeat: no-repeat;
  background-size: 85%;
  animation: textAnimate 2s linear infinite;
  -webkit-background-clip: text;
  -webkit-text-fill-color: rgba(255, 255, 255, 0.2);
}
h1 {
  margin: 10px 0;
}
@keyframes textAnimate {
  0% {
    background-position: -500%;
  }
  100% {
    background-position: 500%;
  }
}
</style>
