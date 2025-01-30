<script lang="ts">
  import ChatRoom from "$lib/components/ChatRoom.svelte";
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

  // Separate game configuration
  const GAME_CONFIG = {
    COUNTDOWN_SECONDS: 3,
    COMPLETED_WAIT_SECONDS: 2,
    ROLL_ANIMATION_DURATION: 5000,
    ROLL_ANIMATION_CURVE: "cubic-bezier(0.05, 0.7, 0, 1)",
    RESET_ANIMATION_DURATION: 500,
    RESET_ANIMATION_CURVE: "linear",
  } as const;

  // Game state management
  class GameController {
    private static async countdown(
      setState: (state: GameState, value?: number) => void
    ) {
      setState("countdown");
      for (let i = GAME_CONFIG.COUNTDOWN_SECONDS; i > 0; i--) {
        setState("countdown", i);
        await new Promise((resolve) => setTimeout(resolve, 1000));
      }
    }

    private static getRandomSelection() {
      const randomNumber =
        SQUARES[Math.floor(Math.random() * SQUARES.length)].number;
      const randomRepetition = Math.floor(Math.random() * REPETITIONS);
      const randomOffset = (Math.random() - 0.5) * (BOX_WIDTH * 0.8);
      return { randomNumber, randomRepetition, randomOffset };
    }

    static async runGameLoop(
      setState: (state: GameState, value?: number) => void,
      updateSelection: (
        number: string,
        repetition: string,
        offset: number
      ) => void,
      resetSelection: () => void
    ) {
      while (true) {
        // Countdown phase
        await this.countdown(setState);

        // Rolling phase
        setState("rolling");
        const { randomNumber, randomRepetition, randomOffset } =
          this.getRandomSelection();
        updateSelection(
          randomNumber.toString(),
          randomRepetition.toString(),
          randomOffset
        );

        // Wait for roll animation
        await new Promise((resolve) =>
          setTimeout(resolve, GAME_CONFIG.ROLL_ANIMATION_DURATION)
        );

        // Completed phase
        setState("completed");
        await new Promise((resolve) =>
          setTimeout(resolve, GAME_CONFIG.COMPLETED_WAIT_SECONDS * 1000)
        );

        // Reset phase
        resetSelection();
        await new Promise((resolve) =>
          setTimeout(resolve, GAME_CONFIG.RESET_ANIMATION_DURATION)
        );
      }
    }
  }

  // State management
  let gameState = $state<GameState>("countdown");
  let countdownValue = $state<number>(GAME_CONFIG.COUNTDOWN_SECONDS);
  let selectedNumber = $state("0");
  let selectedRepetition = $state("0");
  let randomOffset = $state(0);
  let centeredIndex = $state(
    SQUARES.findIndex((square) => square.number === 0)
  );
  let sliderElement = $state<HTMLDivElement | null>(null);
  let animationFrame = $state<number | null>(null);
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

  function updateGameState(state: GameState, value?: number) {
    gameState = state;
    if (value !== undefined) {
      countdownValue = value;
    }
  }

  function updateSelection(number: string, repetition: string, offset: number) {
    isResetting = false;
    selectedNumber = number;
    selectedRepetition = repetition;
    randomOffset = offset;
    startPositionTracking();
  }

  function resetSelection() {
    isResetting = true;
    selectedNumber = "0";
    selectedRepetition = "0";
    centeredIndex = SQUARES.findIndex((square) => square.number === 0);
    randomOffset = 0;
  }

  // Start the game loop
  $effect(() => {
    GameController.runGameLoop(
      updateGameState,
      updateSelection,
      resetSelection
    );

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

<div class="w-full p-4 bg-background text-text">
  <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
    <!-- Game section -->
    <div>
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
          <label for="numberInput" class="block text-sm font-medium mb-2 text-text-secondary">
            Enter a number ({SQUARES.map((s) => s.number).join(", ")}):
          </label>
          <input
            id="numberInput"
            type="number"
            value={selectedNumber}
            onchange={handleInputChange}
            class="border rounded-sm px-3 py-2 w-24 bg-background-secondary text-text border-secondary"
          />
        </div>
        <div>
          <label for="repetitionInput" class="block text-sm font-medium mb-2 text-text-secondary">
            Enter repetition (0-{REPETITIONS - 1}):
          </label>
          <input
            id="repetitionInput"
            type="number"
            value={selectedRepetition}
            onchange={handleInputChange}
            class="border rounded-sm px-3 py-2 w-24 bg-background-secondary text-text border-secondary"
          />
        </div>
      </div>

      <div class="relative w-full h-24 bg-background-secondary">
        <div
          class="absolute left-1/2 top-0 h-full w-0 border-l-2 border-center-bar -translate-x-1/2 z-10"
        ></div>

        <div class="absolute w-full h-full overflow-hidden">
          <div
            bind:this={sliderElement}
            class="absolute flex space-x-2 h-full items-center"
            style="left: 50%; transform: {transform()}; transition: transform {isResetting
              ? `${GAME_CONFIG.RESET_ANIMATION_DURATION}ms ${GAME_CONFIG.RESET_ANIMATION_CURVE}`
              : `${GAME_CONFIG.ROLL_ANIMATION_DURATION}ms ${GAME_CONFIG.ROLL_ANIMATION_CURVE}`}; will-change: transform"
            ontransitionend={handleTransitionEnd}
          >
            {#each Array(REPETITIONS) as _, repIndex}
              {#each SQUARES as square, index}
                <div
                  class="
                    shrink-0
                    flex items-center justify-center
                    w-16 h-16 rounded-lg border-2 text-lg font-medium
                    {repIndex * SQUARES.length + index === centeredIndex
                    ? 'border-primary'
                    : 'border-secondary'}
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

    <!-- Chat section -->
    <div>
      <ChatRoom />
    </div>
  </div>
</div>
