# Attacking GraphQL

## General idea

graphQL is query language for API that lets the client request the needed data 

**key Components:**

- Query -> read data   `{ "query" : blah blah blah}`

- Mutation -> modify data   `{ "mutation" : blah blah blah}`

- Subscription -> real-time updates   `{ "subscription" : blah blah blah}`

## Identifying graphQL:

usually its in the `/graphql` or  `/api/graphql` or `query` endpoints, graphQL relies on one endpoint to fetch all data unlike the REST API which relies on different endpoints.

## Identifying the graphQL engine:

using tool like **Graphw00f** to identify graphQL engine and then see what security features it has

**INTROSPECTION** : thats the first thing to do after finding graphQL API call,  introspection is a GraphQL feature that enables users to query the GraphQL API about the structure of the backend system.

> **REMARK**: if graphQL introspection is disabled by the backend server for any reason there are some tricks to bypass it like:
> 
> 1. changing the request type from post to get for example
> 
> 2. polluting the query by dding new lines, spaces , comments in the introspection query
> 
> 3. sending unvalid queries and the sever might leak field names in the error response
> 
> 4. brute force fields names using a tool called clairvoyance

**identifying**  the graphQL types using the query: 

```
{ __schema { types { name fields{ name } } }
```

The results contain types as **int** or **Boolean** or custom types as **UserObject** 

**identifying** the graphQL queries using the query:

```
{
  __schema {
    queryType {
      fields {
        name
        description
      }
    }
  }
}
```

**Standard GraphQL introspection query**:

```json
{
  "query": "{ __schema { queryType { name } mutationType { name } subscriptionType { name } types { kind name description fields { name description args { name description type { kind name ofType { kind name ofType { kind name } } } defaultValue } type { kind name ofType { kind name ofType { kind name } } } isDeprecated deprecationReason } inputFields { name description type { kind name ofType { kind name } } defaultValue } interfaces { kind name } enumValues { name description isDeprecated deprecationReason } possibleTypes { kind name } } directives { name description locations args { name description type { kind name ofType { kind name } } defaultValue } } } }"
}
```

this query says > dump the entire schema of a GraphQL API

From `__schema`, it extracts:

- **Queries available**
- **Mutations available**
- **All types (objects, enums, inputs, interfaces)**
- **Fields + arguments**
- **Descriptions (docs)**
- **Deprecated fields**
- **Directives**

after that, im free to use [GraphQL Voyager](https://apis.guru/graphql-voyager/) to visualize the schema in form of tables. 

## Types of attacks:

#### IDOR:

broken authorization, particularly IDOR are ocmmon in GraphQL:

1. **first identifying IDOR**: when there is for example a request being made to `/graphql`  which looks like `{ "query" : "{user(username = \" idk\") { id username password }" }` try to change the username(try valid username find out how fuck urself) and see if u get something, 

2. **exploiting IDOR**: we try to identify the data that can be accessed without authorization. using : 
   
   ```For
   {
     __type(name: "UserObject") {
       name
       fields {
         name
         type {
           name
           kind
         }
       }
     }
   }
   ```
   
   the result will have what the **user** object contains like password or msg field. 

## 

## Injection Attacks

when requesting field like that: 

```graphql
{
  "query": "{ user(username: idk) { username password } }"
}
```

the application may be vulnerable to SQL injection for example if it requires the data like `UNION SELECT 1,2,YOUR_DATA,4,5,6` so since we control the variable username  we can perform an attack and see where it takes us 

> important note: every paramater is a potential vulnerability surface, **every**  paramater worth checking against injection attacks such as SQL or XSS or SSTI ...

## Mutation attacks

mutations are GraphQL queries to modify server data, to perform a mutation attack we  start by identifying all mutations supported by the backend and their arguments using this query 

```graphql
{
  "query": "query { __schema { mutationType { name fields { name args { name defaultValue type { ...TypeRef } } } } } } fragment TypeRef on __Type { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name ofType { kind name } } } } } } } }"
}
```

from the result we identify mutation **X** which requires input from the user. 
we next query all fields of the mutation **X** using this query: 

```graphql
{
  "query": "{ __type(name: \"X\") { name inputFields { name description defaultValue } } }"
}
```

## Tools:

### graph00f:

```shell
 cd ~/tools/graphw00f

 python3 main.py -f -t <url>
```

then we have the [Threat MATRIX](https://github.com/nicholasaleks/graphql-threat-matrix/)  which provides information about the identified GraphQL engine. 

## clairvoyance:

**clairvoyance** is a tool to draw the graphQL schema when the introspection is disabled 

```shell
clairvoyance <url/graphql> -o <output_file.json>
```

## Graphql-cop:

**graphQL-cop** is a tool to identify possible attacks in graphql endpoint, for example: 

```shell
cd ~/tools/graphql-cop
source venv/bin/activate
python3 graphql-cop -t <url/graphql>
```
