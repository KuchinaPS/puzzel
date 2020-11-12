<template>
  <div>
    <div class="container">
      <div class="header">
        <button class="restart" @click="newGame"></button>
        <div :class="changeNum" class="time_output">{{ calcTime }}</div>
        <div class="count">Ход: {{ count }}</div>
      </div>
      <transition name="fade">
        <div v-if="winStatus">
          <div class="modal">
            <h2>Вы выиграли!</h2>
            <div class="statistic">
              <div class="count">Ход: {{ count }}</div>
              <div :class="changeNum" class="time_output">Время: {{ time }}</div>
            </div>
            <button class="restart new-game" @click="newGame">Новая игра</button>
          </div>
        </div>
      </transition>
      <div class="wrap">
        <transition-group name="cell">
          <div @click="onChange(item)" v-for="item in items" :key="item"
               :class="{ active : item == null, item:'item' }">
            {{ item }}
          </div>
        </transition-group>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'App',
  data: function () {
    return {
      items: [],
      etalon: [],
      count: 0,
      winStatus: false,
      tics: 0,
      timerID: 0,
      isRunning: false,
      time: 0,
      cells: [],
    }
  },
  mounted() {
    this.cells = this.unSort(16);
    this.items = this.unSort(16);
    this.etalon = this.generateEtalon(15);
    while (this.solvable(this.items) !== true) {
      this.items = this.unSort(16);
    }
    this.timerOn();
  },
  watch: {
    items(value) {
      let winStatus = true;
      for (let i = 0; i < value.length; i++) {
        if (value[i] !== this.etalon[i]) {
          winStatus = false;
          break;
        }
      }
      this.winStatus = winStatus;
      if (this.winStatus === true) {
        this.timeStop();
      }
    }
  },
  computed: {
    calcTime: function () {
      const second = this.tics % 60;
      const minutes = Math.floor(this.tics / 60);
      const formattedMin = minutes === 0 ? "" : `${minutes % 60}:`;
      const hours = Math.floor(this.tics / 3600);
      const formattedHour = hours === 0 ? "" : `${hours}:`;
      return `${formattedHour}${formattedMin}${second}`;
    },
  },
  methods: {
    onChange(item) {
      let index = this.items.indexOf(item);
      let nullIndex = this.items.indexOf(null);
      if ((this.items[index + 1] === null || this.items[index - 4] === null || this.items[index + 4] === null || this.items[index - 1] === null) && item !== null) {
        const template = this.items[index];
        this.items[index] = this.items[nullIndex];
        this.items[nullIndex] = template;
        this.items = this.items.slice();
        this.count++;
      }
    },
    shuffle: function () {
      this.cells = this.unSort(16);
    },
    unSort(length) {
      const arr = [];
      while (arr.length < length) {
        const r = Math.floor(Math.random() * length) + 1;
        const value = r === length ? null : r
        if (arr.indexOf(value) === -1) {
          arr.push(value);
        }
      }
      return arr;
    },
    solvable(a) {
      for (let kDisorder = 0, i = 1, len = a.length - 1; i < len; i++) {
        for (let j = i - 1; j >= 0; j--) if (a[j] > a[i]) kDisorder++;
        return !(kDisorder % 2);
      }
    },
    newGame() {
      this.timeStop();
      this.items = this.unSort(16);
      this.count = 0;
      this.timerOn();
      this.shuffle()
    },
    generateEtalon(size) {
      let arr = [];
      for (let i = 1; i <= size; i++) {
        arr.push(i);
      }
      arr.push(null);
      return arr;
    },
    timerOn: function () {
      this.timerID = setInterval(this.updateTime, 1000);
      this.isRunning = true;
    },
    updateTime: function () {
      this.tics += 1;
    },
    timeStop: function () {
      this.time = this.calcTime;
      this.tics = 0;
      this.isRunning = false;
      clearTimeout(this.timerID);
    }
  }
}
</script>

<style scoped>
.fade-enter-active{
  transition: opacity 0.5s;
}
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active до версии 2.1.8 */
{
  opacity: 0;
}
.cell-move {
  transition: transform 0.5s;
}
*, *:before, *:after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

.container {
  background-image: url("./assets/img/wood.jpg");
  background-size: cover;
  max-width: 300px;
  margin: 0 auto;
  padding: 25px;
  position: relative;
  font-size: 25px;
}

.header {
  display: flex;
  justify-content: space-between;
  padding: 10px 10px 20px;
}

.wrap {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
}

.modal {
  background-color: rgba(210, 105, 30, 0.9);
  border-radius: 5px;
  border: 1px solid saddlebrown;
  position: absolute;
  width: 100%;
  height: 300px;
  top: 35px;
  left: 0;
  font-size: 25px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  text-align: center;
  padding: 20px 15px;
  z-index: 100;
}

.statistic {
  display: flex;
  justify-content: space-around;
  padding: 20px 15px;
  flex-wrap: wrap;
}

.timer {
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  font-size: 22px;
  line-height: 21px;
}

.item {
  background-image: url("./assets/img/wood-item.webp");
  background-size: cover;
  width: 50px;
  height: 50px;
  border-radius: 5px;
  margin: 5px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 25px;
  cursor: pointer;
}

.active {
  background: transparent;
}

.restart {
  position: relative;
  width: 40px;
  height: 40px;
  background: transparent;
  border: none;
  cursor: pointer;
}

.restart:focus {
  outline: none;
  border: none;
}

.restart:before {
  position: absolute;
  content: "";
  width: 35px;
  height: 35px;
  background-image: url("./assets/img/row-restartt.png");
  background-size: cover;
  top: 0;
  left: 0;
}

.new-game {
  margin: 0 auto;
  padding-top: 34px;
}
</style>

