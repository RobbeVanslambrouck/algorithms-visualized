<script lang="ts">
	import { randInt } from '$lib/utils';
	import { flip } from 'svelte/animate';
	import { sineInOut } from 'svelte/easing';
	import { onMount } from 'svelte';

	$: values = [1, 2, 3, 4, 5, 6, 7, 8, 9];
	let stepTime = 500;
	let shuffleTime = 1000;
	let sort: string;
	$: stepper = getStepper(sort);
	let frameId: number;

	onMount(() => {
		startStepper(shuffle());
		return () => cancelAnimationFrame(frameId);
	});

	function getStepper(sort: string) {
		let stepper: Generator;
		switch (sort) {
			case 'selection sort':
				return selectionSort();
			case 'bubble sort':
				return bubbleSort();
			default:
				return selectionSort();
		}
	}

	function* shuffle() {
		for (let i = 0; i < values.length - 1; i++) {
			const randIndex = randInt(i, values.length - 1);
			yield swap(i, randIndex);
		}
	}

	function swap(index1: number, index2: number) {
		[values[index1], values[index2]] = [values[index2], values[index1]];
	}

	function startStepper(stepper: Generator, stepTime = 0) {
		cancelAnimationFrame(frameId);
		frameId = requestAnimationFrame(step);
		let lastUpdateTime = 0;
		function step(time: DOMHighResTimeStamp) {
			frameId = requestAnimationFrame(step);
			if (time - lastUpdateTime <= stepTime) return;
			lastUpdateTime = time;
			const { done } = stepper.next();
			if (done) {
				cancelAnimationFrame(frameId);
			}
		}
	}

	function* selectionSort() {
		for (let i = 0; i < values.length; i++) {
			let minIndex = i;
			for (let j = i + 1; j < values.length; j++) {
				if (values[j] < values[minIndex]) {
					minIndex = j;
				}
			}
			yield swap(i, minIndex);
		}
	}

	function* bubbleSort() {
		let swapped;
		do {
			swapped = false;
			for (let i = 0; i < values.length - 1; i++) {
				if (values[i] > values[i + 1]) {
					swapped = true;
					yield swap(i, i + 1);
				}
			}
		} while (swapped);
	}

	function handleOnClickSort() {
		startStepper(stepper, stepTime === 0 ? 0 : stepTime + 50);
	}
</script>

<h2>sorting algorithms</h2>
<form action="">
	<label for="">choose a sorting algorithm</label>
	<select name="" id="" bind:value={sort}>
		<option value="selection sort" default>selection sort</option>
		<option value="bubble sort">bubble sort</option>
	</select>
</form>
<h3>{sort}</h3>
<button on:click={handleOnClickSort}>Sort</button>
<button
	on:click={() => {
		startStepper(shuffle(), shuffleTime === 0 ? shuffleTime : shuffleTime / values.length);
	}}>shuffle</button
>

<div>
	{#each values as value (value)}
		<div
			class="value"
			style="height: {value * 20}px; background-color: blue;"
			animate:flip={{ duration: stepTime, easing: sineInOut }}
		/>
	{/each}
</div>

<style>
	.value {
		display: inline-block;
		width: 20px;
		margin-right: 5px;
	}
</style>
