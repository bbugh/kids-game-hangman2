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

		guesses.value.add(letter);

		for (let i = 0; i < word.value.length; i++) {
			if (word.value[i] === letter) {
				guess.value[i] = letter;
			}
		}
	}

	const state = ref<"new" | "playing" | "won" | "lost">("new");

	function newGame() {
		state.value = "new";
		word.value = "";

		// TESTING
		word.value = "BACON";
		startGame();
	}

	function startGame() {
		state.value = "playing";
		word.value = word.value.toUpperCase();
		guess.value = Array(word.value.length).fill("_");
	}
</script>

<template>
	<button @click="newGame">New Game</button>
	<div :style="{ display: word ? 'none' : 'block' }"></div>

	<h1>Hangman</h1>

	<div v-if="state === 'new'">
		<p>Enter a word for the other player to guess</p>
		<input v-model="word" />
		<button @click="startGame">Start Game</button>
	</div>

	<pre style="font-size: 200%"><code>
      _____
      |   O
      |  /|\
      |  / \
    </code></pre>

	<div v-if="state === 'playing'">
		<p style="font-size: 200%">Guess: {{ guess.join(" ") }}</p>
	</div>

	{{ guesses }}

	<div>
		<button
			v-for="letter in POSSIBLE_LETTERS"
			:key="letter"
			@click="onChooseLetter($event)"
			:disabled="guesses.has(letter)"
		>
			{{ letter }}
		</button>
	</div>
</template>
