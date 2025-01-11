---
layout: post
title:  "Javascript Generator Functions"
date:   2024-12-01 15:38:00 -0500
categories:
---
## Table of Contents
1. [Introduction](#introduction)
1. [The Case for Function Generators](#case-for-function-generators)
1. [Problem Walk-through](#problem-walk-through)
    - [Problem Statement](#problem-statement)
    - [Solution Approach](#solution-approach)
      - [Iteration #1](#approach-1)
      - [Iteration #3](#approach-3)

## <a name="introduction">Introduction</a>
In most programming languages, tasks that involve sequential or incremental calculation of some new value commonly use a loop written explicitly in the procedural style.

```javascript
for (let i = 0; i < 10; i++) {
    // do something 10 times
}
```
The imperative, procedural style places responsibility on the programmer to handle index 
variables, any breaking of the loop if desired, and is essentially static at run-time unless extreme care is taken to eliminate the chance of a runaway infinite loop. This is a well-documented pitfall of any general-purpose language that supports procedural constructs (C, C++, Java, amongst many others).

>The core reason why this type of looping is prone to bugs is that the author must work in different layers of abstraction in such a short span of the code that it becomes difficult to keep track of all the edge cases that may arise in the loop's construction.

An example would be if the author is writing a loop to iterate over a list of pizza orders in order to calculate how many pizzas with pepperoni were ordered in a month. The data the author is most concerned with is are pizza orders with various attributes. Each pizza order that's inspected is accessed in the array with the iterating variable.
```javascript
// pizzaOrders: Array<PizzaOrder> = [...];
for (let i = 0; i < pizzaOrders.length; i++) {
    // do something with a pizza order ...
    const ithOrderTime = pizzaOrders[i].timeStamp;
}
```
The author, in most cases, does not care about the `ith` order in the result, they just want to work with `pizzaOrders` as an abstract data structure that represents the list of orders. This is a pretty simplistic example, and most modern languages have a construct readily available to avoid the pesky iterator variable. In Javascript, for example, the `for..of` loop allows the author to abstract away the iterator.
```javascript
for (const order of pizzaOrders) {
    // do something with a pizza order ...
    const orderTime = order.timeStamp;
}
```
This approach eliminates the hazard of index variables and streamlines the code. However, not all looping problems are alike. In some algorithms, it is not necessary to loop through the entire list to achieve the goal at-hand and likely requiring the introduction of complex breaking conditions. 

Further, for algorithms that require logic based on the relative lengths of lists (i.e., "perform some operation on lists A and B until all elements are exhausted from the longer of the two"), the practice of nesting for-loops introduces additional complexity in such scenarios, as the author must carefully coordinate outer and inner-loop structures.

## <a name="case-for-function-generators">The Case for Function Generators</a>
Function generators in JavaScript offer an elegant alternative to traditional loops, particularly in cases requiring more nuanced control. Generators, introduced in ES6, allow a function to yield multiple values over time, pausing and resuming execution as needed. This capability enables more expressive and flexible iteration patterns.

Expanding on the pizza orders example, here's an example using a function generator that simplifies filtering orders:
```javascript
function* getPizzaOrdersWithPepperoni(pizzaOrders: Array<PizzaOrder>) {
    for (const order of pizzaOrders) {
        if (order.toppings.includes('pepperoni')) {
            yield order;
        }
    }
}

const pizzaOrders: Array<PizzaOrder>= [
    { toppings: ['pepperoni', 'cheese'] },
    { toppings: ['mushrooms'] },
    { toppings: ['pepperoni', 'olives'] }
];

for (const order of getPizzaOrdersWithPepperoni(pizzaOrders)) {
    // Do something with orders that had pepperoni ...
    console.log('Pepperoni pizza found:', order);
}
```
In this example, the generator function `getPizzaOrdersWithPepperoni` encapsulates the logic for filtering orders. By yielding matching orders, the generator enables lazy evaluation—processing only the necessary elements when needed; offering the following advantages while providing a clean interface to the calling code:

  1. Improved Readability: The generator’s logic is self-contained, reducing cognitive load when reading and maintaining the code.
  1. Lazy Evaluation: Generators only compute values on demand, making them memory-efficient when working with large data sets.
  1. Composition: Generators can be composed to build more complex pipelines, enabling reusable and modular code.

### <a name="Generators as Alternatives to Loops"></a>

Generators shine in scenarios where traditional loops become cumbersome or error-prone. Consider the example of merging two sorted lists into a single sorted output:

```javascript
function* mergeSortedLists(listA: any[], listB: any[]) {
    let i = 0, j = 0;
    while (i < listA.length || j < listB.length) {
        if (i >= listA.length) {
            yield listB[j++];
        } else if (j >= listB.length) {
            yield listA[i++];
        } else if (listA[i] < listB[j]) {
            yield listA[i++];
        } else {
            yield listB[j++];
        }
    }
}

// Example usage:
const listA = [1, 3, 5];
const listB = [2, 4, 6];

for (const value of mergeSortedLists(listA, listB)) {
    console.log(value);
}
```

This generator merges the two lists without requiring nested loops or complex break conditions. Each yield statement produces the next smallest value, simplifying the logic and improving maintainability.

Let's take a look at a simple problem case and iterate a solution from using imperative-style and indexed for-loops to generators.
## <a name="problem-walk-through">Problem Walk-through</a>
### <a name="problem-statement">Problem Statement</a>
>Merge Strings Alternately:
You are given two strings word1 and word2. Merge the strings by adding letters in alternating order, starting with word1. If a string is longer than the other, append the additional letters onto the end of the merged string. Return the merged string.

#### Example:
##### Input:
  - [arg1] `word1 = "dog"`
  - [arg2] `word2 = "cat"`

##### Output:
  - `"dcoagt"`

#####  Constraints:
  - `1 <= word1.length`
  - `word2.length <= 100`
  - `word1` and `word2` consist of lowercase English letters

### <a name="solution-approach">Solution Approach</a>

### <a name="approach1">Approach #1</a>
```javascript
function mergeStrings(word1, word2) {
    let merged = '';
    let i = 0, j = 0;

    // Loop through both strings until one is exhausted
    while (i < word1.length && j < word2.length) {
        merged += word1[i++];
        merged += word2[j++];
    }

    // Append the remaining characters from the longer string
    while (i < word1.length) {
        merged += word1[i++];
    }
    while (j < word2.length) {
        merged += word2[j++];
    }

    return merged;
}

// Example usage:
const word1 = "abc";
const word2 = "pqrstu";
console.log(mergeStrings(word1, word2)); // Output: "apbqcrstu"
```
A few things to note here in this naive approach. First, the author is tracking two separate indices (i and j), increasing the risk of off-by-one errors.
Additionally, more loops are required to handle the leftover characters from the longer string. 
And most importantly, as noted earlier - intermixing layers of abstraction: the merging logic is intermingled with index management, reducing code clarity and bound to be a source of bugs.
#### <a name="approach2">Approach #2</a>
```javascript
function mergeAlternately(word1: string, word2: string): string {
    // Generator function that yields each character
    // of the supplied string.
    function* generateLettersFromWord(word: string) {
        for(letter of word) {
            yield letter;
        }
    }

    // Generator function objects for each word
    const gen1 = generateLettersFromWord(word1);
    const gen2 = generateLettersFromWord(word2);

    // Base case, empty string
    let mergedWord = "";
    let done = false;

    // Calling next() on each generator object and 
    // using reassignment and default values
    while (!done){
        const { done: word1Done = true, value: word1Value = "" } = gen1.next();
        const { done: word2Done = true, value: word2Value = "" } = gen2.next();
        mergedWord += `${word1Value}${word2Value}`;
        done = word1Done && word2Done;
    }
    return mergedWord;
};
```
#### <a name="approach3">Approach #3</a>
```javascript
/**
 * Abstracting away the need for done/value from the generator.
 * Additionally, removes away the limitation on the number of arguments 
 * possible allowing for up to N words to be merged alternately.
 */
function mergeAlternately(word1: string, word2: string): string {
    function* generateLetters(...words: string[]) {
        for (let i = 0; i < Math.max(...words.map(({ length }) => length)); i++) {
            for (const word of words) {
                if (i < word.length) {
                    yield word[i];
                }
            }
        }
    }
    return [...generateLetters(word1, word2)].join("");
}
```
