<template>
	<div class="screenWrapper">
		<h1>Simon Game!</h1>
		<div class="sGameWrapper">
			<div
				class="sButton"
				v-for="sButton in sButtons"
				:key="sButton.id"
				:class="{ [sButton.class]: true, active: sButton.active }"
				@click="buttonClicked(sButton.id)"
			></div>

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
		userSequence() {
			return [];
		},
		gameButtons() {
			return this.sButtons.map((gameButton) => {
				return gameButton;
			});
		},
		timeout() {
			switch (this.level) {
				case 0:
					return 1.5;
				case 1:
					return 1;
				case 2:
					return 0.4;
				default:
					return 1.5;
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
			this.gameButtons.forEach((gameButton) => {
				if (gameButton.id === value) {
					this.gameButtons[gameButton.id].active = true;
					this.playAudio(gameButton.id);
					setTimeout(() => {
						this.gameButtons[gameButton.id].active = false;
					}, (this.timeout / 2) * 1000);
				} else this.gameButtons[gameButton.id].active = false;
			});
		},
		activateButtons() {
			this.setIsButtonActive(true);
			this.gameSequence.forEach((number, index) => {
				setTimeout(() => {
					this.activateButton(number);
				}, this.timeout * 1000 * (index + 1));
			});
			setTimeout(() => {
				this.setIsButtonActive(false);
			}, this.timeout * 1000 * (this.gameSequence.length + 0.6));
		},
	},
};
</script>

<style lang="scss">
$btnGap: 5px;
* {
	--btnSize: 250px;
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
	cursor: pointer;
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
	width: 80%;
	margin-top: $btnGap;
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
