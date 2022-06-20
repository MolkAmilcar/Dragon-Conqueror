<template>
<audio
      ref="audio"
      src="./assets/sounds/bgmusic.mp3"
      preload
      loop
      id="audio"
      muted
    ></audio>
  <div class="wrapper">
    <TheHeader />
    <div class="container">
      <div class="row">
        <div class="col">
          <div class="health">
            <div class="health-bar" :style="playerHealthBar">
              <span class="hp-percent">{{ playerHealth }} %</span>
            </div>
          </div>
          <p class="text-center">Sceptile</p>
          <div class="img-container">
            <img src="./assets/gifs/sceptile.gif" alt="sceptile" />
          </div>
        </div>
        <div class="col">
          <div class="health">
            <div class="health-bar" :style="enemyHealthBar">
              <span class="hp-percent">{{ enemyHealth }} %</span>
            </div>
          </div>
          <p class="text-center">Groudon</p>
          <div class="img-container">
            <img src="./assets/gifs/groudon3d.gif" alt="enemy" />
          </div>
        </div>
      </div>
      <h4 class="text-center controls">Acciones: </h4>
      <div class="row">
        <div class="col controls">
          <button class="control-icon attack" @click="attackenemy">
            <img src="./assets/img/attack.png" alt="attack" />
          </button>

          <button
            class="control-icon"
            @click="specialAttack"
            :disabled="numOfSpecialAttack === 0"
          >
            <img src="./assets/img/sAttack.png" alt="special_attack" />
            <span class="num-of-attempt">{{ numOfSpecialAttack }}</span>
          </button>
          <button
            class="control-icon"
            @click="heal"
            :disabled="numOfHeal === 0"
          >
            <img src="./assets/img/heal.png" alt="heal" /><span
              class="num-of-attempt"
              >{{ numOfHeal }}</span
            >
          </button>
          <button class="control-icon" @click="forfeit">
            <img src="./assets/img/surrender.png" alt="give-up" />
          </button>
        </div>
      </div>
      <div class="row mt-5">
        <div class="container">
          <h4 class="text-center">Battle Logs</h4>
          <ul class="col-md-5 logs">
            <li
              v-for="(logMessage, index) in battleLogMessages"
              :key="index"
              class="text-center"
            >
              <span
                :class="{
                  player: logMessage.attacker === 'player',
                  enemy: logMessage.attacker === 'enemy'
                }"
              >
                <!-- {{ logMessage.attacker === 'player' ? `<img src='./gifs/player1.gif' alt="enemy">` : 'enemy'}} -->
                <img
                  src="./assets/img/smallSceptile.png"
                  alt="player"
                  v-if="logMessage.attacker === 'player'"
                  class="img-icon"
                />
                <img
                  src="./assets/img/smallGroudon.png"
                  alt="player"
                  v-else
                  class="img-icon"
                />
              </span>
              <span v-if="logMessage.actionType === 'heal'">
                Se cura por
                <span class="log-heal">{{ logMessage.actionValue }} hp</span>
              </span>
              <span v-else-if="logMessage.actionType === 'attack'">
                Ataca y hace
                <span class="log-damage"
                  >{{ logMessage.actionValue }} of damage</span
                >
              </span>
              <span v-else-if="logMessage.actionType === 'special-attack'">
                Ataque especial y hace
                <span class="log--damage"
                  >{{ logMessage.actionValue }} of damage</span
                >
              </span>
            </li>
          </ul>
        </div>
      </div>
    </div>
    <div class="overlap" v-if="winner">
      <div class="winner">
        <div class="text-center">
          <h3>Fin del juego</h3>
          <h4 v-if="winner === 'enemy'">Perdiste!</h4>
          <h4 v-else-if="winner === 'player'">Ganaste!</h4>
          <h4 v-else-if="winner === 'draw'">Es un empate!</h4>
          <button @click="startFight" class="btn start-btn mt-4">
            Jugar de nuevo
          </button>
        </div>
      </div>
    </div>
  </div>
</template>


<script>


const getRandomValue = (min, max) => {
  return Math.floor(Math.random() * (max - min)) + min;
};

import TheHeader from "./components/layouts/TheHeader";

export default {
  name: "App",
  components: {
    TheHeader
  },
  data() {
    return {
      playerHealth: 100,
      enemyHealth: 100,
      numOfSpecialAttack: 3,
      numOfHeal: 3,
      winner: null,
      battleLogMessages: [],
      tackle: new Audio(require("./assets/sounds/Tackle.mp3")),
      enemyAttack: new Audio(require("./assets/sounds/Tackle.mp3")),
      healEffect: new Audio(require("./assets/sounds/heal.mp3")),
      specialAttackEffect: new Audio(
        require("./assets/sounds/Absorb.mp3")
      )
    };
  },
  watch: {
    playerHealth(value) {
      if (value <= 0 && this.enemyHealth <= 0) {
        //draw
        this.winner = "draw";
      } else if (value <= 0) {
        this.winner = "enemy";
      }
    },
    enemyHealth(value) {
      if (value <= 0 && this.playerHealth <= 0) {
        //draw
        this.winner = "draw";
      } else if (value <= 0) {
        this.winner = "player";
      }
    }
  },
  computed: {
    playerHealthBar() {
      if (this.playerHealth < 0) {
        return { width: "0%" };
      } else {
        if (this.playerHealth <= 50) {
          return { width: this.playerHealth + "%", background: "red" };
        } else {
          return { width: this.playerHealth + "%" };
        }
      }
    },
    enemyHealthBar() {
      if (this.enemyHealth < 0) {
        return { width: "0%" };
      } else {
        if (this.enemyHealth <= 50) {
          return { width: this.enemyHealth + "%", background: "red" };
        } else {
          return { width: this.enemyHealth + "%" };
        }
      }
    }
  },
  methods: {
    startFight() {
      this.playerHealth = 100;
      this.enemyHealth = 100;
      this.numOfSpecialAttack = 3;
      this.numOfHeal = 3;
      this.winner = null;
      this.battleLogMessages = [];
    },
    attackenemy() {
      const attackDamage = getRandomValue(6, 15);
      // check if enemy health is greater than zero first
      if (this.enemyHealth - attackDamage < 0) {
        this.enemyHealth = 0;
      } else {
        this.enemyHealth -= attackDamage;
      }
      this.tackle.play();
      console.log(this.tackle);
      this.addBattleLog("player", "attack", attackDamage);
      this.attackplayer();
    },
    attackplayer() {
      const attackDamage = getRandomValue(8, 20);
      if (this.playerHealth - attackDamage < 0) {
        this.playerHealth = 0;
      } else {
        this.playerHealth -= attackDamage;
      }
      setTimeout(() => {
        this.enemyAttack.play();
        setTimeout(() => {
          this.enemyAttack.pause();
        }, 2000);
      }, 0);

      this.addBattleLog("enemy", "attack", attackDamage);
    },
    specialAttack() {
      this.numOfSpecialAttack--;
      const attackDamage = getRandomValue(10, 25);
      if (this.enemyHealth - attackDamage < 0) {
        this.enemyHealth = 0;
      } else {
        this.specialAttackEffect.play();
        this.enemyHealth -= attackDamage;
      }
      this.addBattleLog("player", "special-attack", attackDamage);
      this.attackplayer();
    },
    heal() {
      this.numOfHeal--;
      const healValue = getRandomValue(10, 25);
      if (this.playerHealth + healValue > 100) {
        this.playerHealth = 100;
      } else {
        this.playerHealth += healValue;
      }
      this.healEffect.play();
      this.addBattleLog("player", "heal", healValue);
      this.attackplayer();
    },
    forfeit() {
      this.winner = "enemy";
    },
    addBattleLog(who, what, value) {
      this.battleLogMessages.unshift({
        attacker: who,
        actionType: what,
        actionValue: value
      });
    }
  }
};
</script>



<style>

/*Media Queries */
</style>