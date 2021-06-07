## Getting started
This tutorial will show you how to use the middlewares in this **[commercetools Typescript SDK](https://github.com/commercetools/commercetools-sdk-typescript/)** and **[GraphQL API](https://docs.commercetools.com/api/graphql)** to get a simple commercetools Javascript app running.

### Create a API client
[Create API client](https://docs.commercetools.com/tutorials/getting-started#creating-an-api-client) from Merchant Center. If you do not have account [follow the steps to create a free trial account](https://docs.commercetools.com/tutorials/getting-started#first-steps). 
In this guide, we’ll be calling a method of commercetools API using TypeScript SDK and GraphQL API to get the settings of a commercetools project. The commercetools API is the foundation of the commercetools platform, and almost every commercetools client app uses it. Aside from getting project information, the commercetools API allows clients to call methods that can be used for everything from creating products to updating an order’s status. Before we can call any methods, we need to configure our new app to obtain an access token.

### Set up your local project
If you don’t already have a project, let’s create a new one. In an empty directory, you can initialize a new project using the following command:

```
$ npm init -y
```
After you’re done, you’ll have a new package.json file in your directory.

#### Getting client credentials to use the commercetools API
Create a new file called `.env` in this directory. Select **Environment Variables (.env)** in the dropdown on the API client details page mentioned in the previous step and copy client credentials to the `.env` file. Should have values for these variables `CTP_PROJECT_KEY`, `CTP_CLIENT_SECRET`, `CTP_CLIENT_ID`, `CTP_AUTH_URL`, `CTP_API_URL` and `CTP_SCOPES`.

Create a new file called `project.js` in this directory and add the following code:
```js
  const { createClient } = require('@commercetools/sdk-client')
  const { createAuthMiddlewareForClientCredentialsFlow } = require('@commercetools/sdk-middleware-auth')
  const { createHttpMiddleware } = require('@commercetools/sdk-middleware-http')
  const { createApiBuilderFromCtpClient } = require("@commercetools/typescript-sdk")
  const fetch = require('node-fetch')
  require('dotenv').config()
  //reference API client credentials from environment variables
  const {
    CTP_PROJECT_KEY,
    CTP_CLIENT_SECRET,
    CTP_CLIENT_ID,
    CTP_AUTH_URL,
    CTP_API_URL,
    CTP_SCOPES,
  } = process.env
  const projectKey = CTP_PROJECT_KEY
  const authMiddleware = createAuthMiddlewareForClientCredentialsFlow({
    host: CTP_AUTH_URL,
    projectKey,
    credentials: {
      clientId: CTP_CLIENT_ID,
      clientSecret: CTP_CLIENT_SECRET,
    },
    scopes: [CTP_SCOPES],    fetch,
  })
  const httpMiddleware = createHttpMiddleware({
    host: CTP_API_URL,
    fetch,
  })
  const client = createClient({
    middlewares: [authMiddleware, httpMiddleware],
  })
  // Create an API root from API builder of commercetools platform client
  const apiRoot = createApiBuilderFromCtpClient(client);
  console.log('Getting started with commercetools Typescript SDK and GraphQL API');
```
This code creates a **client**, which uses `authMiddleware` and `httpMiddleware`. The `httpMiddleware` reads the `clientId` and `clientSecret` from environment variables. 

Install the `@commercetools/sdk-client`, `@commercetools/sdk-middleware-auth`, `@commercetools/sdk-middleware-http`, `@commercetools/typescript-sdk` and `dotenv` packages and save it to your `package.json` dependencies using the following command:

```
$ npm install @commercetools/sdk-client @commercetools/sdk-middleware-auth @commercetools/sdk-middleware-http @commercetools/typescript-sdk dotenv
```

Run the program using the following command:
```
$ node project.js
Getting started with commercetools Typescript SDK and GraphQL API
```
If you see the same output as above, we’re ready to start.

### Get commercetools project settings with the commercetools GraphQL API
In this guide, we’ll get project settings information using commercetools GraphQL API. 

Re-open `project.js` and add the following code:
```js
// GraphQL query to get commercetools project settings
const projectSettingsQuery = `
    query {
        project {
        name
        languages
        currencies
        countries
        version
        createdAt
        }
    }
`;

(async () => {
    try {
       await apiRoot.withProjectKey({projectKey}).graphql()
            .post({
                body : {
                    query: projectSettingsQuery,
                    variables: {}
                }
            })
            .execute()
            .then(data => {
                console.log('Project information --->', data);
            })
            .catch(error => {
                console.log('ERROR --->', error);
            })
    } catch (error) {
        console.log('ERROR --->', error);
    }
})();
```
Then the client will **execute** get project information request from `apiRoot` using **TypeScript SDK** and **GraphQL** query[projectSettingsQuery] to get project settings. `projectSettingsQuery` retrieves project's `name`, `languages`,`currencies`,`countries`, `version` and `createdAt` fields. To explore commercetools GraphQL API, you can use an interactive [GraphiQL environment](https://github.com/graphql/graphiql/tree/main/packages/graphiql#readme) available as a part of our [ImpEx & API Playground](https://docs.commercetools.com/docs/login).

Run the program. The output should look like the following if the request is successful:

```shell
$ node project.js
Getting started with commercetools Typescript SDK and GraphQL API
Project information ---> { 
    body: { 
        data: { 
            project: <projectData> 
        } 
    }, 
    statusCode: 200 
}
```

### Next Steps
You just built your first commercetools Javascript app with TypeScript SDK and GraphQL API! 🥳🎉

There’s plenty more to learn and explore about this TypeScript SDK and the commercetools platform. Here are some advanced tutorials and links:
- [Create a new customer using GraphQL mutation and TypeScript SDK](#create-a-new-customer)
- [How to use where clause with GraphQL query using TypeScript SDK](#how-to-use-where-clause-with-graphql-query-using-typescript-sdk)
- [commercetools developer tutorials](https://docs.commercetools.com/tutorials/)
- [commercetools JS SDK training](https://github.com/commercetools/commercetools-js-sdk-v2-training)


## Create a new Customer using commercetools GraphQL API and TypeScript SDK
This tutorial will show you how to create a new customer using **[commercetools Typescript SDK](https://github.com/commercetools/commercetools-sdk-typescript/)** and **[GraphQL](https://docs.commercetools.com/api/graphql)** on your commercetools project. Assuming you already finished [Getting started with TypeScript SDK and GraphQL](#getting-started) tutorials and basic project steps are not repeated here again. 

### Set up `customer.js` file
Create a new file called `customer.js` in your project root directory and add the following code:

```js
const { createClient } = require('@commercetools/sdk-client')
  const { createAuthMiddlewareForClientCredentialsFlow } = require('@commercetools/sdk-middleware-auth')
  const { createHttpMiddleware } = require('@commercetools/sdk-middleware-http')
  const { createApiBuilderFromCtpClient } = require("@commercetools/typescript-sdk")
  const fetch = require('node-fetch')
  require('dotenv').config()
  //reference API client credentials from environment variables
  const {
    CTP_PROJECT_KEY,
    CTP_CLIENT_SECRET,
    CTP_CLIENT_ID,
    CTP_AUTH_URL,
    CTP_API_URL,
    CTP_SCOPES,
  } = process.env
  const projectKey = CTP_PROJECT_KEY
  const authMiddleware = createAuthMiddlewareForClientCredentialsFlow({
    host: CTP_AUTH_URL,
    projectKey,
    credentials: {
      clientId: CTP_CLIENT_ID,
      clientSecret: CTP_CLIENT_SECRET,
    },
    scopes: [CTP_SCOPES],    fetch,
  })
  const httpMiddleware = createHttpMiddleware({
    host: CTP_API_URL,
    fetch,
  })
  const client = createClient({
    middlewares: [authMiddleware, httpMiddleware],
  })
  // Create an API root from API builder of commercetools platform client
  const apiRoot = createApiBuilderFromCtpClient(client);
  console.log('Create a new customer using GraphQL and TypeScript SDK');
```
Environment variables and Nodejs dependencies `@commercetools/sdk-client`, `@commercetools/sdk-middleware-auth`,`@commercetools/sdk-middleware-http`, `@commercetools/typescript-sdk`, and `dotenv` are already installed as part of [Getting started with TypeScript SDK and GraphQL](../getting-started-with-graphql-ts-sdk/getting-started.md) tutorial. Back at the command line, run the program using the following command:
```
$ node customer.js
Create a new customer using GraphQL and TypeScript SDK
```
If you see the same output as above, we’re ready to start.

### Create an API client
You already have API client from the [Getting started with TypeScript SDK and GraphQL](../getting-started-with-graphql-ts-sdk/getting-started.md) tutorial on `project.js` file.

### Create GraphQL query and mutation
Add the following code to `customer.js`.

```js
// New customer data
const createCustomerMutationVariable = {
    "newCustomer": {
      "email": "your.test-1@test.com",
      "password": "123",
      "firstName": "yourFirstName", 
      "lastName": "yourLastName"
    }
  };

// mutation to create new customer
const createCustomerMutation = `
    mutation createCustomer ($newCustomer: CustomerSignUpDraft!) {
        customerSignUp (draft: $newCustomer) {
            customer {
                id
            }
        }
    }
`;

// GraphQL query to get Customer `email` and `firstName`
const getCustomerByIdQuery = `
    query ($id: String) {
        customer (id: $id) {
          email
          firstName
        }
    }
`;
```

`createCustomerMutationVariable` contains the new customer data and mandatory field `email` and `password` and optional fields `firstName` and `lastName`. To find out list of all possible fields you can refer [CustomerDraft](https://docs.commercetools.com/api/projects/customers#customerdraft) documentation. Make sure you pass unique `email` value, you will get an error if try to create a new customer using already existing customer's email. 

`createCustomerMutation` is the GraphQL **mutation** to create a **new customer** on the commercetools project and returns `id`.

`getCustomerByIdQuery` is the GraphQL **query** to get newly created customer info `email` and `firstName` by `id`.

To explore commercetools GraphQL API you can use an interactive [GraphiQL environment](https://github.com/graphql/graphiql/tree/main/packages/graphiql#readme) which is available as a part of [ImpEx & API Playground](https://docs.commercetools.com/docs/login).

### Call API to create new Customer using TypeScript SDK and GraphQL API

Add the following code to `customer.js`.
```js
// Create a new customer and return customer id
const createNewCustomer = async () => {
    const result  = await apiRoot.withProjectKey({projectKey}).graphql().post({
        body : {
            query: createCustomerMutation,
            variables: createCustomerMutationVariable,
        }
    }).execute()

    // Get customerId from the result
    const {
        body: {
            data: {
                customerSignUp: {
                    customer: {
                        id:customerId
                    }
                }
            }
        }
    } = result
    
    return customerId
};

// Get customer's email and firstName by customer id
const getCustomerById = async (customerId) => apiRoot.withProjectKey({projectKey}).graphql().post({
        body: {
            query: getCustomerByIdQuery,
            variables: {
                id: customerId
            }
        }
    })
    .execute();

(async () => {
    try {
        const newlyCreatedCustomerId = await createNewCustomer()
        const newlyCreatedCustomer = await getCustomerById(newlyCreatedCustomerId)
        console.log('Newly created customer info ---->', JSON.stringify(newlyCreatedCustomer))
    } catch (error) {
        console.log('ERROR --->', error)
    }
})();
```

Run the program. The output should look like the following if the request is successful:
```
$ node customer.js
Create a new customer using GraphQL and TypeScript SDK
Newly created customer info ----> {"body":{"data":{"customer":{"email":"your.test@test.com","firstName":"yourFirstName"}}},"statusCode":200}
```


## How to use where clause with GraphQL query using TypeScript SDK
This tutorial will show you how to use **where clause** with **[GraphQL query](https://docs.commercetools.com/api/graphql)** and **[commercetools Typescript SDK](https://github.com/commercetools/commercetools-sdk-typescript/)**. Assuming you already finished [Getting started with TypeScript SDK and GraphQL](#getting-started) tutorials and basic project steps are not repeated here again. 

### Set up `whereClauseQuery.js` file
Create a new file called `whereClauseQuery.js` in your project root directory and add the following code:

```js
const { createClient } = require('@commercetools/sdk-client')
  const { createAuthMiddlewareForClientCredentialsFlow } = require('@commercetools/sdk-middleware-auth')
  const { createHttpMiddleware } = require('@commercetools/sdk-middleware-http')
  const { createApiBuilderFromCtpClient } = require("@commercetools/typescript-sdk")
  const fetch = require('node-fetch')
  require('dotenv').config()
  //reference API client credentials from environment variables
  const {
    CTP_PROJECT_KEY,
    CTP_CLIENT_SECRET,
    CTP_CLIENT_ID,
    CTP_AUTH_URL,
    CTP_API_URL,
    CTP_SCOPES,
  } = process.env
  const projectKey = CTP_PROJECT_KEY
  const authMiddleware = createAuthMiddlewareForClientCredentialsFlow({
    host: CTP_AUTH_URL,
    projectKey,
    credentials: {
      clientId: CTP_CLIENT_ID,
      clientSecret: CTP_CLIENT_SECRET,
    },
    scopes: [CTP_SCOPES],    fetch,
  })
  const httpMiddleware = createHttpMiddleware({
    host: CTP_API_URL,
    fetch,
  })
  const client = createClient({
    middlewares: [authMiddleware, httpMiddleware],
  })
  // Create an API root from API builder of commercetools platform client
  const apiRoot = createApiBuilderFromCtpClient(client);
  console.log('where clause query with GraphQL and TypeScript SDK');
```

Environment variables and Nodejs dependencies `@commercetools/sdk-client`, `@commercetools/sdk-middleware-auth`,`@commercetools/sdk-middleware-http`, `@commercetools/typescript-sdk`, and `dotenv` are already installed as part of [Getting started with TypeScript SDK and GraphQL](#getting-started) tutorial. Back at the command line, run the program using the following command:
```
$ node whereClauseQuery.js
where clause query with GraphQL and TypeScript SDK
```
If you see the same output as above, we’re ready to start.

### Create GraphQL query and mutation
Add the following code to `whereClauseQuery.js`.

```js
// where clause with query by customer id
const whereClauseCustomerIdVariable = {
    "where": "id=\"<your-customer-id>\""
  };

// GraphQL query to get customer `email` and `firstName` using where clause and customer id query predicate
const getCustomerByWhereClauseQuery = `
    query ($where: String) {
        customers (where: $where) {
          results {
            email
            firstName
          }
        }
    }
`;
```

`whereClauseCustomerIdVariable` contains where clause with value `"id=\"<your-customer-id>\""`. For details about `where` query, refer [Query Predicates](https://docs.commercetools.com/api/predicates/query) documentation. Replace `<your-customer-id>` with your customer [id](https://docs.commercetools.com/api/general-concepts#identifier) for which you would like to retrieve customer `email` and `firstName`.

`getCustomerByWhereClauseQuery` is the GraphQL **query** to get  customer info `email` and `firstName` by using **where clause**  and **customer id** query predicate.

To explore commercetools GraphQL API, you can use an interactive [GraphiQL environment](https://github.com/graphql/graphiql/tree/main/packages/graphiql#readme) available as a part of [ImpEx & API Playground](https://docs.commercetools.com/docs/login).

### Call API to get customer information using TypeScript SDK and GraphQL

Add the following code to `whereClauseQuery.js`.
```js
// Get customer's email and firstName by customer id
const getCustomerByWhereClause = async () => apiRoot.withProjectKey({projectKey}).graphql().post({
        body: {
            query: getCustomerByWhereClauseQuery,
            variables: whereClauseCustomerIdVariable
        }
    })
    .execute();

(async () => {
    try {
        const result = await getCustomerByWhereClause()
        console.log('Customer info ---->', JSON.stringify(result))
    } catch (error) {
        console.log('ERROR --->', error)
    }
})();
```

Run the program. The output should look like the following if the request is successful:
```
$ node whereClauseQuery.js
where clause query with GraphQL and TypeScript SDK
Customer info ----> {"body":{"data":{"customer":{"email":"your.test@test.com","firstName":"yourFirstName"}}},"statusCode":200}
```
