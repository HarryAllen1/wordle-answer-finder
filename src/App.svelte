<script lang="ts">
  import { solutionList } from './lib/word-list';

  const wordleEpoch = new Date(2021, 5, 19, 0, 0, 0, 0);

  const Ba = (e: Date, a: Date) => {
    const s = new Date(e),
      t = new Date(a).setHours(0, 0, 0, 0) - s.setHours(0, 0, 0, 0);
    return Math.round(t / 864e5);
  };
  const Fa = (e: Date) => {
    return Ba(wordleEpoch, e);
  };
  const getSolution = (e: Date) => {
    let a: number,
      s = Fa(e);
    return (a = s % solutionList.length), solutionList[a];
  };
  let solution = '';
  $: selectedDate = '';
</script>

<div id="main" class="flex content-center justify-center flex-col items-center">
  <div
    class="max-w-lg grid content-center mt-6 rounded-lg border-slate-300 border-[1px] p-6 gap-4"
  >
    <h1
      class="text-center text-gray-900 text-3xl font-extrabold tracking-tight "
    >
      Wordle Solution Finder
    </h1>
    <p class="text-center">
      A program that finds the Wordle solution for any day, even in the future.
    </p>
    <div class="w-full flex justify-center ">
      <input
        type="date"
        bind:value={selectedDate}
        class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50"
      />
    </div>

    <button
      on:click={() => {
        const date = new Date(selectedDate);
        date.setDate(date.getDate() + 1);
        if (date.getTime() < wordleEpoch.getTime()) {
          return (solution = 'The date must be after June 19th, 2021');
        }
        solution = getSolution(date);
      }}
      type="button"
      class="inline-flex justify-center px-4 py-2 text-sm font-medium text-blue-900 bg-blue-100 border border-transparent rounded-md hover:bg-blue-200 focus:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 focus-visible:ring-blue-500"
    >
      View Solution
    </button>

    <p class="text-center">
      {solution == undefined ? 'You must choose a date' : solution}
    </p>
  </div>
  <a
    class="text-center underline text-slate-400 hover:text-slate-500"
    target="_blank"
    rel="noopener"
    href="https://github.com/MajesticString/wordle-answer-finder"
  >
    Source Code
  </a>
</div>
