<template>
  <div id="app">
    <div>
      <div class="squares">
        <div
          class="square"
          @click="clicked(1)"
          :class="{ lit: isLit[1] }"
        ></div>
        <div
          class="square"
          @click="clicked(2)"
          :class="{ lit: isLit[2] }"
        ></div>
        <div
          class="square"
          @click="clicked(3)"
          :class="{ lit: isLit[3] }"
        ></div>
        <div
          class="square"
          @click="clicked(4)"
          :class="{ lit: isLit[4] }"
        ></div>
        <div class="start" @click="startGame">
          {{ centerButton }}
          <span v-show="playing">{{ showScore }}</span>
        </div>
      </div>
    </div>
    <div v-show="playing" class="bottom-bar">
      <p>Mode:</p>
      <button
        @click="
          easy = true;
          medium = false;
          hard = false;
        "
      >
        Easy
      </button>
      <button
        @click="
          hard = false;
          medium = true;
          easy = false;
        "
      >
        Medium
      </button>
      <button
        @click="
          hard = true;
          medium = false;
          easy = false;
        "
      >
        Hard
      </button>
    </div>
  </div>
</template>

<script>
import { Howl } from "howler";

export default {
  name: "App",
  data: function () {
    return {
      centerButton: "START",
      easy: true,
      hard: false,
      medium: false,
      playing: false,
      isClickable: false,
      isWon: false,
      isWrong: false,
      strict: false,
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
    // if strict changes in middle of game, reset it
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
      this.centerButton = "RESET";
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
      // check for incorrect
      for (let i = 0; i < this.playerSequence.length; i++) {
        if (this.playerSequence[i] !== this.sequence[i]) {
          this.centerButton = "Wrong!";
          this.isWrong = true;
          this.isClickable = false;

          setTimeout(() => {
            this.startGame();
          }, 1000);

          this.playerSequence = [];
          this.centerButton = "Wrong!";

          setTimeout(() => {
            this.centerButton = "RESET";
            this.isWrong = false;
          }, 1000);

          this.showSequence(true);
        }
      }
      // if all correct and same length , continue
      if (this.playerSequence.length === this.sequence.length) {
        this.playerSequence = [];
        this.score++;
        // if win condition, show win, dont continue.
        if (this.score === 20) {
          this.centerButton = "Winner!";
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
        // dont add number on incorrect answers
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
$red: #aa2525;
$white: #fff;
$yellow: #aaaa25;
$green: #45aa25;
$blue: #2525aa;

#app {
  display: flex;
  flex-direction: column;
  justify-content: center;

  div {
    margin: 0 auto;
  }

  .squares {
    display: flex;
    flex-wrap: wrap;
    width: 200px;

    .square {
      cursor: pointer;
      height: 100px;
      width: 100px;

      &:hover {
      }

      &:nth-of-type(1) {
        background: $green;
        clip-path: circle(99.8% at 100% 100%);

        &.lit {
          background: lighten($green, 25%);
        }
      }
      &:nth-of-type(2) {
        background: $red;
        clip-path: circle(99.8% at 0 100%);

        &.lit {
          background: lighten($red, 25%);
        }
      }
      &:nth-of-type(3) {
        background: $yellow;
        clip-path: circle(99.8% at 100% 0);

        &.lit {
          background: lighten($yellow, 25%);
        }
      }
      &:nth-of-type(4) {
        background: $blue;
        clip-path: circle(99.8% at 0 0);

        &.lit {
          background: lighten($blue, 25%);
        }
      }
    }
  }

  .start {
    margin-top: 30px;
    padding: 10px;
    border: black 1px solid;
    width: 100px;
    cursor: pointer;
    text-align: center;
    display: flex;
    flex-direction: column;

    &:hover {
      color: orange;
    }

    span {
      font-size: 10px;
      color: orange;
    }
  }

  button {
    padding: 10px;
    border: black 1px solid;
    background: none;
    outline: none;
    width: 100px;
    cursor: pointer;
    text-align: center;

    &:hover {
      color: orange;
    }
  }
}
</style>
