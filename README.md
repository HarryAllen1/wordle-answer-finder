# Wordle Solver

This program gets the Wordle solution for any word, even in the future.

## How?

After digging through some of the source (in the inspect element panel),
I discovered that there is an algorithm for picking the word for the day.
This just uses the algorithm in a more user friendly manner.

```ts
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
```
