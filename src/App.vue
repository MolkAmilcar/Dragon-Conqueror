<template>
  <div class="wrapper">
    <TheHeader />
    <div class="container">
      <div class="row">
        <div class="col">
          <div class="health">
            <div class="health-bar" :style="warriorHealthBar">
              <span class="hp-percent">{{ warriorHealth }} %</span>
            </div>
          </div>
          <p class="text-center">Warrior</p>
          <div class="img-container">
            <img src="./assets/gifs/warrior1.gif" alt="warrior" />
          </div>
        </div>
        <div class="col">
          <div class="health">
            <div class="health-bar" :style="dragonHealthBar">
              <span class="hp-percent">{{ dragonHealth }} %</span>
            </div>
          </div>
          <p class="text-center">Dragon</p>
          <div class="img-container">
            <img src="./assets/gifs/dragon1.gif" alt="dragon" />
          </div>
        </div>
      </div>
      <div class="row">
        <div class="col controls">
          <button class="control-icon attack" @click="attackDragon">
            <img src="./assets/img/attack_icon.svg" alt="attack" />
          </button>

          <button
            class="control-icon"
            @click="specialAttack"
            :disabled="numOfSpecialAttack === 0"
          >
            <img src="./assets/img/special_attack.svg" alt="special_attack" />
            <span class="num-of-attempt">{{ numOfSpecialAttack }}</span>
          </button>
          <button
            class="control-icon"
            @click="heal"
            :disabled="numOfHeal === 0"
          >
            <img src="./assets/img/heal_icon.png" alt="heal" /><span
              class="num-of-attempt"
              >{{ numOfHeal }}</span
            >
          </button>
          <button class="control-icon" @click="forfeit">
            <img src="./assets/img/surrender_icon.png" alt="give-up" />
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
                  warrior: logMessage.attacker === 'warrior',
                  dragon: logMessage.attacker === 'dragon'
                }"
              >
                <!-- {{ logMessage.attacker === 'warrior' ? `<img src='./gifs/warrior1.gif' alt="dragon">` : 'dragon'}} -->
                <img
                  src="./assets/gifs/warrior1.gif"
                  alt="warrior"
                  v-if="logMessage.attacker === 'warrior'"
                  class="img-icon"
                />
                <img
                  src="./assets/gifs/dragon1.gif"
                  alt="warrior"
                  v-else
                  class="img-icon"
                />
              </span>
              <span v-if="logMessage.actionType === 'heal'">
                heals himself for
                <span class="log-heal">{{ logMessage.actionValue }} hp</span>
              </span>
              <span v-else-if="logMessage.actionType === 'attack'">
                attack and deals
                <span class="log-damage"
                  >{{ logMessage.actionValue }} of damage</span
                >
              </span>
              <span v-else-if="logMessage.actionType === 'special-attack'">
                use special attack and deals
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
          <h3>Game Over!</h3>
          <h4 v-if="winner === 'dragon'">You Lose!</h4>
          <h4 v-else-if="winner === 'warrior'">You Win!</h4>
          <h4 v-else-if="winner === 'draw'">It's a Draw!</h4>
          <button @click="startFight" class="btn start-btn mt-4">
            Battle Again
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
      warriorHealth: 100,
      dragonHealth: 100,
      numOfSpecialAttack: 3,
      numOfHeal: 3,
      winner: null,
      battleLogMessages: [],
      swordSlash: new Audio(require("./assets/sounds/sword_slash.mp3")),
      dragonBreath: new Audio(require("./assets/sounds/dragon_breath.mp3")),
      healEffect: new Audio(require("./assets/sounds/heal_effect.mp3")),
      specialAttackEffect: new Audio(
        require("./assets/sounds/special_attack.mp3")
      )
    };
  },
  watch: {
    warriorHealth(value) {
      if (value <= 0 && this.dragonHealth <= 0) {
        //draw
        this.winner = "draw";
      } else if (value <= 0) {
        this.winner = "dragon";
      }
    },
    dragonHealth(value) {
      if (value <= 0 && this.warriorHealth <= 0) {
        //draw
        this.winner = "draw";
      } else if (value <= 0) {
        this.winner = "warrior";
      }
    }
  },
  computed: {
    warriorHealthBar() {
      if (this.warriorHealth < 0) {
        return { width: "0%" };
      } else {
        if (this.warriorHealth <= 50) {
          return { width: this.warriorHealth + "%", background: "red" };
        } else {
          return { width: this.warriorHealth + "%" };
        }
      }
    },
    dragonHealthBar() {
      if (this.dragonHealth < 0) {
        return { width: "0%" };
      } else {
        if (this.dragonHealth <= 50) {
          return { width: this.dragonHealth + "%", background: "red" };
        } else {
          return { width: this.dragonHealth + "%" };
        }
      }
    }
  },
  methods: {
    startFight() {
      this.warriorHealth = 100;
      this.dragonHealth = 100;
      this.numOfSpecialAttack = 3;
      this.numOfHeal = 3;
      this.winner = null;
      this.battleLogMessages = [];
    },
    attackDragon() {
      const attackDamage = getRandomValue(6, 15);
      // check if dragon health is greater than zero first
      if (this.dragonHealth - attackDamage < 0) {
        this.dragonHealth = 0;
      } else {
        this.dragonHealth -= attackDamage;
      }
      this.swordSlash.play();
      console.log(this.swordSlash);
      this.addBattleLog("warrior", "attack", attackDamage);
      this.attackWarrior();
    },
    attackWarrior() {
      const attackDamage = getRandomValue(8, 20);
      if (this.warriorHealth - attackDamage < 0) {
        this.warriorHealth = 0;
      } else {
        this.warriorHealth -= attackDamage;
      }
      setTimeout(() => {
        this.dragonBreath.play();
        setTimeout(() => {
          this.dragonBreath.pause();
        }, 2000);
      }, 0);

      this.addBattleLog("dragon", "attack", attackDamage);
    },
    specialAttack() {
      this.numOfSpecialAttack--;
      const attackDamage = getRandomValue(10, 25);
      if (this.dragonHealth - attackDamage < 0) {
        this.dragonHealth = 0;
      } else {
        this.specialAttackEffect.play();
        this.dragonHealth -= attackDamage;
      }
      this.addBattleLog("warrior", "special-attack", attackDamage);
      this.attackWarrior();
    },
    heal() {
      this.numOfHeal--;
      const healValue = getRandomValue(10, 25);
      if (this.warriorHealth + healValue > 100) {
        this.warriorHealth = 100;
      } else {
        this.warriorHealth += healValue;
      }
      this.healEffect.play();
      this.addBattleLog("warrior", "heal", healValue);
      this.attackWarrior();
    },
    forfeit() {
      this.winner = "dragon";
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