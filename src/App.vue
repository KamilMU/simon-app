<template>
  <div class="container">
    <h1>Simon Says</h1>
    <div class="simon">
      <ul class="squares">
        <li class="square" @click="clicked(1)" :class="{ lit: isLit[1] }"></li>
        <li class="square" @click="clicked(2)" :class="{ lit: isLit[2] }"></li>
        <li class="square" @click="clicked(3)" :class="{ lit: isLit[3] }"></li>
        <li class="square" @click="clicked(4)" :class="{ lit: isLit[4] }"></li>
      </ul>
    </div>
    <div class="game-info">
      <h2>Round: {{ score }}</h2>
      <button @click="startGame">Start</button>
      <p>{{ messageToUser }}</p>
    </div>
    <div class="game-options">
      <h2>Game Options:</h2>
      <input
        type="radio"
        name="mode"
        checked
        @change="
          easy = true;
          medium = false;
          hard = false;
        "
      />
      Easy
      <br />
      <input
        type="radio"
        name="mode"
        @change="
          easy = false;
          medium = true;
          hard = false;
        "
      />
      Medium
      <br />
      <input
        type="radio"
        name="mode"
        @change="
          easy = false;
          medium = false;
          hard = true;
        "
      />
      Hard
      <br />
    </div>
  </div>
</template>

<script>
import { Howl } from "howler";

export default {
  name: "App",
  data: function () {
    return {
      messageToUser: "",
      easy: true,
      hard: false,
      medium: false,
      playing: false,
      isClickable: false,
      isWon: false,
      isWrong: false,
      score: 0,
      sequence: [],
      sequenceInterval: null,
      playerSequence: [],
      sounds: [
        new Howl({
          src: ["https://s3.amazonaws.com/freecodecamp/simonSound1.mp3"],
        }),
        new Howl({
          src: ["https://s3.amazonaws.com/freecodecamp/simonSound2.mp3"],
        }),
        new Howl({
          src: ["https://s3.amazonaws.com/freecodecamp/simonSound3.mp3"],
        }),
        new Howl({
          src: ["https://s3.amazonaws.com/freecodecamp/simonSound4.mp3"],
        }),
      ],
      isLit: {
        1: false,
        2: false,
        3: false,
        4: false,
      },
    };
  },
  computed: {
    showScore() {
      if (this.isWon) {
        return "Play Again?";
      }
      return "Score: " + this.score;
    },
  },
  watch: {
    easy() {
      this.startGame();
    },
    medium() {
      this.startGame();
    },
    hard() {
      this.startGame();
    },
  },
  methods: {
    startGame() {
      this.playing = true;
      this.sequence = [];
      this.playerSequence = [];
      this.messageToUser = "";
      this.isWon = false;
      this.isWrong = false;
      this.score = 0;
      clearInterval(this.sequenceInterval);
      this.showSequence();
    },
    clicked(tile) {
      if (this.isClickable) {
        this.playSound(tile);
        this.lightUp(tile);
        this.playerSequence.push(tile);
        this.checkWinLose();
      }
    },
    checkWinLose() {
      for (let i = 0; i < this.playerSequence.length; i++) {
        if (this.playerSequence[i] !== this.sequence[i]) {
          this.messageToUser = `Sorry, you lost after ${this.score} rounds`;
          this.isWrong = true;
          this.isClickable = false;
          this.playerSequence = [];
          this.showSequence(true);

          setTimeout(() => {
            this.startGame();
          }, 2000);

          setTimeout(() => {
            this.messageToUser = "";
            this.isWrong = false;
          }, 1000);
        }
      }

      if (this.playerSequence.length === this.sequence.length) {
        this.playerSequence = [];
        this.score++;

        if (this.score === 20) {
          this.messageToUser = "Winner!";
          this.isClickable = false;
          this.isWon = true;
        } else {
          this.showSequence();
        }
      }
    },
    lightUp(tile) {
      this.isLit[tile] = true;
      setTimeout(() => {
        this.isLit[tile] = false;
      }, 300);
    },
    playSound(tile) {
      this.sounds[tile - 1].play();
    },
    showSequence(redo) {
      let currentIndex = 0;
      let speed;

      if (this.hard) {
        speed = 400;
      } else if (this.medium) {
        speed = 1000;
      } else {
        speed = 1500;
      }

      this.isClickable = false;
      if (!redo) {
        this.sequence.push(Math.floor(Math.random() * 4 + 1));
      }
      this.sequenceInterval = setInterval(() => {
        if (currentIndex >= this.sequence.length) {
          clearInterval(this.sequenceInterval);
          return (this.isClickable = true);
        }
        this.playSound(this.sequence[currentIndex]);
        this.lightUp(this.sequence[currentIndex]);
        currentIndex++;
      }, speed);
    },
  },
};
</script>

<style lang="scss">
$bg-color: #343434;
$red: #ff5643;
$white: #fff;
$yellow: #feef33;
$green: #bede15;
$blue: dodgerblue;

@mixin square {
  height: 290px;
  -webkit-border-radius: 150px 150px 150px 150px;
  border-radius: 150px 150px 150px 150px;
  position: absolute;
  text-indent: 10000px;
  opacity: 0.6;
}

body {
  h1 {
    margin: 1em 0 2em;
    text-align: center;
  }

  input {
    cursor: pointer;
  }

  .container {
    width: 540px;
    margin: 0 auto;

    .simon {
      background: #fff;
      position: relative;
      float: left;
      margin-right: 3em;
      width: 302px;
      height: 295px;
      -webkit-border-radius: 150px 150px 150px 150px;
      border-radius: 150px 150px 150px 150px;
      -moz-box-shadow: 2px 1px 12px #aaa;
      -webkit-box-shadow: 2px 1px 12px #aaa;
      -o-box-shadow: 2px 1px 12px #aaa;
      box-shadow: 2px 1px 12px #aaa;

      .squares {
        margin: 0;
        padding: 0;
        list-style: none;

        .square {
          cursor: pointer;
          @include square;

          &:hover {
            border: 2px solid black;
          }

          &.lit {
            opacity: 1;
          }

          &:nth-of-type(1) {
            background: $blue;
            clip: rect(0px, 150px, 150px, 0px);
            width: 300px;
          }
          &:nth-of-type(2) {
            background: $red;
            clip: rect(0px, 300px, 150px, 150px);
            width: 296px;
          }
          &:nth-of-type(3) {
            background: $yellow;
            clip: rect(150px, 150px, 300px, 0px);
            width: 300px;
          }
          &:nth-of-type(4) {
            background: $green;
            clip: rect(150px, 300px, 300px, 150px);
            width: 296px;
          }
        }
      }
    }

    .game-info {
      margin-top: 90px;

      button {
        width: 5em;
        box-sizing: border-box;
        cursor: pointer;
        outline: none;
        font-size: 1.4em;
        -webkit-border-radius: 10px 10px 10px 10px;
        border-radius: 10px 10px 10px 10px;
        background: #6dabe8;
        border: none;
        padding: 0.3em 0.6em;
      }

      button:active {
        background: #e5e5e5;
        -webkit-box-shadow: inset 0px 0px 5px #c1c1c1;
        -moz-box-shadow: inset 0px 0px 5px #c1c1c1;
        box-shadow: inset 0px 0px 5px #c1c1c1;
        outline: none;
      }
    }
  }
}
</style>
