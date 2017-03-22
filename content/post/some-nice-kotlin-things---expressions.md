+++
draft = false
date = "2017-03-22T17:19:53-05:00"
title = "Some nice Kotlin things - expressions"
categories = ["kotlin", "expressions"]
author = "Jesse Hodges"
+++

Kotlin has a lot of great features. As someone coming to kotlin from a java background, I want to highlight some of the little things that make it great. Today, it's expression assignments.

<!--more-->
## Assignment from an if expression

### java
```java
@Test
public void assignmentAsResultOfIf(){
    boolean newSearch = false;
    String searchQuery = "leonardo && raphael";

    final SearchResult res;
    if (newSearch) {
        res = searchNew(searchQuery);
    } else {
        res = searchOld(searchQuery);
    }
    assertThat(res.getType()).isEqualTo("old");
}
```
Notice how we have to declare the `SearchResult` variable outside of the if block scope. Fortunately we can still make it `final`. 

### kotlin

```kotlin
@Test
fun assignmentAsResultOfIf() {
    val newSearch = false
    val searchQuery = "leonardo && raphael"
    val res = if (newSearch) {
        searchNew(searchQuery)
    } else {
        searchOld(searchQuery)
    }
    assertThat(res.type).isEqualTo("old")
}
```
In kotlin, we can make this more succinct by assigning the result of the `if` expression to our result.

## Assignment as a result of a try expression

### java
```java
@Test
public void assignmentAsResultOfTry(){
    boolean doThrow = false;
    String searchQuery = "leonardo && raphael";

    SearchResult res;
    try {
        res = searchNewExceptional(searchQuery, doThrow);
    } catch (Exception e) {
        res = searchOld(searchQuery);
    }
    assertThat(res.getType()).isEqualTo("new");
}
```
Again, we have to declare the result outside of the scope of the `try` block, only now it can't be `final` anymore, becuase the catch block cannot actually know if the variable has already been initialized.  You might put the try block into it's own method to resolve this. 

### kotlin
```kotlin
@Test
fun assignmentAsResultOfTry() {
    val doThrow = false
    val searchQuery = "leonardo && raphael"
    val res = try {
        searchNewExceptional(searchQuery, doThrow)
    } catch (e: Exception) {
        searchOld(searchQuery)
    }
    assertThat(res.type).isEqualTo("new")
}
```
The kotlin version stays succinct, and we can maintain our final-ness with no extra work.

## Assignment as a result of a when/switch expression

### java
```java
@Test
public void assignmentAsResultOfSwitch(){
    int type = 2;
    String searchQuery = "leonardo && raphael";
    final SearchResult res;
    switch (type) {
        case 0:
            res = searchOld(searchQuery);
            break;
        case 1:
            res = searchNew(searchQuery);
            break;
        default:
            res = searchNew(searchQuery);
    }
    assertThat(res.getType()).isEqualTo("new");
}
```
At least we get `final` back here. 

### kotlin
```kotlin
@Test
fun assignmentAsResultOfWhen() {
    val type = 2
    val searchQuery = "leonardo && raphael"
    val res = when (type) {
        0 -> searchOld(searchQuery)
        1 -> searchNew(searchQuery)
        else -> searchNew(searchQuery)
    }
    assertThat(res.type).isEqualTo("new")
}
```

Similar pattern to other examples, although `when` in kotlin is much more powerful than java's `switch`, but that's another topic.

## Conclusion

These are some quickly contrived examples to show the basic pattern, but it's one of the many little things that make kotlin fun to work with. One might reasonably look at this and wonder what the big deal is. After all, the basic workings of the kotlin examples all can be accomplished by adding another java method, but in the end your code readability suffers a bit in java. And in the end, code readability leads to better code that's easier to maintain, so any improvements here should not be taken lightly, however small they may seem.

Source code for examples are [here](https://github.com/gjesse/some-nice-kotlin-things)