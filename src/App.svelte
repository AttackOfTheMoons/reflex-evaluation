<script lang="ts">
  import { onMount } from "svelte";
  import Box from "./Box.svelte";
  import BrokenHeart from "./BrokenHeart.svelte";

  let boxes: { id: number; isActive: boolean; label: string }[] = [
    { id: 0, isActive: false, label: "1" },
    { id: 1, isActive: false, label: "2" },
  ];

  const SUCCESSES_NEEDED = 4;

  let selectedBoxId: number;
  let gameStarted: boolean = false;
  let gamePaused: boolean = false;
  let successCount: number;
  let overallSuccessCount: number = 0;
  let overallFailCount: number = 0;
  let speedMultiplier: number;
  let startTime: number;
  let finalTime: number;
  let instruction: string = "";
  let targetBoxId: number;
  let flickerInterval: number;
  let speedTracker: Array<number> = [];
  let average: number;
  let max: number;
  let min: number;

  $: if (speedMultiplier) {
    clearInterval(flickerInterval);
    flickerInterval = setInterval(() => {
      if (!gamePaused) {
        changeBoxHighlight();
      }
    }, 1000 * speedMultiplier);
  }

  $: if (typeof selectedBoxId === "number") {
    boxes[selectedBoxId].isActive = true;
  }

  $: if (overallFailCount >= 3) {
    const sum = speedTracker.reduce((acc, num) => acc + num, 0);
    average = sum / speedTracker.length;
    max = Math.max(...speedTracker);
    min = Math.min(...speedTracker);
  }

  function getRandomBoxId(): number {
    return Math.floor(Math.random() * boxes.length);
  }

  function changeBoxHighlight() {
    if (typeof selectedBoxId === "number") {
      boxes[selectedBoxId].isActive = false;
      selectedBoxId = (selectedBoxId + 1) % boxes.length;
    } else {
      selectedBoxId = getRandomBoxId();
    }
  }

  function startGame() {
    speedMultiplier = 1;
    successCount = 0;
    gameStarted = true;
    changeBoxHighlight();
    targetBoxId = getRandomBoxId();
    instruction = `Press space when box ${boxes[targetBoxId].label} is highlighted`;
    startTime = Date.now();
  }

  function handleKeyPress(event: KeyboardEvent) {
    if (!gameStarted) {
      startGame();
      return;
    }

    if (event.key === " ") {
      if (gamePaused) {
        // Unpause the game.
        gamePaused = false;
        changeBoxHighlight();
        startTime = Date.now();
        return;
      }
      if (selectedBoxId !== null) {
        let endTime = Date.now();
        finalTime = (endTime - startTime) / 1000;
        if (selectedBoxId === targetBoxId) {
          // Suceeded
          successCount++;
          overallSuccessCount++;
          speedTracker.push(finalTime);
          if (successCount >= SUCCESSES_NEEDED) {
            speedMultiplier *= 0.9;
            successCount = 0;
          }
        } else {
          // Failed
          overallFailCount++;
          gamePaused = true;
        }
        targetBoxId = getRandomBoxId();
        instruction = `Press space when box ${boxes[targetBoxId].label} is highlighted`;
        startTime = Date.now();
      }
    }
  }

  onMount(() => {
    window.addEventListener("keydown", handleKeyPress);

    return () => {
      clearInterval(flickerInterval);
      window.removeEventListener("keydown", handleKeyPress);
    };
  });
</script>

<main>
  {#if !gameStarted}
    <h1>Welcome to the Reaction Test</h1>
    <p>
      Participate in an evaluation that requires selecting objects within
      specified time intervals. Your objective is to react promptly when the
      designated target box becomes active. React swiftly by pressing the
      spacebar when prompted.
    </p>
    <button on:click={startGame}>Initiate Game</button>
  {:else if overallFailCount >= 3}
    <!-- Game Over -->
    <h1>Game Over!</h1>
    <p>
      Overall Accuracy: {(
        (overallSuccessCount / (overallSuccessCount + overallFailCount)) *
        100
      ).toFixed(2)}% ({overallSuccessCount} hits | {overallFailCount} fails)
    </p>
    <p>Final Speed Multiplier: {speedMultiplier.toFixed(2)}x</p>
    {#if speedTracker.length !== 0}
      <p>
        You were able to click boxes on average in {average.toFixed(2)} seconds.
      </p>
      <p>Max speed: {max} seconds. Min speed: {min} seconds</p>
    {:else}
      <p>
        You weren't able to click any of the boxes... How did you even manage to
        open this website?
      </p>
    {/if}
    <button
      on:click={() => {
        location.reload();
      }}>Try Again?</button
    >
  {:else}
    {#if gamePaused}
      <BrokenHeart {overallFailCount} />
    {:else}
      <p>{instruction}</p>
    {/if}
    <p>Current Speed: {speedMultiplier.toFixed(2)}x</p>
    <p>
      Progress: <progress
        value={successCount % SUCCESSES_NEEDED}
        max={SUCCESSES_NEEDED - 1}
      ></progress>
    </p>
    {#each boxes as box (box.id)}
      <Box isActive={box.isActive} label={box.label} />
    {/each}
    {#if finalTime}
      <p>Time taken: {finalTime.toFixed(2)} seconds</p>
    {/if}
  {/if}
</main>

<style>
  main {
    text-align: center;
    margin-top: 50px;
  }
</style>
