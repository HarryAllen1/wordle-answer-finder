<script lang="ts">
  import { solutionList } from './lib/word-list';
  import { initializeApp } from 'firebase/app';
  import {
    getFirestore,
    doc,
    setDoc,
    getDoc,
    Firestore,
    Timestamp,
  } from 'firebase/firestore';
  import {
    Listbox,
    ListboxButton,
    ListboxOptions,
    ListboxOption,
  } from '@rgossiaux/svelte-headlessui';
  import { getCountry } from './lib/worldle';
  import { DateTime } from 'luxon';
  import { onMount } from 'svelte';
  import { getBrdl } from './lib/brdl';

  const supportedWordles = [
    {
      id: 1,
      name: 'Wordle',
    },
    {
      id: 2,
      name: 'Worldle',
    },
    {
      id: 3,
      name: 'Brdl',
    },
  ];

  let defaultWordle = supportedWordles[0];
  onMount(() => {
    defaultWordle =
      supportedWordles[
        localStorage.getItem('selectedWordle')
          ? parseInt(localStorage.getItem('selectedWordle'))
          : 0
      ];
  });

  const app = initializeApp({
    apiKey: 'AIzaSyDrtSNOgy7GO3O2T_wJlVfR5Xi4uPFQOec',
    authDomain: 'wordle-solution-finder.firebaseapp.com',
    projectId: 'wordle-solution-finder',
    storageBucket: 'wordle-solution-finder.appspot.com',
    messagingSenderId: '360937075654',
    appId: '1:360937075654:web:7f5e73e59ccaaf37c678c7',
  });
  const db = getFirestore(app);

  const logEvent = async (db: Firestore, event: string) => {
    const currentCount = await getDoc(doc(db, 'events', event));
    await setDoc(doc(db, 'events', event), {
      count: (currentCount.data()?.count ?? 0) + 1,
      lastTime: new Timestamp(Date.now() / 1000, 0),
    });
  };

  logEvent(db, 'page_visit');

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
  const currentDate = new Date();
  $: selectedDate = DateTime.now().minus({ days: 1 }).toJSDate();
  $: selectedDate, logEvent(db, 'date_selected');
</script>

<div
  id="main"
  class="flex content-center justify-center flex-col items-center "
>
  <div class="w-72 mt-5">
    <div class="relative">
      Wordle Variant:
      <Listbox
        value={defaultWordle}
        on:change={(e) => {
          defaultWordle = e.detail;
          localStorage.setItem('selectedWordle', (e.detail.id - 1).toString());
        }}
      >
        <ListboxButton
          class="relative w-full py-2 pl-3 pr-10 text-left bg-white rounded-lg shadow-lg cursor-default border-slate-300 border-[1px] focus:outline-none focus-visible:ring-2 focus-visible:ring-opacity-75 focus-visible:ring-white focus-visible:ring-offset-blue-300 focus-visible:ring-offset-2 focus-visible:border-indigo-500 sm:text-sm"
          >{defaultWordle.name}</ListboxButton
        >
        <ListboxOptions
          class="absolute w-full py-1 mt-1 overflow-auto text-base bg-white rounded-md shadow-lg max-h-60 ring-1 ring-black ring-opacity-5 focus:outline-none sm:text-sm"
        >
          {#each supportedWordles as wordle (wordle.id)}
            <ListboxOption
              class={({ active }) =>
                `cursor-default select-none flex flex-row relative py-2 pl-10 pr-4 ${
                  active ? 'text-blue-900 bg-blue-100' : 'text-gray-900'
                }`}
              value={wordle}
              let:active
              let:selected
            >
              <span class={`block truncate ${'font-normal'}`}>
                {wordle.name}
              </span>
              {#if selected}
                <div class="flex-grow" />
                <span
                  class={'absolute inset-y-0 right-0 flex items-center pr-4' +
                  active
                    ? 'text-opacity-0'
                    : 'text-indigo-600'}
                >
                  <svg class="w-5 h-5" viewBox="0 0 20 20" fill="currentColor">
                    <path
                      fill-rule="evenodd"
                      d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z"
                      clip-rule="evenodd"
                    />
                  </svg>
                </span>
              {/if}
            </ListboxOption>
          {/each}
        </ListboxOptions>
      </Listbox>
    </div>
  </div>

  <div
    class="max-w-[80%] grid content-center mt-6 rounded-lg border-slate-300 border-[1px] p-6 gap-4"
  >
    <h1
      class="text-center text-gray-900 text-3xl font-extrabold tracking-tight "
    >
      {defaultWordle.name} Solution Finder
    </h1>
    <p class="text-center">
      A program that finds the {defaultWordle.name} solution for any day, even in
      the future.
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
        if (
          defaultWordle.name !== 'Brdl' &&
          date.getTime() < wordleEpoch.getTime()
        ) {
          logEvent(db, 'someone_is_an_idiot');
          return (solution = 'The date must be after June 19th, 2021');
        } else if (
          defaultWordle.name === 'Brdl' &&
          date.getTime() < new Date('1/23/2022').getTime()
        ) {
          logEvent(db, 'someone_is_an_idiot');
          return (solution = 'The date must be after 1/23/2022');
        }
        logEvent(db, 'solution_found');
        solution =
          defaultWordle.name === 'Wordle'
            ? getSolution(date)
            : defaultWordle.name === 'Worldle'
            ? `${
                getCountry(DateTime.fromJSDate(date).toFormat('yyyy-MM-dd'))
                  .name
              } (lat: ${
                getCountry(DateTime.fromJSDate(date).toFormat('yyyy-MM-dd'))
                  .latitude
              }, long: ${
                getCountry(DateTime.fromJSDate(date).toFormat('yyyy-MM-dd'))
                  .longitude
              })`
            : getBrdl(new Date(selectedDate));
      }}
      type="button"
      class="inline-flex justify-center px-4 py-2 text-sm font-medium text-blue-900 bg-blue-100 border border-transparent rounded-md hover:bg-blue-200 focus:outline-none focus-visible:ring-2 focus-visible:ring-offset-2 focus-visible:ring-blue-500"
    >
      View{defaultWordle.name === 'Worldle' ? ' Worldle' : ''} Solution
    </button>

    <p class="text-center">
      {solution ?? 'You must choose a date'}
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
