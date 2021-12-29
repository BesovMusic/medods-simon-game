<template>
	<div class="screenWrapper">
		<h1>Simon Game!</h1>
		<div class="sGameWrapper">
			<button
				class="sButton"
				v-for="sButton in sButtons"
				:key="sButton.id"
				:class="{ [sButton.class]: true, active: sButton.active }"
				@click="buttonClicked(sButton.id)"
			></button>

			<div class="scoreScreen">
				<span v-show="isGameOn">Round: {{ round }}</span>
				<button v-show="!isGameOn" @click="startGame" class="menuBtn">
					New game
				</button>
				<button v-show="!isGameOn" class="menuBtn" @click="switchLevel">
					{{ levels[level] }}
				</button>
			</div>
		</div>
	</div>
</template>

<script>
export default {
	name: 'SimonBoard',
	data() {
		return {
			round: 0,
			isGameOn: false,
			isGameFail: false,
			isButtonActive: false,
			sButtons: [
				{ id: 0, class: 'sButton--topLeft', active: false },
				{ id: 1, class: 'sButton--topRight', active: false },
				{ id: 2, class: 'sButton--botLeft', active: false },
				{ id: 3, class: 'sButton--botRight', active: false },
			],
			gameSequence: [],
			userSequence: [],
			levels: ['easy', 'medium', 'hard'],
			level: 0,
			sounds: [
				'https://s3.amazonaws.com/freecodecamp/simonSound1.mp3',
				'https://s3.amazonaws.com/freecodecamp/simonSound2.mp3',
				'https://s3.amazonaws.com/freecodecamp/simonSound3.mp3',
				'https://s3.amazonaws.com/freecodecamp/simonSound4.mp3',
			],
		};
	},
	computed: {
		timeout() {
			switch (this.level) {
				case 0:
					return 1500;
				case 1:
					return 1000;
				case 2:
					return 400;
				default:
					return 1500;
			}
		},
	},
	methods: {
		playAudio(index) {
			const audio = new Audio(this.sounds[index]);
			audio.play();
		},
		switchLevel() {
			if (this.level < this.levels.length - 1) {
				this.level++;
			} else {
				this.level = 0;
			}
		},
		buttonClicked(id) {
			if (this.isButtonActive) return;
			if (!this.isGameOn) return;
			this.setUserSequence(id);
			this.setIsFail(id);
			this.activateButton(id);
			if (
				!this.isGameFail &&
				this.gameSequence.length === this.userSequence.length
			) {
				this.startNextRound();
			}
		},
		getRandomNumber() {
			return Math.floor(Math.random() * Math.floor(this.sButtons.length));
		},
		setIsGameOn(value) {
			this.isGameOn = value;
		},
		startGame() {
			this.clearUserSequence();
			this.pushSequence();
			this.setIsGameOn(true);
			this.round = 1;
			this.isGameFail = false;
			this.activateButtons();
		},
		pushSequence() {
			this.gameSequence.push(this.getRandomNumber());
		},
		startNextRound() {
			setTimeout(() => {
				this.pushSequence();
				this.clearUserSequence();
				this.round++;
				this.activateButtons();
			}, 500);
		},
		setUserSequence(value) {
			this.userSequence.push(value);
			console.log(this.userSequence);
		},
		setIsFail(id) {
			if (this.gameSequence[this.userSequence.length - 1] !== id) {
				this.gameFail();
			}
		},
		gameFail() {
			this.isGameFail = true;
			this.setIsGameOn(false);
			this.gameSequence.length = 0;
		},
		clearUserSequence() {
			this.userSequence.length = 0;
		},
		setIsButtonActive(value) {
			this.isButtonActive = value;
		},
		activateButton(value) {
			this.sButtons.forEach((gameButton) => {
				if (gameButton.id === value) {
					this.sButtons[gameButton.id].active = true;
					this.playAudio(gameButton.id);
					setTimeout(() => {
						this.sButtons[gameButton.id].active = false;
					}, this.timeout / 2);
				} else this.sButtons[gameButton.id].active = false;
			});
		},
		activateButtons() {
			this.setIsButtonActive(true);
			this.gameSequence.forEach((number, index) => {
				setTimeout(() => {
					this.activateButton(number);
				}, this.timeout * (index + 1));
			});
			setTimeout(() => {
				this.setIsButtonActive(false);
			}, this.timeout * (this.gameSequence.length + 0.6));
		},
	},
};
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@400;700&display=swap');

$btnGap: 5px;
* {
	--btnSize: 250px;
	font-family: 'Montserrat', sans-serif;
}

.screenWrapper {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	margin: 5px;
}

.sGameWrapper {
	display: flex;
	flex-wrap: wrap;
	justify-content: center;
	align-items: center;
	height: 100%;
	width: calc((var(--btnSize) * 2) + $btnGap);
	gap: $btnGap;
	position: relative;
}
.sButton {
	flex: var(--btnSize);
	height: var(--btnSize);
	border: 2px solid black;
	border-radius: 10px;
	transition: 0.15s ease-in opacity;
	&--topLeft {
		background-color: yellow;
		border-top-left-radius: 100%;
	}
	&--topRight {
		background-color: green;
		border-top-right-radius: 100%;
	}
	&--botLeft {
		background-color: blue;
		border-bottom-left-radius: 100%;
	}
	&--botRight {
		background-color: red;
		border-bottom-right-radius: 100%;
	}
}
.active {
	opacity: 0.3;
}
.scoreScreen {
	background-color: #fff;
	border-radius: 100%;
	position: absolute;
	width: calc(var(--btnSize) / 1.5);
	height: calc(var(--btnSize) / 1.5);
	border: 2px solid black;
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;
}
.menuBtn {
	width: calc(100% - $btnGap * 2);
	height: calc(50% - $btnGap);
	border-radius: 50% 50% 0 0 / 100% 100% 0 0;
	font-weight: bold;
	background-color: white;
	border: 2px solid black;
	cursor: pointer;
	&:last-child {
		border-radius: 0 0 50% 50% / 0 0 100% 100%;
	}
	&--green {
		background-color: lightgreen;
	}
}
@media (max-width: 425px) {
	* {
		--btnSize: 150px;
		font-size: 10px;
		$btnSize: 150px;
	}
	h1 {
		font-size: 20px;
	}
}
</style>
