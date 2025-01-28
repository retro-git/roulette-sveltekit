<script lang="ts">
  // Define types
  interface Square {
    number: number;
    color: string;
  }

  // Constants
  const SQUARES: Square[] = [
    { number: 5, color: "#FF6B6B" },
    { number: 2, color: "#4ECDC4" },
    { number: 8, color: "#45B7D1" },
    { number: 1, color: "#96CEB4" },
    { number: 4, color: "#FFD93D" },
    { number: 0, color: "#6C5B7B" },
    { number: 3, color: "#87CEEB" },
  ];

  const REPETITIONS = 10;
  const BOX_WIDTH = 64;
  const SPACING = 8;
  const BOX_TOTAL_WIDTH = BOX_WIDTH + SPACING;

  // FSM States and configuration
  type GameState = "countdown" | "rolling" | "completed";
  const COUNTDOWN_SECONDS = 3;
  const COMPLETED_WAIT_SECONDS = 2;

  let gameState = $state<GameState>("countdown");
  let countdownValue = $state(COUNTDOWN_SECONDS);
  let selectedNumber = $state("0");
  let selectedRepetition = $state("0");
  let randomOffset = $state(0);
  let centeredIndex = $state(
    SQUARES.findIndex((square) => square.number === 0)
  );
  let sliderElement = $state<HTMLDivElement | null>(null);
  let animationFrame = $state<number | null>(null);

  // Add a flag to track which transition curve to use
  let isResetting = $state(false);

  // Computed values
  let selectedNumberAsInt = $derived(parseInt(selectedNumber) || 0);
  let selectedRepetitionAsInt = $derived(parseInt(selectedRepetition) || 0);
  let selectedIndex = $derived(
    SQUARES.length * selectedRepetitionAsInt +
      SQUARES.findIndex((square) => square.number === selectedNumberAsInt)
  );
  let transform = $derived(() => {
    return `translateX(${-(selectedIndex * BOX_TOTAL_WIDTH) - BOX_WIDTH / 2 + (isResetting || gameState === "countdown" ? 0 : randomOffset)}px)`;
  });

  function handleInputChange(e: Event): void {
    const input = e.target as HTMLInputElement;
    let value = input.value.replace(/^0+/, "");
    const numValue = parseInt(value) || 0;

    if (input.id === "repetitionInput") {
      if (numValue >= 0 && numValue < REPETITIONS) {
        selectedRepetition = value === "" ? "" : numValue.toString();
        startPositionTracking();
      }
    } else {
      if (SQUARES.some((square) => square.number === numValue)) {
        selectedNumber = value === "" ? "" : numValue.toString();
        startPositionTracking();
      }
    }
  }

  function startPositionTracking(): void {
    if (animationFrame) {
      cancelAnimationFrame(animationFrame);
    }

    function trackPosition(): void {
      if (!sliderElement) return;

      const transform = new WebKitCSSMatrix(
        window.getComputedStyle(sliderElement).transform
      );
      const currentX = transform.m41;

      const adjustedX = -currentX - BOX_WIDTH / 2;
      const closestIndex = Math.round(adjustedX / BOX_TOTAL_WIDTH);

      if (closestIndex >= 0 && closestIndex < SQUARES.length * REPETITIONS) {
        centeredIndex = closestIndex;
      }

      animationFrame = requestAnimationFrame(trackPosition);
    }

    animationFrame = requestAnimationFrame(trackPosition);
  }

  async function runGameLoop() {
    while (true) {
      isResetting = false;
      randomOffset = 0;

      // Countdown state
      gameState = "countdown";
      for (let i = COUNTDOWN_SECONDS; i > 0; i--) {
        countdownValue = i;
        await new Promise((resolve) => setTimeout(resolve, 1000));
      }

      // Rolling state
      gameState = "rolling";
      const randomNumber =
        SQUARES[Math.floor(Math.random() * SQUARES.length)].number;
      const randomRepetition = Math.floor(Math.random() * REPETITIONS);
      randomOffset = (Math.random() - 0.5) * (BOX_WIDTH * 0.8);
      selectedNumber = randomNumber.toString();
      selectedRepetition = randomRepetition.toString();
      startPositionTracking();

      // Wait for animation to complete (5 seconds based on CSS transition)
      await new Promise((resolve) => setTimeout(resolve, 5000));

      // Completed state
      gameState = "completed";
      await new Promise((resolve) =>
        setTimeout(resolve, COMPLETED_WAIT_SECONDS * 1000)
      );

      // Reset to initial state with different curve
      isResetting = true;
      selectedNumber = "0";
      selectedRepetition = "0";
      centeredIndex = SQUARES.findIndex((square) => square.number === 0);

      // Wait for reset animation to complete (2 seconds)
      await new Promise((resolve) => setTimeout(resolve, 2000));
    }
  }

  // Start the game loop when component mounts
  $effect(() => {
    runGameLoop();

    return () => {
      if (animationFrame) {
        cancelAnimationFrame(animationFrame);
      }
    };
  });

  function handleTransitionEnd() {
    if (animationFrame) {
      cancelAnimationFrame(animationFrame);
    }
  }
</script>

<div class="w-full p-4">
  <div class="mb-4 text-center text-2xl font-bold">
    {#if gameState === "countdown"}
      Rolling in {countdownValue}...
    {:else if gameState === "rolling"}
      Rolling...
    {:else}
      Complete!
    {/if}
  </div>

  <div class="mb-4 flex gap-4">
    <div>
      <label for="numberInput" class="block text-sm font-medium mb-2">
        Enter a number ({SQUARES.map((s) => s.number).join(", ")}):
      </label>
      <input
        id="numberInput"
        type="number"
        value={selectedNumber}
        onchange={handleInputChange}
        class="border rounded px-3 py-2 w-24"
      />
    </div>
    <div>
      <label for="repetitionInput" class="block text-sm font-medium mb-2">
        Enter repetition (0-{REPETITIONS - 1}):
      </label>
      <input
        id="repetitionInput"
        type="number"
        value={selectedRepetition}
        onchange={handleInputChange}
        class="border rounded px-3 py-2 w-24"
      />
    </div>
  </div>

  <div class="relative w-full h-24 bg-gray-50">
    <div
      class="absolute left-1/2 top-0 h-full w-0 border-l-2 border-blue-400 -translate-x-1/2"
    ></div>

    <div class="absolute w-full h-full overflow-hidden">
      <div
        bind:this={sliderElement}
        class="absolute flex space-x-2 h-full items-center"
        style="left: 50%; transform: {transform()}; transition: transform {isResetting
          ? '500ms linear'
          : '5000ms cubic-bezier(0.05, 0.7, 0, 1)'}; will-change: transform"
        ontransitionend={handleTransitionEnd}
      >
        {#each Array(REPETITIONS) as _, repIndex}
          {#each SQUARES as square, index}
            <div
              class="
                flex-shrink-0
                flex items-center justify-center
                w-16 h-16 rounded-lg border-2 text-lg font-medium
                {repIndex * SQUARES.length + index === centeredIndex
                ? 'border-blue-500'
                : 'border-gray-300'}
              "
              style:background-color={repIndex * SQUARES.length + index ===
              centeredIndex
                ? square.color + "40"
                : square.color + "20"}
            >
              {square.number}
            </div>
          {/each}
        {/each}
      </div>
    </div>
  </div>
</div>
