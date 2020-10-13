---
title: "Schema & Types"
metaTitle: "Hypi tutorial introducing GraphQL."
metaDescription: "Introduction to GraphQL Schemas and Types."
---
## Overview
On this page, you'll learn all you need to know about the GraphQL type system and how it describes what data can be queried. Since GraphQL can be used with any backend framework or programming language, we'll stay away from implementation-specific details and talk only about the concepts.

### Type system

If you've seen a GraphQL query before, you know that the GraphQL query language is basically about selecting fields on objects. So, for example, in the following query:



<div className={"code-container"}>

<div className={"code-column"}>

       query getMyTodos($arcql:String) {
           todos(arcql:$arcql) {
             hypi {
                id
                created
              }
             description
             completed
           }
         }

         # Variables
         {
            "arcql": "* SORT hypi.updated DESC"
         }

</div>
<div className={"code-column"}>

    {
           "data": {
             "todos": [
               {
                  "hypi": {
                   "id": "c94144ec-52aa-4337-9d8b-15fa8ec5c979"
                   "created": "2019-11-02T15:45:16Z"
                   }
                  "description": "Buy milk for the office",
                  "completed":"false",
               },

               ]
             }

     }

</div>
</div>

<ul>
<li>We start with a special "root" object</li>
<li>We select the hero field on that</li>
<li>For the object returned by hero, we select the name and appearsIn fields</li>
</ul>

Because the shape of a GraphQL query closely matches the result, you can predict what the query will return without knowing that much about the server. But it's useful to have an exact description of the data we can ask for - what fields can we select? What kinds of objects might they return? What fields are available on those sub-objects? That's where the schema comes in.

Every GraphQL service defines a set of types which completely describe the set of possible data you can query on that service. Then, when queries come in, they are validated and executed against that schema.

### Type language

GraphQL services can be written in any language. Since we can't rely on a specific programming language syntax, like JavaScript, to talk about GraphQL schemas, we'll define our own simple language. We'll use the "GraphQL schema language" - it's similar to the query language, and allows us to talk about GraphQL schemas in a language-agnostic way.

### Object types and fields

The most basic components of a GraphQL schema are object types, which just represent a kind of object you can fetch from your service, and what fields it has. In the GraphQL schema language, we might represent it like this:

    type Character {
      name: String!
      appearsIn: [Episode!]!
    }


The language is pretty readable, but let's go over it so that we can have a shared vocabulary:

<ul>
<li>
`Character` is a GraphQL Object Type, meaning it's a type with some fields. Most of the types in your schema will be object types
</li>
<li>
`name` and `appearsIn` are fields on the `Character` type. That means that `name` and `appearsIn` are the only fields that can appear in any part of a GraphQL query that operates on the `Character` type.
</li>
<li>
`String` is one of the built-in scalar types - these are types that resolve to a single scalar object, and can't have sub-selections in the query. We'll go over scalar types more later.
</li>
<li>
`String!` means that the field is non-nullable, meaning that the GraphQL service promises to always give you a value when you query this field. In the type language, we'll represent those with an exclamation mark.
</li>
<li>
`[Episode!]!` represents an array of Episode objects. Since it is also non-nullable, you can always expect an array (with zero or more items) when you query the appearsIn field. And since Episode! is also non-nullable, you can always expect every item of the array to be an Episode object.
</li>
</ul>

Now you know what a GraphQL object type looks like, and how to read the basics of the GraphQL type language.

### Arguments
Every field on a GraphQL object type can have zero or more arguments, for example the length field below:

    type Starship {
      id: ID!
      name: String!
      length(unit: LengthUnit = METER): Float
    }

All arguments are named. Unlike languages like JavaScript and Python where functions take a list of ordered arguments, all arguments in GraphQL are passed by name specifically. In this case, the length field has one defined argument, unit.

Arguments can be either required or optional. When an argument is optional, we can define a default value - if the unit argument is not passed, it will be set to METER by default.

### The Query and Mutation types

Most types in your schema will just be normal object types, but there are two types that are special within a schema:

    schema {
      query: Query
      mutation: Mutation
    }

Every GraphQL service has a query type and may or may not have a mutation type. These types are the same as a regular object type, but they are special because they define the entry point of every GraphQL query. So if you see a query that looks like:


<div className={"code-container"}>

<div className={"code-column"}>

      query {
        hero {
          name
        }
        droid(id: "2000") {
          name
        }
      }

</div>
<div className={"code-column"}>

    {
      "data": {
        "hero": {
          "name": "R2-D2"
        },
        "droid": {
          "name": "C-3PO"
        }
      }
    }

</div>
</div>

That means that the GraphQL service needs to have a Query type with hero and droid fields

    type Query {
      hero(episode: Episode): Character
      droid(id: ID!): Droid
    }

Mutations work in a similar way - you define fields on the Mutation type, and those are available as the root mutation fields you can call in your query.

It's important to remember that other than the special status of being the "entry point" into the schema, the `Query` and `Mutation` types are the same as any other GraphQL object type, and their fields work exactly the same way.


### Scalar types

A GraphQL object type has a name and fields, but at some point those fields have to resolve to some concrete data. That's where the scalar types come in: they represent the leaves of the query.

In the following query, the name and appearsIn fields will resolve to scalar types:



<div className={"code-container"}>

<div className={"code-column"}>

      {
        hero {
          name
          appearsIn
        }
      }

</div>
<div className={"code-column"}>

    {{
       "data": {
         "hero": {
           "name": "R2-D2",
           "appearsIn": [
             "NEWHOPE",
             "EMPIRE",
             "JEDI"
           ]
         }
       }
     }

</div>
</div>


We know this because those fields don't have any sub-fields - they are the leaves of the query.

GraphQL comes with a set of default scalar types out of the box:



<ul>
  <li>`Int`: A signed 32‐bit integer.</li>
  <li>`Float`: A signed double-precision floating-point value.</li>
  <li>`String`: A UTF‐8 character sequence.</li>
  <li>`Boolean`: `true` or `false`</li>
  <li>`ID`: The ID scalar type represents a unique identifier, often used to refetch an object or as the key
    for a cache. The ID type is serialized in the same way as a String; however, defining it as an `ID`
    signifies that it is not intended to be human‐readable.
  </li>
</ul>


In most GraphQL service implementations, there is also a way to specify custom scalar types. For example, we could define a Date type:

    scalar Date

Then it's up to our implementation to define how that type should be serialized, deserialized, and validated. For example, you could specify that the Date type should always be serialized into an integer timestamp, and your client should know to expect that format for any date fields.

### Enumeration types
Also called Enums, enumeration types are a special kind of scalar that is restricted to a particular set of allowed values. This allows you to:

Validate that any arguments of this type are one of the allowed values
Communicate through the type system that a field will always be one of a finite set of values
Here's what an enum definition might look like in the GraphQL schema language:

    enum Episode {
      NEWHOPE
      EMPIRE
      JEDI
    }

This means that wherever we use the type `Episode` in our schema, we expect it to be exactly one of `NEWHOPE`, `EMPIRE`, or `JEDI`.

Note that GraphQL service implementations in various languages will have their own language-specific way to deal with enums. In languages that support enums as a first-class citizen, the implementation might take advantage of that; in a language like JavaScript with no enum support, these values might be internally mapped to a set of integers. However, these details don't leak out to the client, which can operate entirely in terms of the string names of the enum values.

### Lists and Non-Null
Object types, scalars, and enums are the only kinds of types you can define in GraphQL. But when you use the types in other parts of the schema, or in your query variable declarations, you can apply additional type modifiers that affect validation of those values. Let's look at an example:

    type Character {
      name: String!
      appearsIn: [Episode]!
    }

Here, we're using a String type and marking it as Non-Null by adding an exclamation mark, ! after the type name. This means that our server always expects to return a non-null value for this field, and if it ends up getting a null value that will actually trigger a GraphQL execution error, letting the client know that something has gone wrong.

The Non-Null type modifier can also be used when defining arguments for a field, which will cause the GraphQL server to return a validation error if a null value is passed as that argument, whether in the GraphQL string or in the variables.

