<template>
	<div id="app">
		<v-container fluid >
			<v-row style="margin: 0 auto; justify-content: center; flex-wrap: nowrap;">
				<v-col cols="12">
					{{ message }}
				</v-col>
			</v-row>
			<v-row justify="center" align="middle" class="flex-nowrap mx-auto" style="margin: 0 auto; justify-content: center; flex-wrap: nowrap;">
				<v-col cols="12" md="6">
					<v-card class="pa-2" style="padding: 8px;">
						<h1 class="text-center">YOU</h1>
						<div class="healthbar">
							<div
								class="healthbar healthbar-inner text-center"
								style="margin: 0;"
								:style="{width: playerHealth + '%'}"
								:class="{ 'yellow': playerYellow, 'red': playerRed }">
								{{ playerHealth }}
							</div>
						</div>
					</v-card>
				</v-col>
				<v-col cols="12" md="6">
					<v-card style="padding: 8px;">
						<h1 class="text-center">MONSTER</h1>
						<div class="healthbar">
							<div
								class="healthbar healthbar-inner text-center"
								style="margin: 0;"
								:style="{width: monsterHealth + '%'}"
								:class="{ 'yellow': monsterYellow, 'red': monsterRed }">
								{{ monsterHealth }}
							</div>
						</div>
					</v-card>
				</v-col>
			</v-row>
			<v-row justify="center" v-if="!gameIsRunning" style="justify-content:center; margin: 0 auto;">
				<v-col cols="12">
					<v-btn id="start-game" @click="startGame">START NEW GAME</v-btn>
				</v-col>
			</v-row>
			<v-row justify="center" v-else style="justify-content:center; margin: 0 auto;">
				<v-col cold="12">
					<v-btn id="attack" @click="attack">ATTACK</v-btn>
					<v-btn id="special-attack" @click="specialAttack" :disabled=disableSpecial>SPECIAL ATTACK</v-btn>
					<v-btn id="heal" @click="heal">HEAL</v-btn>
					<v-btn id="give-up" @click="giveUp">GIVE UP</v-btn>
				</v-col>
			</v-row>
			<v-row class="log" v-if="turns.length > 0" style="justify-content:center; margin: 0 auto;">
				<v-col cols="12">
					<ul>
					<li v-for="turn in turns"
						:class="{'player-turn': turn.isPlayer, 'monster-turn': !turn.isPlayer}" v-bind:key="turn.id">
						{{ turn.text }}
					</li>
					</ul>
				</v-col>
			</v-row>
		</v-container>
	</div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'app',
  components: {
    // HelloWorld
  },
  data: function() {
    return {
      playerHealth: 100,
      monsterHealth: 100,
      gameIsRunning: false,
	  turns: [],
	  spAttCount: 0,
	  message: '',
	  disableSpecial: false,
	  playerYellow: false,
	  playerRed: false,
	  monsterYellow: false,
	  monsterRed: false
    }
  },
  methods: {
    startGame: function () {
        this.gameIsRunning = true;
        this.playerHealth = 100;
        this.monsterHealth = 100;
		this.turns = [];
		this.spAttCount = 0;
		this.message = '';
		this.disableSpecial = false;
		this.playerYellow = false;
		this.playerRed = false;
		this.monsterYellow = false;
		this.monsterRed = false;
    },
    attack: function () {
		this.message = '';
        var damage = this.calculateDamage(3, 10);
        this.monsterHealth -= damage;
        this.turns.unshift({
            isPlayer: true,
            text: 'Player hits Monster for ' + damage
        });
        if (this.checkWin()) {
            return;
        }
		if (this.monsterHealth < 50 && this.monsterHealth > 24) {
			this.monsterYellow = true;
			this.monsterRed = false;
		}
		else if (this.monsterHealth < 25) {
			this.monsterYellow = false;
			this.monsterRed = true;
		}

        this.monsterAttacks();
    },
    specialAttack: function () {
		if (this.spAttCount < 2) {
			this.spAttCount += 1;
			var damage = this.calculateDamage(10, 30);
			this.monsterHealth -= damage;
			this.turns.unshift({
				isPlayer: true,
				text: 'Player hits Monster hard for ' + damage
			});
			if (this.checkWin()) {
				return;
			}
			if (this.monsterHealth < 50 && this.monsterHealth > 24) {
				this.monsterYellow = true;
				this.monsterRed = false;
			}
			else if (this.monsterHealth < 25) {
				this.monsterYellow = false;
				this.monsterRed = true;
			}

			this.monsterAttacks();

			if (this.spAttCount === 2) {
				this.disableSpecial = true;
			}
		}
		else {
			this.message = 'You can only use the special attack two times';
		}
    },
    heal: function () {
		this.message = '';
        if (this.playerHealth <= 90) {
            this.playerHealth += 10;
        } else {
            this.playerHealth = 100;
        }
        this.turns.unshift({
            isPlayer: true,
            text: 'Player heals for 10'
        });
        this.monsterAttacks();
		if (this.playerHealth > 49) {
			this.playerYellow = false;
			this.playerRed = false;
		}
    },
    giveUp: function () {
		this.message = '';
        this.gameIsRunning = false;
    },
    monsterAttacks: function() {
        var damage = this.calculateDamage(5, 12);
        this.playerHealth -= damage;
        this.checkWin();
        this.turns.unshift({
            isPlayer: false,
            text: 'Monster hits Player for ' + damage
		});
		if (this.playerHealth < 50 && this.playerHealth > 24) {
			this.playerYellow = true;
			this.playerRed = false;
		}
		else if (this.playerHealth < 25) {
			this.playerYellow = false;
			this.playerRed = true;
		}
    },
    calculateDamage: function(min, max) {
        return Math.max(Math.floor(Math.random() * max) + 1, min);
    },
    checkWin: function() {
        if (this.monsterHealth <= 0) {
            if (confirm('You won! New Game?')) {
                this.startGame();
            } else {
                this.gameIsRunning = false;
            }
            return true;
        } else if (this.playerHealth <= 0) {
            if (confirm('You lost! New Game?')) {
                this.startGame();
            } else {
                this.gameIsRunning = false;
            }
            return true;
        }
        return false;
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.text-center {
    text-align: center;
}

.healthbar {
    width: 80%;
    height: 40px;
    background-color: #eee;
    margin: auto;
    transition: width 500ms;
}
.healthbar-inner {
	background-color: green;
	color: white;
}
.healthbar-inner.yellow {
	background-color: yellow;
	color: black;
}
.healthbar-inner.red {
	background-color: red;
}

.controls, .log {
    margin-top: 30px;
    text-align: center;
    padding: 10px;
    border: 1px solid #ccc;
    box-shadow: 0px 3px 6px #ccc;
}

.turn {
    margin-top: 20px;
    margin-bottom: 20px;
    font-weight: bold;
    font-size: 22px;
}

.log ul {
    list-style: none;
    font-weight: bold;
    text-transform: uppercase;
}

.log ul li {
    margin: 5px;
}

.log ul .player-turn {
    color: blue;
    background-color: #e4e8ff;
}

.log ul .monster-turn {
    color: red;
    background-color: #ffc0c1;
}

button {
    font-size: 20px;
    background-color: #eee;
    padding: 12px;
    box-shadow: 0 1px 1px black;
    margin: 10px;
}

#start-game {
    background-color: #aaffb0;
}

#start-game:hover {
    background-color: #76ff7e;
}

#attack {
    background-color: #ff7367;
}

#attack:hover {
    background-color: #ff3f43;
}

#special-attack {
    background-color: #ffaf4f;
}

#special-attack:hover {
    background-color: #ff9a2b;
}

#heal {
    background-color: #aaffb0;
}

#heal:hover {
    background-color: #76ff7e;
}

#give-up {
    background-color: #ffffff;
}

#give-up:hover {
    background-color: #c7c7c7;
}
</style>
