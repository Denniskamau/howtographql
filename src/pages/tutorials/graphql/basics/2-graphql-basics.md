---
title: GraphQL Basics
question: "What are GraphQL subscription used for?"
answers: ["Event-based realtime functionality", "Schema-based realtime functionality", "You use them to subscribe to the GraphQL Weekly newsletter", "They combine Queries and Mutations and allows to read and write data"]
correctAnswer: 0
description: "In this chapter, you learn about the basic GraphQL concepts, such as Queries, Mutations, Subscriptions and the GraphQL Schema"
---

In this chapter, you'll learn the basic concepts of GraphQL. That includes a first glimpse at the syntax for defining _types_ as well as sending _queries_ and _mutations_. We also prepared your own sandbox environment, based on [graphql-up](https://github.com/graphcool/graphql-up), that you can use directly on the website to try out what you learn.  

### The Schema Definition Language (SDL)

GraphQL has its own type system that’s used to define the _schema_ of an API. The syntax for writing schemas is called [Schema Definition Language](https://www.graph.cool/docs/faq/graphql-sdl-schema-definition-language-kr84dktnp0/) (SDL).

Here is an example how we can use the SDL to define a simple type called `User`:

```graphql(nocopy)
type Person {
  name: String!
  age: Int!
}
```

This type has two *fields*, they’re called `name` and `age` and are both of type `String`. The `!` following the type means that this field is *required*.

It’s also possible to express relationships between types. In the example of a *blogging* application, a `Person` could be associated with a `Post`:

```graphql(nocopy)
type Post {
  title: String!
  author: Person!
}
```

Conversely, the other end of the relationship needs to be placed on the `Person` type:

```graphql(nocopy)
type Person {
  name: String!
  age: Int!
  posts: [Post!]!
}
```

Note that we just created a *one-to-many*-relationship between `Person` and `Post` since the `posts` field on `Person` is actually an *array* of posts.

### Fetching Data with Queries

When working with REST APIs, data is loaded from specific endpoints. Each endpoint has a clearly defined structure of the information that it returns.

The approach that’s taken in GraphQL is radically different. Instead of having multiple endpoints that return fix data structures, GraphQL APIs typically only expose *a single endpoint*. This works because the structure of the data that’s returned is not fixed. Instead, it’s completely flexible and let’s the client decide what data is actually needed. 

That means that the client needs to send more *information* to the server to express its data needs - this information is called a *query*.

> If you want to try out the following queries and mutations, click one of the **Get your GraphQL Endpoint**-buttons and the server will be generated for you. Its functionality is exposed through a [GraphQL Playground](https://www.graph.cool/docs/faq/tips-and-tricks-graphql-playground-ook6luephu/) that allows you to explore the API in an interactive manner.

#### Basic Queries

Let’s take a look at an example query that a client could send to a server:

<Playground>

```graphql(nocopy)
{
  allPersons {
    name
  }
}
```

</Playground>

This query would return a list of all users currently stored in the database. Here’s an example response:

```js(nocopy)
{
  "allPersons": [
    { "name": "Johnny" },
    { "name": "Sarah" },
    { "name": "Alice" }
  ]
}
```

Notice that each user only has the `name` in the response, but the `age` is not returned by the server. That’s because the `name` was the only field that was specified in the query.

If the client also needed the users' `age`, all it has to do is to slightly adjust the query and include the new field in the query’s payload:

<Playground>

```graphql
{
  allPersons {
    name
    age
  }
}
```

</Playground>

One of the major advantages of GraphQL is that it allows for naturally querying *nested* information. For example, if you wanted to load all the `posts` that a `Person` has written, you could simply follow the structure of your types to request this information:

<Playground>

```graphql
{
  allPersons {
    name
    age
    posts {
      title
    }
  }
}
```

</Playground>


#### Queries with Arguments

In GraphQL, each *field* can have zero or more arguments if that's specified in the *schema*. For example, the `allPersons` could have a `last` parameter to only return up to a specific number of users. Here's what a corresponding query would look like:

<Playground>

```graphql(nocopy)
{
  allPersons(last: 2) {
    name
  }
}
```

</Playground>

### Writing Data with Mutations

Next to requesting information from a server, the majority of applications also need some way of making changes to the data that’s currently stored in the backend. With GraphQL, these changes are made using so-called *mutations*. There generally are three kinds of mutations:

* creating new data
* updating existing data
* deleting existing data

Mutations follow the same syntactical structure as queries, but they always need to start with the `mutation` keyword. Here’s an example for how we might create a new `Person`:


<Playground>

```graphql(nocopy)
mutation {
  createPerson(name: "Bob", age: 36) {
    id
  }
}
```

</Playground>

Notice that similar to the query we wrote before, we’re able to specify a payload where we can ask for different properties of the new `Person`. In our case, we’re asking for the `name` and the `age` - though admittedly that’s not super helpful in our example since we obviously already know them. However, being able to also query information when sending mutations can be a very powerful tool that allows to retrieve new information in a single roundtrip! 

The server response for the above mutation would look as follows:

```js(nocopy)
{
  "createPerson": {
    "name": "Alice",
    "age": 36
  }
}
```

One pattern you’ll often find is that GraphQL types have unique *IDs* that are generated by the server. Extending our `Person` type from the before, we could add an `id` as follows:

```graphql(nocopy)
type Person {
  id: ID!
  name: String!
  age: Int!
}
```

Now, when a new `Person` is created, you could directly ask for the `id` since that is information that wasn’t available on the client beforehand:

```graphql(nocopy)
mutation {
  createPerson(name: "Alice", age: 36) {
    id
  }
}
```

### Realtime Updates with Subscriptions

Another important requirement for many applications today is to have a *realtime* connection to the server in order to get immediately informed about important events. For this use case, GraphQL offers the concept of *subscriptions*. 

When a client *subscribes* to an event, it will hold a steady connection to the server. Whenever that particular event then actually happens, the server pushes the corresponding data to the client. Unlike queries and mutations that follow a typical “*request-response*-cycle”, subscriptions represent a *stream* of data sent over to the client.

Subscriptions are written using the same syntax as queries and mutations. Here’s an example where we subscribe on events happening on the `Person` type:

```graphql(nocopy)
subscription {
  newPerson {
    name
    age
  }
}
```

After a client sent this subscription to a server, a connection is opened between them. Then, whenever a new mutation is performed that creates a new `Person`, the server sends the information about this user over to the client:

```graphql(nocopy)
{
  "newPerson": {
    "name": "Jane",
    "age": 23
  }
}
```

### Defining a Schema

Now that you have a rough idea of what queries, mutations and subscriptions look like, let’s put it all together with how you can write a schema that would allow to execute the examples you’ve seen so far.

The *schema* is one of the most important concepts when working with a GraphQL API. It specifies the capabilities of the API and defines how clients can request the data. It is often seen as a *contract* between the server and client.

Generally, a schema is simply a collection of GraphQL types. However, when writing the schema for an API, there are some special *root* types:

```graphql(nocopy)
type Query { ... }
type Mutation { ... }
type Subscription { ... }
```

The `Query`, `Mutation` and `Subscription` types are the *entry points* for the requests sent by the client. To enable the `allPersons`-query that we save before, the `Query` type would have to be written as follows:

```graphql(nocopy)
type Query {
  allPersons: [Person!]!
}
```

`allPersons` is called a *root field* of the API. Considering again the example where we added the `limit` argument to the `allPersons` field, we'd have to write the `Query` as follows:

```graphql(nocopy)
type Query {
  allPersons(limit: Int): [Person!]!
}
```

Similarly, for the `createPerson`-mutation, we’ll have to add a root field to the `Mutation` type:

```graphql(nocopy)
type Mutation {
  createPerson(name: String!, age: String!): Person!
}
```

Notice that this root field takes two arguments as well, the `name` and the `age` of the new `Person`.

Finally, for the subscriptions, we’d have to add the `newPerson` root field:

```graphql(nocopy)
type Subscription {
  newPerson: Person!
}
```