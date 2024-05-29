<script setup lang="ts">
	import { ref, onMounted, onUnmounted } from "vue";

	import data from "./data2.json";

	const word = ref("");
	const guess = ref<string[]>([]);
	const guesses = ref(new Set<string>());
	const hint = ref("");

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

	function onChooseLetter(letter: string) {
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
		hint.value = "";
	}

	function startGame() {
		state.value = "playing";
		word.value = word.value.toUpperCase();

		guess.value = Array(word.value.length);

		for (let i = 0; i < word.value.length; i++) {
			if (POSSIBLE_LETTERS.includes(word.value[i])) {
				guess.value[i] = "_";
			} else {
				guess.value[i] = word.value[i];
			}
		}

		guesses.value = new Set<string>();
	}

	function randomWord() {
		const categories = Object.keys(data.words);
		const randomCategory = categories[Math.floor(Math.random() * categories.length)];
		hint.value = randomCategory;
		const randomWord = (data.words as any)[randomCategory][
			Math.floor(Math.random() * (data.words as any)[randomCategory].length)
		];
		word.value = randomWord.toUpperCase();
	}

	function startGameRandomWord() {
		state.value = "playing";
		randomWord();

		guess.value = Array(word.value.length);

		for (let i = 0; i < word.value.length; i++) {
			if (POSSIBLE_LETTERS.includes(word.value[i])) {
				guess.value[i] = "_";
			} else {
				guess.value[i] = word.value[i];
			}
		}

		guesses.value = new Set<string>();
	}

	function onKeyDown(event: KeyboardEvent) {
		if (state.value !== "playing") return;

		const letter = event.key.toUpperCase();
		if (!POSSIBLE_LETTERS.includes(letter)) return;

		if (guesses.value.has(letter)) return;

		onChooseLetter(letter);
	}

	onMounted(() => {
		window.addEventListener("keydown", onKeyDown);
	});

	onUnmounted(() => {
		window.removeEventListener("keydown", onKeyDown);
	});
</script>

<template>
	<main style="font-size: 20px">
		<button @click="newGame">New Game</button>
		<div :style="{ display: word ? 'none' : 'block' }"></div>

		<h1 style="margin-bottom: 0">Hangman 2</h1>

		<div v-if="state === 'new'" style="display: flex; gap: 1em">
			<div>
				<button @click="startGameRandomWord">Play With a Random Word</button>
			</div>
			<div>OR</div>
			<div>
				<form @submit.prevent="startGame">
					<input v-model="word" placeholder="Enter a word to guess" />
					<button type="submit">Start</button>
				</form>
			</div>
		</div>

		<div v-if="state !== 'new'">
			<pre style="margin: 0"><code>
      _____
      |   <span v-if="guesses.size > 0">O</span>
      |  <span v-if="guesses.size > 1">/</span><span v-if="guesses.size > 2">|</span><span v-if="guesses.size > 3">\</span>
      |  <span v-if="guesses.size > 4">/</span> <span v-if="guesses.size > 5">\</span>
    </code></pre>

			<div style="font-size: 1.5em">
				Guess ({{ 6 - guesses.size }} left):
				<div style="display: flex; flex-wrap: wrap; gap: 0.5em">
					<div
						v-for="(letter, index) in guess"
						:key="index"
						style="
							border: 1px solid gray;
							border-radius: 3px;
							width: 50px;
							height: 50px;
							display: flex;
							align-items: center;
							justify-content: center;
						"
					>
						{{ letter }}
					</div>
				</div>
				<div v-if="hint" style="font-size: 0.75em; margin-top: 10px">hint: {{ hint }}</div>
			</div>

			<p style="font-size: 1.5em; margin-bottom: 10px">Letters:</p>
			<small>Click on a letter or type on your keyboard</small>
			<div style="display: flex; flex-wrap: wrap; gap: 1em">
				<button
					v-for="letter in POSSIBLE_LETTERS"
					:key="letter"
					@click="onChooseLetter(letter)"
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
		</div>
	</main>
</template>

<style scoped>
	button:disabled {
		cursor: not-allowed;
		opacity: 0.5;
	}
</style>
