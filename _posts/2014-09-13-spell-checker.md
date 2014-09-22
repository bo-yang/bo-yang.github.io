---
layout: post
title: Spell Checker
categories: 
- Algorithm
- Python
tags:
- Algorithm
- Python
status: publish
type: post
published: true
author: Bo Yang
---

### 1. Problem

>Write a program that reads a large list of English words (e.g. from `/usr/share/dict/words` on a unix system) into memory, and then reads words from stdin, and prints either the best spelling suggestion, or "NO SUGGESTION" if no suggestion can be found. The program should print "`>`" as a prompt before reading each word, and should loop until killed.
>
>Your solution should be faster than O(n) per word checked, where n is the length of the dictionary. That is to say, you can't scan the dictionary every time you want to spellcheck a word.
>
>For example:
>
>	>sheeeeep
>	sheep
>
>	>peepple
>	people
>
>	>sheeple
>	NO SUGGESTION
> 
>The class of spelling mistakes to be corrected is as follows:
>
>Case (upper/lower) errors:
>
>	"inSIDE" => "inside"
>
>Repeated letters:
>
>	"jjoobbb" => "job"
>
>Incorrect vowels:
>
>	"weke" => "wake"
>
>In addition, any combination of the above types of error in a single word should be corrected (e.g. `"CUNsperrICY" => "conspiracy"`).
>
>If there are many possible corrections of an input word, your program can choose one in any way you like, however your results must match the examples above (e.g. `"sheeeeep"` should return `"sheep"` and not `"shap"`).
>
>Final step: Write a second program that generates words with spelling mistakes of the above form, starting with correctly spelled English words. Pipe its output into the first program and verify that there are no occurrences of "NO SUGGESTION" in the output.

### 2. Solution

All words loaded from dictionary need to be transformed into patterns (such `p**pl*` for `people`) at first, and then stored into a hash table(`dict`).Since multiple words may mapped into the same pattern, **`defaultdict`** is used to store the words with identical patterns in a list. 

Given a (mis-spelled) word, it will also be transformed into a pattern, then search the pattern in the hash table. Since mis-spelled consequtive vowels could be mapped into one star(*)(i.e. `peepple => p*pl*`, but `people => p**pl*`),this program will also try to repeat the stars so as toenlarge candidates.

Based on the patterns, the candidate words could be found in the hash table(`defaultdict`). In order to determine the closest match, [Levenshtein distance](http://en.wikipedia.org/wiki/Levenshtein_distance) is used to calculate the distance between a candidate word and the given (mis-spelled) word. 

In order to guaranttee that the suggestions are reasonable, the English word frequencies(stored in a hash table, please refer to [http://en.wiktionary.org/wiki/Wiktionary:Frequency_lists#English](http://en.wiktionary.org/wiki/Wiktionary:Frequency_lists#English)) are adopted to choose the more popular candidate by default - if there are multiple suggestions to a pattern, choose the most frequent one.

Time complexity: `O(n*m)` for loading dictionary and `O(1 + m)` for word checking, where `n` is the number of entries in the dictionary, and m is the number of characters in the string. Finding a word in hash table usually takes constant time `O(1)`, and word operations costs `O(m)`.

### 3. Code

For the python implementation of this problem, please refer to [https://github.com/bo-yang/ProgrammingChallenges/tree/master/spell_check](https://github.com/bo-yang/ProgrammingChallenges/tree/master/spell_check).
