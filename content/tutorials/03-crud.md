# Hypi GraphQL CRUD Tutorial

In this tutorial we’re going to show you how to use our GraphQL API to perform CRUD operations on your app.

For the purpose of this tutorial, we’ve created an app with the name hypi-tutorials, and in the **Editor** section we’ve created a simple **schema** for our application.
```graphql
interface Character {
	id: ID!
	name: String!
	friends: [Character!]
}

type Actor implements Character {
	id: ID!
	name: String!
	friends: [Character!]
	starsIn: [Episode!]
}

type Director implements Character {
	id: ID!
	name: String!
	friends: [Character!]
	hasDirected: [Episode!]
}

enum Episode {
	S01E01
	S01E02
	S01E03
	S01E04
	S01E05
	S01E06
}

```

Now, with a created instance of the app, navigate to the Api Editor of the instance, or do it programmatically with a http request to:

[https://api.hypi.app/graphql](https://api.hypi.app/graphql)

with the http headers

    "Authorization": "", - your authorization token
    "hypi-domain": "" –the domain of your Hypi app

Let’s start performing some operations on your Hypi app.
For the purpose of this tutorial, we will use the **Api Editor.**

Documentation for the queries and mutations mentioned in this tutorial can be found on the right side of the Api Editor under the “DOCS” tab.

## Create

For this tutorial we would like to create an `Actor` object with the `name` “Will Smith”, `age` 17, and he stars in episodes 1, 2 and 3. 
The mutation will look like this.

```graphql
mutation {
	upsert(values: {
			Actor: { 
			name: "Will Smith", 
			age: 17, 
			starsIn: [S01E01, S01E02, S01E03] 
			}
		}
	) {
		id
	}
}
```
As you can see, if you are familiar with **GraphQL**, the call is pretty simple, you pass in the object that you want to create as a `values` parameter to the `upsert` mutation, and specify what fields of the **Magic Hypi object** you want in the response.

You can look into all of the fields that the **Magic Hypi object** contains under its section in the Hypi GraphQL API documentation, here we just return the `id`.

As expected, the response looks like this
```json
{
  "data": {
    "upsert": [
      {
        "id": "01ED4X75AG1G96PDC8A231KTC8"
      }
    ]
  }
}
```
So, we have successfully used the upsert method to create an `Actor` in our database.

Next, let’s see how we can update that actor.
## Update

As the “DOCS” say, to update an existing value, we should provide the **`hypi.id`** for the object along with the data that we want to save for that object.

Let’s say that we want to update the `age` of the actor we just created.

The `mutation` will look like this.

```graphql
mutation {
	upsert(values: { 
			Actor: { 
				age: 18, 
				hypi: { 
					id: "01ED4X75AG1G96PDC8A231KTC8" 
				} 
			}
		}
	) {
		id
	}
}
```
As you can see, the difference between the create and the update is that the update contains

```json
hypi: {
	id: "01ED4X75AG1G96PDC8A231KTC8"
}
```

Which is the `id` that we got back in the create response. As expected, the response remains the same as create.

## Read

For reading the data, we will show you two queries that Hypi provides out of the box for any object

**1.  get()**

The **get** `query` provides a way to get a single object by its `hypi.id`. 
The query will like this.

```graphql
query {
	get(type: Actor, id: "01ED4X75AG1G96PDC8A231KTC8") {
		... on Actor {
			name
			age
		}
	}
}
```
As you can see the get query accepts `type` and `id` as parameters and it returns whatever fields you specify for the type that you are using it for.

The response from this query will be

```json
{
  "data": {
    "get": {
      "name": "Will Smith",
      "age": 18
    }
  }
}
```
As we expected, we got the `name` and the `age` of the `Actor` with `id` “01ED4X75AG1G96PDC8A231KTC8”.

**2.  find()**

The **find** `query` is used to search through your data, for the purpose of this tutorial we’ve created a few more actors in the database.

The find query accepts two mandatory parameters `Type` and `arcql`, the `Type` is the type of object you want to query, and the `arcql` parameter is the query that you want to use for searching though the data.

To learn more about **Arcql**, please refer to the **Developer Hub** under the section **“Arc Query Language”**.

In this tutorial we will cover listing of all of the actors and searching them by `name`.

For getting all of the actors we will send the **wildcard** “*” as the `arcql` parameter, indicating that I want all of the actors. In the response, we want the `name` of the actor along with his `hypi.id`. 
The query will look like this.

```graphql
query {
	find(type: Actor, arcql: "*") {
		... on HypiFilterConnection {
			edges {
				node {
					... on Actor {
						name
						hypi {
							id
						}
					}
				}
			}
		}
	}
}
```
As expected, the response is a list of all of the edges of type `Actor`

```json
{
  "data": {
    "find": {
      "edges": [
        {
          "node": {
            "name": "Will Smith",
            "hypi": {
              "id": "01ED4X75AG1G96PDC8A231KTC8"
            }
          }
        },
        {
          "node": {
            "name": "Alfonso Ribeiro",
            "hypi": {
              "id": "01ED4ZCASYQ57YK2VV3WCM3D0E"
            }
          }
        },
        {
          "node": {
            "name": "Will Werrell",
            "hypi": {
              "id": "01ED506E9GQ3Z75RHE0609RNYN"
            }
          }
        }
      ]
    }
  }
}
```

Next, we will use **Arcql** to create a typical auto-complete query for the `name` field. 
The arcql will look like this.

    “name ^ ‘Will’”
   
And the query will look like this.
```graphql
query {
	find(type: Actor, arcql: "name ^ 'Will'") {
		... on HypiFilterConnection {
			edges {
				node {
					... on Actor {
						name
						hypi {
							id
						}
					}
				}
			}
		}
	}
}
```
As expected, the response contains only the actors with the names starting with ‘Will’

```json
{
  "data": {
    "find": {
      "edges": [
        {
          "node": {
            "name": "Will Smith",
            "hypi": {
              "id": "01ED4X75AG1G96PDC8A231KTC8"
            }
          }
        },
        {
          "node": {
            "name": "Will Werrell",
            "hypi": {
              "id": "01ED506E9GQ3Z75RHE0609RNYN"
            }
          }
        }
      ]
    }
  }
}
```

## Delete

Hypi offers **soft delete** and **hard delete** of objects.

**1. Soft Delete**

Soft deleting an object in the **Hypi Universe**, is referred to as ‘**trashing**’ an object.

The field “trashed”, which is a part of the **Magic Hypi object** is used to track that.

The **trash** `mutation` accepts two parameters, the `Type` of the object and an `arcql` query. 
It will trash up to 25 of the objects that match the query. 
The mutation call looks like this.

```graphql
mutation {
	trash(type: Actor, arcql: "hypi.id='01ED4X75AG1G96PDC8A231KTC8'")
}
```
The trash mutation returns the number of trashed items
```json
{
  "data": {
    "trash": 1
  }
}
```
We can verify that the item has been trashed by calling the **get** query with the `id` of the trashed object.

It should not return any data.

You can find include the trashed items in searches by calling find with the parameter `“includeTrashed: True”`

The **untrash** `mutation` accepts two parameters, the `Type` of the object and an `arcql` query. It will untrash all of the objects that match the query. 

The mutation call looks like this.

```graphql
mutation {
	untrash(type: Actor, arcql: "hypi.id='01ED4X75AG1G96PDC8A231KTC8'")
}
```
The untrash mutation returns the number of untrashed items
```json
{
  "data": {
    "untrash": 1
  }
}
```

**2. Hard Delete**

Hard delete is done by calling the **delete** `mutation`. The delete mutation accepts two parameters, the `Type` of the object and an `arcql` query. It will delete up to 25 of the objects that match the query. The mutation call looks like this.

```graphql
mutation {
	delete(type: Actor, arcql: "hypi.id='01ED4X75AG1G96PDC8A231KTC8'")
}
```
The delete mutation returns the number of deleted items
```json
{
  "data": {
    "delete": 1
  }
}
```
Once an object has been hard deleted, there is no way to recover it.

>In this tutorial you learned about how to perform CRUD operations on your Hypi GraphQL API.
>Next you can navigate to your instance’s Api Editor and try these operations out for yourself.
