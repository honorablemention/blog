---
layout: post
title:  "Javascript Generators"
date:   2024-12-01 15:38:00 -0500
categories:
---

In most programming languages, tasks that involve sequential or incremental calculation of some new value commonly use a loop written explicitly in the procedural style.

```javascript
for (let i = 0; i < 10; i++) {
    // do something 10 times
}
```

The imperative, procedural style places responsibility on the programmer to handle index 
variables, any breaking of the loop if desired, and is essentially static at run-time unless extreme care is taken to eliminate the chance of a runaway infinite loop. This is a well-documented pitfall of any general-purpose language that supports procedural constructs (C, C++, Java, amongst many others). The core reason why this type of looping is prone to bugs is that the author must work in different layers of abstraction in such a short span of the code that it becomes difficult to keep track of all the edge cases that may arise in the loop's construction.

but it has a verbosity to it that makes it cognitively heavier to digest by readers 


```javascript
function mergeAlternately(word1: string, word2: string): string {

    function* generateLettersFromWord(word: string) {
        for(let i= 0; i < word.length; i++) {
            yield word[i];
        }
    }

    const gen1 = generateLettersFromWord(word1);
    const gen2 = generateLettersFromWord(word2);

    let a = "";
    let done = true;
    while (done){
        const { done: word1Done = true, value: word1Value = "" } = gen1.next();
        const { done: word2Done = true, value: word2Value = "" } = gen2.next();
        a += `${word1Value}${word2Value}`;
        done = !(word1Done && word2Done);
    }
    return a;
};
```
