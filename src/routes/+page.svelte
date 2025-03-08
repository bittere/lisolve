<script lang="ts">
  import { Button, AlertDialog } from "bits-ui";

  let inputValue = $state("");
  let dialogOpen = $state(false);
  let logs = $state<string[]>([]);
  let closeButtonActive = $state(false);

  let keepGoing = $state(false);

  function startProcess(cookie: string) {
    let nextPuzzleId = "kg46g";
    const form = new FormData();
    form.append("win", "true");
    form.append("rated", "true");

    const int = setInterval(async () => {
      if (keepGoing) {
        const req = await fetch(
          `https://px.bittere.workers.dev/?url=${encodeURIComponent(`https://lichess.org/training/complete/mix/${nextPuzzleId}`)}&origin=${encodeURIComponent("https://lichess.org")}&cookies=${encodeURIComponent(`lila2=${inputValue}`)}`,
          {
            method: "POST",
            body: form,
          }
        );
        const body = await req.json();
        logs.push(`${body.round.ratingDiff} awarded...`);
        nextPuzzleId = body.next.puzzle.id;
      } else {
        clearInterval(int);
      }
    }, 1000);
  }

  function handleClick() {
    if (!inputValue) return alert("Please enter a valid LILA2 cookie.");

    closeButtonActive = false;
    logs = [];
    dialogOpen = true;

    keepGoing = true;
    startProcess(inputValue);
  }

  function stopProcess() {
    logs.push("Process stopped by user.");
    logs = logs;
    closeButtonActive = true;
    keepGoing = false;
  }
</script>

<section
  class="mx-auto px-6 py-12 md:py-24 text-center dark:bg-neutral-950 min-h-screen text-neutral-900 dark:text-neutral-100"
>
  <h1
    class="text-3xl sm:text-5xl md:text-6xl lg:text-7xl font-bold tracking-tighter mb-4"
  >
    Lisolve.
  </h1>
  <p class="text-neutral-600 dark:text-neutral-400 mb-8 max-w-md mx-auto">
    Lichess says, exploit our puzzles as you like. Well, here it is: the Lichess
    Puzzle Solver. Just enter your <code>LILA2</code> cookie here.
  </p>
  <div class="flex flex-col md:flex-row justify-center items-center gap-4">
    <input
      type="text"
      bind:value={inputValue}
      placeholder="Enter cookie..."
      class="px-4 py-2 rounded-md border border-neutral-300 dark:border-neutral-700 bg-white dark:bg-neutral-800 text-neutral-900 dark:text-neutral-100 focus:outline-none w-full md:max-w-sm"
    />
    <Button.Root
      onclick={handleClick}
      class="px-5 py-2 rounded-md bg-neutral-500 hover:bg-neutral-600 text-white font-medium shadow-md transition-colors duration-200 focus:outline-none w-full md:w-auto"
    >
      Let's Go
    </Button.Root>
  </div>
</section>

<AlertDialog.Root bind:open={dialogOpen}>
  <AlertDialog.Portal>
    <AlertDialog.Overlay class="fixed inset-0 bg-black/50 z-50" />
    <AlertDialog.Content
      class="fixed left-[50%] top-[50%] -translate-x-1/2 -translate-y-1/2 bg-white dark:bg-neutral-900 shadow-lg rounded-lg p-6 z-50 w-full max-w-lg"
    >
      <AlertDialog.Title
        class="text-lg font-semibold mb-2 dark:text-neutral-100"
        >Logs</AlertDialog.Title
      >
      <div
        class="font-mono rounded-md p-4 mb-4 max-h-60 overflow-y-auto bg-neutral-100 dark:bg-neutral-800 text-neutral-800 dark:text-neutral-200"
      >
        {#each logs as log}
          <p>{log}</p>
        {/each}
      </div>
      <div class="flex gap-4 justify-end">
        <AlertDialog.Cancel asChild>
          <Button.Root
            disabled={!closeButtonActive}
            class="dark:disabled:text-neutral-500 disabled:text-neutral-400 text-neutral-900 dark:text-white px-4 py-2 rounded-md disabled:cursor-not-allowed"
          >
            Close
          </Button.Root>
        </AlertDialog.Cancel>
        <Button.Root
          onclick={stopProcess}
          class="bg-red-500 text-white px-4 py-2 rounded-md"
        >
          Stop
        </Button.Root>
      </div>
    </AlertDialog.Content>
  </AlertDialog.Portal>
</AlertDialog.Root>
