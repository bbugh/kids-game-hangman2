<script setup lang="ts">
	import { ref } from "vue";

	const word = ref("");
	const guess = ref<string[]>([]);
	const guesses = ref(new Set<string>());
	const POSSIBLE_LETTERS = [
		"A",
		"B",
		"C",
		"D",
		"E",
		"F",
		"G",
		"H",
		"I",
		"J",
		"K",
		"L",
		"M",
		"N",
		"O",
		"P",
		"Q",
		"R",
		"S",
		"T",
		"U",
		"V",
		"W",
		"X",
		"Y",
		"Z",
	];

	function onChooseLetter(event: MouseEvent) {
		const button = event.target as HTMLButtonElement;
		const letter = button.textContent;
		if (!letter) throw new Error("No letter found");

		if (guesses.value.has(letter)) return;

		const isCorrect = word.value.includes(letter);

		if (!isCorrect) {
			guesses.value.add(letter);
			if (guesses.value.size >= 6) {
				state.value = "lost";
			}
			return;
		}

		for (let i = 0; i < word.value.length; i++) {
			if (word.value[i] === letter) {
				guess.value[i] = letter;
			}
		}

		if (guess.value.join("") === word.value) {
			state.value = "won";
		}
	}

	const state = ref<"new" | "playing" | "won" | "lost">("new");

	function newGame() {
		state.value = "new";
		word.value = "";
		guess.value = [];
		guesses.value = new Set<string>();
	}

	function startGame() {
		state.value = "playing";
		word.value = word.value.toUpperCase();
		guess.value = Array(word.value.length).fill("_");
		guesses.value = new Set<string>();
	}
</script>

<template>
	<main style="font-size: 20px">
		<button @click="newGame">New Game</button>
		<div :style="{ display: word ? 'none' : 'block' }"></div>

		<h1>Hangman 2</h1>

		<div v-if="state === 'new'">
			<p>Enter a word for the other player to guess</p>
			<form @submit.prevent="startGame">
				<input v-model="word" />
				<button type="submit">Start Game</button>
			</form>
		</div>

		<pre><code>
      _____
      |   <span v-if="guesses.size > 0">O</span>
      |  <span v-if="guesses.size > 1">/</span><span v-if="guesses.size > 2">|</span><span v-if="guesses.size > 3">\</span>
      |  <span v-if="guesses.size > 4">/</span> <span v-if="guesses.size > 5">\</span>
    </code></pre>

		<div v-if="state !== 'new'" style="font-size: 1.5em">
			<p>Guess: {{ guess.join(" ") }}</p>
		</div>

		<div v-if="state === 'playing'" style="display: flex; flex-wrap: wrap; gap: 1em">
			<button
				v-for="letter in POSSIBLE_LETTERS"
				:key="letter"
				@click="onChooseLetter($event)"
				:disabled="guesses.has(letter)"
				:style="{
					fontSize: '24px',
					minWidth: '4ch',
					padding: '0.5em',
					textAlign: 'center',
					color: guess.includes(letter) ? 'green' : 'black',
					borderColor: guess.includes(letter) ? 'green' : '',
					backgroundColor: guess.includes(letter) ? 'lightgreen' : '',
				}"
			>
				{{ letter }}
			</button>
		</div>

		<div v-if="state === 'won'">
			<p style="color: green">You won! 👏🏻👏🏻👏🏻</p>

			<button @click="newGame">New Game</button>
		</div>

		<div v-if="state === 'lost'">
			<p style="color: red">You lost! 😭 The word was {{ word }}</p>

			<button @click="newGame">New Game</button>
		</div>
	</main>
</template>

<style scoped>
	button:disabled {
		cursor: not-allowed;
		opacity: 0.5;
	}
</style>
