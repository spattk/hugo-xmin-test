---
title: Java Coding Cheatsheet
date: 2024-05-23T10:18:00.000-07:00
author: Sitesh Pattanaik
---
Formula to make life easier while coding leetcode-styled problems in java.

## Convert an `Array` to a `List`
```
Map<Integer, List<Character>> numToChar = new HashMap<>();
```
#### 1. Using `List.of`
```
numToChar.put(1, List.of('a', 'b', 'c')) 
```
- creates an immutable read only array
- can't be modified
- can't be extended
- doesn't create an hidden arrays, hence efficient
- the exception to the above statement is, it create a hidden array only when the number of elements that are passed as params is greater than 10 as `static <E> List<E> of(E e1, E e2, E e3, E e4) { ... }` supports till 10 params

#### 2. Using `Arrays.asList`
```
numToChar.put(1, Arrays.asList('a', 'b', 'c'))
```
- creates an fixed sized mutable array
- can't be extended
- creates a hidden array

#### 3. Using `new ArrayList`
```
numToChar.put(1, new ArrayList<>(List.of('a', 'b', 'c'))); 
```
- creates fully mutable and dynamic list
- it creates a whole new structure and copies the data from it.
- can wrap `Arrays.asList()` as well.

## Create a Array of HashSet
#### 1. Using arrays
```
Set<String>[] array = (Set<String>[]) new Set[size];
for (int i = 0; i < size; i++) {
    array[i] = new HashSet<String>();
}
```
- this is fine but there is a need to manually cast it.

#### 2. Using List
```
List<Set<String>> listOfSets = new ArrayList<>();
for (int i = 0; i < size; i++) {
    listOfSets.add(new HashSet<>());
}
```
- much more flexible and much cleaner
