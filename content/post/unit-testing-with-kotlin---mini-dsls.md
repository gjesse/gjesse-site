+++
draft = true
date = "2017-03-20T09:51:19-05:00"
title = "Unit testing with kotlin - mini dsls"

+++

Recently I was writing a series of unit tests for a homegrown query language. The query language was designed to be end-user friendly for moderately to advanced technical users. The data being searched on is stored in ElasticSearch, and we needed to test a variety of edge cases and ensure that they were correctly translated into ES queries.

Initially I considered inspecting our internally generated ES queries, but that approach was quickly abandoned. ES queries are complex, and honestly making assertions on the structure of the query is just an implementation detail. What I really cared about was that a search for `abc || efg` matched what you would expect it to match. Fortunately ES is just in java, so we can spin up an internal cluster just for testing purposes. However that takes a little while, so internal ES instance was setup as a static `@ClassRule`, so that all test variations could take advantage of the same server.

That done, all I had to submit a plain text query string to a class that produced an elasticSearch query, then run the query and assert on the final results. However, before that I needed to create some entities in ES to search on. So my first iteration looked something like this:

```kotlin
@Test
fun `simple OR query matches 2 entities`() {

    val query = "abc || def"
    val entity1 = "abc"
    val entity2 = "def"

    dao.put(entity1)
    dao.put(entity2)

    val results = dao.query(query)

    assertThat(results)
        .containsAll(listOf(entity1, entity2))
        .hasSize(2)
    
}
```

This works pretty well.  However as soon as I have another test case, it starts breaking down. 

```kotlin
@Test
fun `asterisk matches all sources`() {

    val query = "*"
    val entity1 = "abc"
    val entity2 = "def"

    dao.put(entity1)
    dao.put(entity2)

    val results = dao.query(query)

    assertThat(results)
        .containsAll(listOf(entity1, entity2))
        .hasSize(2)
}
```

Looks good, right? well, not necessarily. It *might* appear work, but only if our `dao` code is using the entity as the primary key. It is, but we don't want to rely on that - our next test might need to put 4 different entities, or name them differently, and now we have persisted, shared state between our tests. Worse, it's not immediately obvious that this is happening, so when your tests start breaking later, it will be that much harder to track down the problem.

So we have a few possible solutions: 

  1. Precreate a shared entity set for all tests to use.
     
     This is a fairly resonable approach and will usually scale to a small number of tests. Inevitably though, you'll want to test something that's *not* in that set.. and you still have the shared state problem.
  
  1. use Junit `@Params`, and create and delete using `@Before` and `@After`.
  
     Another not-unreasonable approach, but again I find it lacks flexibility over time. As well, the input is separated from each individual test in another part of the file, so figuring out why any one test is failing requires the added context of figuring out how `@Params` are being used.
  
  1. Each test creates & deletes it's own data
  
     Avoids the problems with the other solutions, but now we have tests that look like this:

```kotlin
@Test
fun `test template`() {
    // setup
    val entity1 = "abc"
    val entity2 = "def"
    val query = "*"

    dao.put(entity1)
    dao.put(entity2)

    // act
    val results = dao.query(query)

    // assert
    assertThat(results)
        .containsAll(listOf(entity1, entity2))
        .hasSize(2)

    // cleanup
    dao.delete(entity1)
    dao.delete(entity2)
}
```