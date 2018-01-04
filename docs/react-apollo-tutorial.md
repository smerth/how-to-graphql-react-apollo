# React + Apollo Tutorial - Introduction

[React + Apollo Tutorial](https://www.howtographql.com/react-apollo/0-introduction/)

### Overview

In the previous tutorials, you learned about major concepts and benefits of GraphQL. Now is the time to get your hands dirty and start out with an actual project!

You’re going to build a simple clone of [Hackernews](https://news.ycombinator.com/). Here’s a list of the features the app will have:

- Display a list of links

- Search the list of links

- Users can authenticate

- Authenticated users can create new links

- Authenticated users can upvote links (one vote per link and user)

- Realtime updates when other users upvote a link or create a new one


In this track, you’ll use the following technologies for building the app:



Frontend:

- [React](https://facebook.github.io/react/): Frontend framework for building user interfaces

- [Apollo Client 2.0](https://github.com/apollographql/apollo-client): Fully-featured, production-ready, caching GraphQL client


Backend:

- [Graphcool Framework](https://www.graph.cool/): Flexible backend development framework combining GraphQL + Serverless


You’ll create the React project with [`create-react-app`](https://github.com/facebookincubator/create-react-app), a popular command-line tool that gives you a blank project with all required build configuration already setup.

### Why a GraphQL Client?

In the [Clients](https://www.howtographql.com/advanced/0-clients/) section in the GraphQL part, we already covered the responsibilities of a GraphQL client on a higher level, now it’s time to get more concrete.

In short, you should use a GraphQL client for tasks that are repetitive and agnostic to the app you’re building. For example, being able to send queries and mutations without having to worry about lower-level networking details or maintaining a local cache. This is functionality that you’ll want in any frontend application that’s talking to a GraphQL server - why build it yourself if you can use one of the amazing GraphQL clients out there?

There are a few GraphQL client libraries available. For very simple use cases (such as writing scripts), [`graphql-request`](https://github.com/graphcool/graphql-request) might already be enough for your needs. However, chances are that you’re writing a somewhat larger application where you want to benefit from caching, optimistic UI updates and other handy features. In these cases, you pretty much have the choice between [Apollo Client](https://github.com/apollographql/apollo-client) and [Relay](https://facebook.github.io/relay/).

### Apollo vs Relay

The most common question heard from people that are getting started with GraphQL on the frontend is which GraphQL client they should use. We’ll try to provide a few hints that’ll help you decide which of these clients is the right one for your next project!

[Relay](https://facebook.github.io/relay/) is Facebook’s homegrown GraphQL client that they open-sourced alongside GraphQL in 2015. It incorporates all the learnings that Facebook gathered since they started using GraphQL in 2012. Relay is heavily optimized for performance and tries to reduce network traffic where possible. An interesting side-note is that Relay itself actually started out as a *routing* framework that eventually got combined with data loading responsibilities. It thus evolved into a powerful data management solution that can be used in Javascript apps to interface with GraphQL APIs.

The performance benefits of Relay come at the cost of a notable learning curve. Relay is a pretty complex framework and understanding all its bits and pieces does require some time to really get into it. The overall situation in that respect has improved with the release of the 1.0 version, called [Relay Modern](https://facebook.github.io/relay/docs/relay-modern.html), but if you’re for something to *just get started* with GraphQL, Relay might not be the right choice just yet.

[Apollo Client](https://github.com/apollographql/apollo-client) is a community-driven effort to build an easy-to-understand, flexible and powerful GraphQL client. Apollo has the ambition to build one library for every major development platform that people use to build web and mobile applications. Right now there is a Javascript client with bindings for popular frameworks like [React](https://github.com/apollographql/react-apollo), [Angular](https://github.com/apollographql/apollo-angular), [Ember](https://github.com/bgentry/ember-apollo-client) or [Vue](https://github.com/Akryum/vue-apollo) as well as early versions of [iOS](https://github.com/apollographql/apollo-ios) and [Android](https://github.com/apollographql/apollo-android) clients. Apollo is production-ready and has handy features like caching, optimistic UI, subscription support and many more.



# Getting Started

### Backend

Since this is a frontend track, you don’t want to spend too much time setting up the backend. This is why you’ll use the [Graphcool Framework](https://www.graph.cool/) for this tutorial. It provides a fast and easy way to build and deploy GraphQL backends (also called Graphcool *services*).

#### The Data Model

You’ll use the [Graphcool CLI](https://www.graph.cool/docs/reference/cli/overview-kie1quohli/) to build (and deploy) your GraphQL API based on the data model that you need for the app. 

Speaking of the data model, here is what the final version looks like written in the [GraphQL Schema Definition Language](https://www.graph.cool/docs/faq/graphql-sdl-schema-definition-language-kr84dktnp0/) (SDL):

```jsx
type User @model {
  id: ID! @isUnique     # required system field (read-only)
  createdAt: DateTime!  # optional system field (read-only)
  updatedAt: DateTime!  # optional system field (read-only)

  email: String! @isUnique # for authentication
  password: String!        # for authentication

  name: String!
  links: [Link!]! @relation(name: "UsersLinks")
  votes: [Vote!]! @relation(name: "UsersVotes")
}

type Link @model { 
  id: ID! @isUnique     # required system field (read-only)
  createdAt: DateTime!  # optional system field (read-only)
  updatedAt: DateTime!  # optional system field (read-only)

  description: String!
  url: String!
  postedBy: User! @relation(name: "UsersLinks")
  votes: [Vote!]! @relation(name: "VotesOnLink")
}

type Vote @model {
  id: ID! @isUnique     # required system field (read-only)
  createdAt: DateTime!  # optional system field (read-only)

  user: User! @relation(name: "UsersVotes")
  link: Link! @relation(name: "VotesOnLink")
}

```

As you can see from the comments, some fields on your model types are read-only. This means they will be managed for you by the Graphcool Framework.

In general, there are a few things to note about these type definitions:



- Every type annotated with the `@model`-directive will be *mapped* to the database and corresponding CRUD-operations are added to the GraphQL API of your Graphcool service. 

- The `@isUnique`-directive means that the annotated field can never have the same value for two different records of that type (also called *nodes*). Since this is a read-only field, the Graphcool Framework will take care of managing this constraint.

- `createdAt` and `updatedAt` are special fields that are managed by the Graphcool Framework as well. `createdAt` will carry the date for when a node of that type was created, `updatedAt` when it was last updated. 

  ​

#### Creating the GraphQL Server

For starting out, you’re not going to use the full data model that you saw above. That’s because we want to evolve the schema when it becomes necessary for the features you’re going to implement. For now, the data model will contain only the `Link` type.

The first thing you need to do to get your GraphQL server is install the Graphcool CLI with npm.

Open up a terminal window and type the following:

```bash
npm install -g graphcool
```

Now you can go and create the server. There are two steps involved in this:

1. Creating the local file structure that contains all required configuration for your backend. This is done with the [`graphcool init`](https://www.graph.cool/docs/reference/graphcool-cli/commands-aiteerae6l#graphcool-init) command.

2. Configuring the data model and deploying the server with [`graphcool deploy`](https://www.graph.cool/docs/reference/graphcool-cli/commands-aiteerae6l#graphcool-deploy).

   ​


Type the following command into the terminal to bootstrap your Graphcool service:

```bash
graphcool init server
```

This will create a new directory called `server` and place the following files in there:



- `graphcool.yml`: This is the [root configuration](https://www.graph.cool/docs/reference/service-definition/graphcool.yml-foatho8aip) file for your Graphcool service. It tells the Graphcool Framework where to find your data model (and other type definitions), specifies the [*permission rules*](https://www.graph.cool/docs/reference/auth/authorization/overview-iegoo0heez) and provides information about any integrated *serverless functions*.

- `types.graphql`: This specifies the data model for your application, all type definitions are written in GraphQL SDL. 

- `package.json`: If you’re integrating any serverless functions that are using dependencies from npm, you need to list those dependencies here. Note that this file is completely independent from the dependencies of your frontend which you’ll create in a bit. Since this tutorial won’t actually use any serverless functions, you can simply ignore it.

- `src`: The `src` directory is used to for the code of the serverless functions you’re integrating in your Graphcool service. It currently contains the setup for a simple “Hello World”-[resolver](https://www.graph.cool/docs/reference/functions/resolvers-su6wu3yoo2) function (which you can delete if you like). Again, you can ignore this directory since we’re not going to use any functions in this tutorial.



> smerth - OK, so far so good.



Next you need to make sure that the data model of the GraphQL server is correct, so you need to adjust the type definitions in `types.graphql`.

Open `types.graphql` and replace its current contents with the (not yet complete) definition of the `Link`type:

**.../hackernews-react-apollo/server/types.graphql

```javascript
type Link @model {
  id: ID! @isUnique     # required system field (read-only)
  createdAt: DateTime!  # optional system field (read-only)
  updatedAt: DateTime!  # optional system field (read-only)

  description: String!
  url: String!
}
```

As mentioned above, we’ll start with only a sub-part of the actual data model and evolve our schema and API when necessary. This change is all you need to put your GraphQL server into production.

Open a terminal and navigate into the `server` directory. Then deploy the server with the following command:

$.../hackernews-react-apollo/server

```bash
graphcool deploy
```

When prompted, select any of the **Shared Clusters** deployment options, e.g. `shared-eu-west`. 

For any other prompt, you can just hit **Enter** to go with the suggested default value.

> 
>
> Note that this command will open up a browser window first and ask you to authenticate on the Graphcool platform (if you haven’t done so before).
>
> 



> smerth - ok it works



#### Populate The Database & GraphQL Playgrounds

Before you move on to setup the frontend, go ahead and create some initial data in the project so you’ve got something to see once you start rendering data in the app!

You’ll do this by using a GraphQL [Playground](https://github.com/graphcool/graphql-playground) which is an interactive environment that allows you to send queries and mutations. It’s a great way to explore the capabilities of a GraphQL API.

Still in the `server` directory in your terminal, run the following command:

$.../hackernews-react-apollo/server

```bash
graphcool playground
```

> smerth - neat! it opens up an instance of graphiql.



The left pane of the Playground is the *editor* that you can use to write your queries and mutations (and even realtime subscriptions). Once you click the play button in the middle, the response to the request will be displayed in the *results* pane on the right.

Copy the following two mutations into the *editor* pane:

```javascript
mutation CreateGraphcoolLink {
  createLink(
    description: "The coolest GraphQL backend 😎",
    url: "https://graph.cool") {
    id
  }
}

mutation CreateApolloLink {
  createLink(
    description: "The best GraphQL client",
    url: "http://dev.apollodata.com/") {
    id
  }
}

```

Since you’re adding two mutations to the editor at once, the mutations need to have *operation names*. In your case, these are `CreateGraphcoolLink` and `CreateApolloLink`.

Click the *Play*-button in the middle of the two panes and select each mutation from the dropdown exactly once.

![img](https://imgur.com/ZBgeq22.png)

This creates two new `Link` records in the database. You can verify that the mutations actually worked by either viewing the currently stored data in the [data browser](https://graph-cool.netlify.com/docs/reference/console/data-browser-och3ookaeb/) (simply click *DATA* in the left side-menu) or by sending the following query in the already open Playground:

```
{
  allLinks {
    id
    description
    url
  }
}
```

If everything went well, the query will return the following data (the `id`s will of course be different in your case):

```javascript
{
  "data": {
    "allLinks": [
      {
        "id": "cj4jo6xxat8o901420m0yy60i",
        "description": "The coolest GraphQL backend 😎",
        "url": "https://graph.cool"
      },
      {
        "id": "cj4jo6z4it8on0142p7q015hc",
        "description": "The best GraphQL client",
        "url": "http://dev.apollodata.com/"
      }
    ]
  }

```

> smerth - cool, as in graphicool!



### Frontend

#### Creating the App

Next, you are going to create the React project! As mentioned in the beginning, you’ll use `create-react-app`for that.

If you haven’t already, you need to install `create-react-app` using npm:

```bash
npm install -g create-react-app
```

Next, you can use it to bootstrap your React application:

@ root folder

```bash
create-react-app hackernews-react-apollo
```

This will create a new directory called `hackernews-react-apollo` that has all the basic configuration setup. 

Make sure everything works by navigating into the directory and starting the app:

```bash
cd hackernews-react-apollo
yarn start
```

This will open a browser and navigate to `http://localhost:3000` where the app is running. If everything went well, you’ll see the following:

![img](https://imgur.com/Yujwwi6.png)



> smerth - OK!



Next, go ahead and move the `server` directory that contains the definition of your Graphcool service into the `hackernews-react-apollo` directory to manage everything in one place.

To improve the project structure, move on to create two directories, both inside the `src` folder. The first is called `components` and will hold all our React components. Call the second one `styles`, that one is for all the CSS files you’ll use.

Now clean up the existing files accordingly. Move `App.js` into `components` and `App.css` as well as `index.css` into `styles`.



> !!! smerth - moving these files breaks the build so make the following changes

@ App.js

```jsx
import logo from './../logo.svg';
import './../styles/App.css';
```

and @ Index.js

```jsx
import ReactDOM from 'react-dom'
import './styles/index.css'
```





Your project structure should now look as follows:

```bash
.
├── README.md
├── node_modules
├── package.json
├── public
│   ├── favicon.ico
│   ├── index.html
│   └── manifest.json
├── server
│   ├── graphcool.yml
│   ├── package.json
│   ├── types.graphql
│   └── src
│       ├── hello.js
│       └── hello.graphql
├── src
│   ├── App.test.js
│   ├── components
│   │   └── App.js
│   ├── index.js
│   ├── logo.svg
│   ├── registerServiceWorker.js
│   └── styles
│       ├── App.css
│       └── index.css
└── yarn.lock

```



> smerth - OK the app compiles with the new file structure



#### Prepare Styling

This tutorial is about the concepts of GraphQL and how you can use it from within a React application, so we want to spend the least time possible on styling issues. To ease up usage of CSS in this project, you’ll use the [Tachyons](http://tachyons.io/) library which provides a number of CSS classes.

Open `public/index.html` and add a third `link` tag right below the two existing ones that pulls in Tachyons:

**.../hackernews-react-apollo/public/index.html

```html
<link rel="manifest" href="%PUBLIC_URL%/manifest.json">
<link rel="shortcut icon" href="%PUBLIC_URL%/favicon.ico">
<link rel="stylesheet" href="https://unpkg.com/tachyons@4.2.1/css/tachyons.min.css"/>
```

Since we still want to have a bit more custom styling here and there, we also prepared some styles for you that you need to include in the project.

Open `index.css` and replace its content with the following:

**.../hackernews-react-apollo/src/styles/index.css

```css
body {
  margin: 0;
  padding: 0;
  font-family: Verdana, Geneva, sans-serif;
}

input {
  max-width: 500px;
}

.gray {
  color: #828282;
}

.orange {
  background-color: #ff6600;
}

.background-gray {
  background-color: rgb(246,246,239);
}

.f11 {
  font-size: 11px;
}

.w85 {
  width: 85%;
}

.button {
  font-family: monospace;
  font-size: 10pt;
  color: black;
  background-color: buttonface;
  text-align: center;
  padding: 2px 6px 3px;
  border-width: 2px;
  border-style: outset;
  border-color: buttonface;
  cursor: pointer;
  max-width: 250px;
}

```





#### Installing Apollo

Next, you need to pull in the functionality of Apollo Client (and its React bindings) which comes in several packages:

$.../hackernews-react-apollo

```bash
yarn add apollo-client-preset react-apollo graphql-tag graphql
```

Here’s an overview of the packages you just installed:

- ​  [`apollo-client-preset`](https://www.npmjs.com/package/apollo-client-preset) offers some convenience by bundling several packages you need when working with Apollo Client:

  - `apollo-client`

  - `apollo-cache-inmemory`

  - `apollo-link`

  - `apollo-link-http`


  ​

- [`react-apollo`](https://github.com/apollographql/react-apollo) contains the bindings to use Apollo Client with React.

- [`graphql-tag`](https://github.com/apollographql/graphql-tag) is a GraphQL parser. Every GraphQL operation you hand over to Apollo Client will have to be parsed by the `gql` function.

- [`graphql`](https://github.com/graphql/graphql-js) contains Facebook’s reference implementation of GraphQL - Apollo Client uses some of its functionality as well. 


That’s it, you’re ready to write some code! 🚀







#### Configuring the `ApolloClient`

Apollo abstracts away all lower-level networking logic and provides a nice interface to the GraphQL API. In contrast to working with REST APIs, you don’t have to deal with constructing your own HTTP requests any more - instead you can simply write queries and mutations and send them using the `ApolloClient`.

The first thing you have to do when using Apollo is configure your `ApolloClient` instance. It needs to know the endpoint of your GraphQL API so it can deal with the network connections.

Open `src/index.js` and replace the contents with the following:

**src/index.js

```jsx
import React from 'react'
import ReactDOM from 'react-dom'
import './styles/index.css'
import App from './components/App'
import registerServiceWorker from './registerServiceWorker'
// 1
import { ApolloProvider } from 'react-apollo'
import { ApolloClient } from 'apollo-client'
import { HttpLink } from 'apollo-link-http'
import { InMemoryCache } from 'apollo-cache-inmemory'

// 2
const httpLink = new HttpLink({ uri: '__SIMPLE_API_ENDPOINT__' })

// 3
const client = new ApolloClient({
  link: httpLink,
  cache: new InMemoryCache()
})

// 4
ReactDOM.render(
  <ApolloProvider client={client}>
    <App />
  </ApolloProvider>
  , document.getElementById('root')
)
registerServiceWorker()

```

> 
>
> Note: The project that was generated by `create-react-app` uses semicolons and double quotes for strings. All the code that you’re going to add will use **no semicolons** and **single quotes**. You’re also free to delete any existing semicolons and replace double with single quotes 🔥
>
> 



> smerth - ok, done







Let’s try to understand what’s going on in that code snippet:

1. You’re importing the required dependencies from the installed npm packages.

2. Here you create the `HttpLink` that will connect your `ApolloClient` instance with the GraphQL API; you’ll replace the placeholder `__SIMPLE_API_ENDPOINT__` with your actual endpoint in a bit.

3. Now you instantiate the `ApolloClient` by passing in the `httpLink` and a new instance of an `InMemoryCache`.

4. Finally you render the root component of your React app. The `App` is wrapped with the higher-order component `ApolloProvider` that gets passed the `client` as a prop.

   ​




Next you need to replace the placeholder for the GraphQL endpoint with your actual endpoint. But where do you get your endpoint from?

There are two ways for you to get your endpoint. You can either open the [Graphcool Console](https://console.graph.cool/) and click the *Endoints*-button in the bottom-left corner. The second option is to use the CLI.

In the terminal, navigate into the `server` directory and use the following command to get access to the API endpoints of your Graphcools service:

$.../hackernews-react-apollo/server

```bash
graphcool info
```

Copy the endpoint for the `Simple API` and paste it into `src/index.js` to replace the current placeholder `__SIMPLE_API_ENDPOINT__`.

That’s it, you’re all set to start for loading some data into your app! 😎



# Queries: Loading Links

### Preparing the React components

The first piece of functionality you’ll implement in the app is loading and displaying a list of `Link` elements. Let's walk our way up in the React component hierarchy and start with the component that’ll render a single link. 

Create a new file called `Link.js` in the `components` directory and add the following code:

**.../hackernews-react-apollo/src/components/Link.js

```jsx
import React, { Component } from 'react'

class Link extends Component {

  render() {
    return (
      <div>
        <div>{this.props.link.description} ({this.props.link.url})</div>
      </div>
    )
  }
  
  _voteForLink = async () => {
    // ... you'll implement this in chapter 6  
  }

}

export default Link

```

This is a simple React component that expects a `link` in its `props` and renders the link’s `description` and `url`. Easy as pie! 🍰

Next, you’ll implement the component that renders a list of links.

Again, in the `components` directory, go ahead and create a new file called `LinkList.js` and add the following code:

**.../hackernews-react-apollo/src/components/LinkList.js

```jsx
import React, { Component } from 'react'
import Link from './Link'

class LinkList extends Component {

  render() {

    const linksToRender = [{
      id: '1',
      description: 'The Coolest GraphQL Backend 😎',
      url: 'https://www.graph.cool'
    }, {
      id: '2',
      description: 'The Best GraphQL Client',
      url: 'http://dev.apollodata.com/'
    }]

    return (
      <div>
        {linksToRender.map(link => (
          <Link key={link.id} link={link}/>
        ))}
      </div>
    )
  }

}

export default LinkList

```

Here, you’re using mock data for now to make sure the component setup works. You’ll soon replace this with some actual data loaded from the server - patience, young Padawan!

To complete the setup, open `App.js` and replace the current contents with the following:

**.../hackernews-react-apollo/src/components/App.js

```jsx
import React, { Component } from 'react'
import LinkList from './LinkList'

class App extends Component {
  render() {
    return (
      <LinkList />
    )
  }
}

export default App

```

Run the app to check if everything works so far! The app should now display the two links from the `linksToRender` array:

![img](https://imgur.com/FlMveso.png)

> Smerth - supercool



### Writing the GraphQL Query

You’ll now load the actual links that are stored on the server. The first thing you need to do for that is define the GraphQL query that you want to send to the API. 

Here is what it looks like:

```javascript
query AllLinks {
  allLinks {
    id
    createdAt
    description
    url
  }
}

```

You could now simply execute this query in a Playground and retrieve the results from your GraphQL server. But how can you use it inside your JavaScript code?

### Queries with Apollo Client

When using Apollo, you’ve got two ways of sending queries to the server.

The first one is to directly use the [`query`](https://www.apollographql.com/docs/react/reference/index.html#ApolloClient.query) method on the `ApolloClient` directly. This is a more *imperative* way of fetching data and will allow you to process the response as a *promise*.

A practical example would look as follows:

```javascript
client.query({
  query: gql`
    query AllLinks {
      allLinks {
        id
      }
    }
  `
}).then(response => console.log(response.data.allLinks))

```

A more idiomatic way when using React however is to use Apollo’s higher-order component [`graphql`](https://www.apollographql.com/docs/react/basics/setup.html#graphql) to wrap your React component with a query.

With this approach, all you need to do when it comes to data fetching is write the GraphQL query and `graphql`will fetch the data for you under the hood and then make it available in your component’s props.

In general, the process for you to add some data fetching logic will be very similar every time:

1. ​write the query as a JavaScript constant using the `gql` parser function
2. use the `graphql` container to wrap your component with the query
3. access the query results in the component’s `props`




Open up `LinkList.js` and add the query to the bottom of the file, also replacing the current `export LinkList` statement:

**.../hackernews-react-apollo/src/components/LinkList.js

```jsx
// 1
const ALL_LINKS_QUERY = gql`
  # 2
  query AllLinksQuery {
    allLinks {
      id
      createdAt
      url
      description
    }
  }
`

// 3
export default graphql(ALL_LINKS_QUERY, { name: 'allLinksQuery' }) (LinkList)


```

What’s going on here?

1. First, you create the JavaScript constant called `ALL_LINKS_QUERY` that stores the query. The `gql` function is used to parse the plain string that contains the GraphQL code (if you’re unfamililar with the backtick-syntax, you can read up on JavaScript’s [tagged template literals](http://wesbos.com/tagged-template-literals/)).

2. Now you define the actual GraphQL query. `AllLinksQuery` is the *operation name* and will be used by Apollo to refer to this query in its internals. (Notice the `#` which denotes a GraphQL comment). 

3. Finally, you’re using the `graphql` container to combine the `LinkList` component with the `ALL_LINKS_QUERY`. Note that you’re also passing an options object to the function call where you specify a `name` to be `allLinksQuery`. This is the **name of the prop that Apollo injects** into the `LinkList`component. If you didn’t specify it here, the injected prop would be called `data`.

   ​


For this code to work, you also need to import the corresponding dependencies. Add the following two lines to the top of the file, right below the other import statements:

**.../hackernews-react-apollo/src/components/LinkList.js

```jsx
import { graphql } from 'react-apollo'
import gql from 'graphql-tag'
```

Awesome, that’s all your data fetching code, can you believe that?

You can now finally remove the mock data and render actual links that are fetched from the server.

Still in `LinkList.js`, update `render` as follows:

**.../hackernews-react-apollo/src/components/LinkList.js

```jsx
render() {

  // 1
  if (this.props.allLinksQuery && this.props.allLinksQuery.loading) {
    return <div>Loading</div>
  }

  // 2
  if (this.props.allLinksQuery && this.props.allLinksQuery.error) {
    return <div>Error</div>
  }

  // 3
  const linksToRender = this.props.allLinksQuery.allLinks

  return (
    <div>
      {linksToRender.map(link => (
        <Link key={link.id} link={link}/>
      ))}
    </div>
  )
}

```

Let’s walk through what’s happening in this code. As expected, Apollo injected a new prop into the component called `allLinksQuery`. This prop itself has 3 fields that provide information about the *state* of the network request:

1. `loading`: Is `true` as long as the request is still ongoing and the response hasn’t been received.

2. `error`: In case the request fails, this field will contain information about what exactly went wrong.

3. `allLinks`: This is the actual data that was received from the server. It’s an array of `Link` elements.

   ​


> In fact, the injected prop contains even more functionality. You can read more in the [documentation](https://www.apollographql.com/docs/react/basics/queries.html#graphql-query-data).



That’s it! Go ahead and run `yarn start` again. You should see the exact same screen as before.



> Smerth - cool like Iceland...



# Mutations: Creating Links

In this section, you’ll learn how you can send mutations with Apollo. It’s actually not that different from sending queries and follows the same three steps that were mentioned before, with a minor (but logical) difference in step 3:

1. write the mutation as a Javascript constant using the `gql` parser function

2. use the `graphql` container to wrap your component with the mutation

3. use the mutation function that gets injected into the component’s props

   ​


### Preparing the React components

Like before, let’s start by writing the React component where users will be able to add new links.

Create a new file in the `src/components` directory and call it `CreateLink.js`. Then paste the following code into it:

**.../hackernews-react-apollo/src/components/CreateLink.js

```jsx
import React, { Component } from 'react'

class CreateLink extends Component {

  state = {
    description: '',
    url: ''
  }

  render() {
    return (
      <div>
        <div className='flex flex-column mt3'>
          <input
            className='mb2'
            value={this.state.description}
            onChange={(e) => this.setState({ description: e.target.value })}
            type='text'
            placeholder='A description for the link'
          />
          <input
            className='mb2'
            value={this.state.url}
            onChange={(e) => this.setState({ url: e.target.value })}
            type='text'
            placeholder='The URL for the link'
          />
        </div>
        <button
          onClick={() => this._createLink()}
        >
          Submit
        </button>
      </div>
    )
  }

  _createLink = async () => {
    // ... you'll implement this in a bit
  }

}

export default CreateLink

```

This is a standard setup for a React component with two `input` fields where users can provide the `url` and `description` of the `Link` they want to create. The data that’s typed into these fields is stored in the component’s `state` and will be used in `_createLink` when the mutation is sent.

### Writing the Mutation

But how can you now actually send the mutation? Let’s follow the three steps from before.

First you need to define the mutation in your JavaScript code and wrap your component with the `graphql`container. You’ll do that in a similar way as with the query before.

In `CreateLink.js`, add the following statement to the bottom of the file (also replacing the current `export default CreateLink` statement):

**.../hackernews-react-apollo/src/components/CreateLink.js

```jsx
// 1
const CREATE_LINK_MUTATION = gql`
  # 2
  mutation CreateLinkMutation($description: String!, $url: String!) {
    createLink(
      description: $description,
      url: $url,
    ) {
      id
      createdAt
      url
      description
    }
  }
`

// 3
export default graphql(CREATE_LINK_MUTATION, { name: 'createLinkMutation' })(CreateLink)

```

Let’s take a closer look again to understand what’s going on:

1. ​

2. You first create the JavaScript constant called `CREATE_LINK_MUTATION` that stores the mutation.

3. Now you define the actual GraphQL mutation. It takes two arguments, `url` and `description`, that you’ll have to provide when calling the mutation. 

4. Lastly, you’re using the `graphql` container to combine the `CreateLink` component with the `CREATE_LINK_MUTATION`. The specified `name` again refers to the name of the prop that’s injected into `CreateLink`. This time, a function will be injected that’s called `createLinkMutation` and that you can call and pass in the required arguments. 

5. ​

Before moving on, you need to import the Apollo dependencies. Add the following to the top of `CreateLink.js`:

**.../hackernews-react-apollo/src/components/CreateLink.js

```jsx
import { graphql } from 'react-apollo'
import gql from 'graphql-tag'
```

Let’s see the mutation in action!



Still in `CreateLink.js`, implement the `_createLink` mutation as follows:

**.../hackernews-react-apollo/src/components/CreateLink.js

```jsx
_createLink = async () => {
  const { description, url } = this.state
  await this.props.createLinkMutation({
    variables: {
      description,
      url
    }
  })
}

```

As promised, all you need to do is call the function that Apollo injects into `CreateLink` and pass the variables that represent the user input. 

Go ahead and see if the mutation works. To be able to test the code, open `App.js` and change `render` to looks as follows:

**.../hackernews-react-apollo/src/components/App.js

```jsx
render() {
  return (
    <CreateLink />
  )
}

```

Next, import the `CreateLink` component by adding the following statement to the top of `App.js`:

**.../hackernews-react-apollo/src/components/App.js

```jsx
import CreateLink from './CreateLink'
```

Now, run `yarn start`, you’ll see the following screen:

![img](https://imgur.com/AJNlEfj.png)

Two input fields and a *submit*-button - not very pretty but functional.

Enter some data into the fields, e.g.:

- **Description**: `The best learning resource for GraphQL`

- **URL**: `www.howtographql.com`

  ​


Then click the *submit*-button. You won’t get any visual feedback in the UI, but let’s see if the query actually worked by checking the current list of links in a Playground.

> 
>
> At this point, you’ll only see the new link after refreshing the page. In chapter six you will take care of updating the UI properly after a mutation was performed.
>
> 

Type `graphcool playground` into a terminal and send the following query:

```
{
  allLinks {
    description
    url
  }
}
```

You’ll see the following server response:

```javascript
{
  "data": {
    "allLinks": [
      {
        "description": "The best learning resource for GraphQL",
        "url": "www.howtographql.com"
      },
      // ...
    ]
  }
}

```

Awesome! The mutation works, great job! 💪



> smerth - cool as a kukumber



# Routing

In this section, you’ll learn how to use the [`react-router`](https://github.com/ReactTraining/react-router) library with Apollo to implement some navigation functionality!

### Install Dependencies

First add the dependency to the app. Open a terminal, navigate to your project directory and type: 

$.../hackernews-react-apollo

```bash
yarn add react-router react-router-dom
```

### Create a Header

Before you’re moving on to configure the different routes for your application, you need to create a `Header`component that users can use to navigate between the different parts of your app.

Create a new file in `src/components` and call it `Header.js`. Then paste the following code inside of it:

**.../hackernews-react-apollo/src/components/Header.js

```jsx
import React, { Component } from 'react'
import { Link } from 'react-router-dom'
import { withRouter } from 'react-router'

class Header extends Component {

  render() {
    return (
      <div className='flex pa1 justify-between nowrap orange'>
        <div className='flex flex-fixed black'>
          <div className='fw7 mr1'>Hacker News</div>
          <Link to='/' className='ml1 no-underline black'>new</Link>
          <div className='ml1'>|</div>
          <Link to='/create' className='ml1 no-underline black'>submit</Link>
        </div>
      </div>
    )
  }

}

export default withRouter(Header)

```

This simply renders two `Link` components that users can use to navigate between the `LinkList` and the `CreateLink` components. 

> 
>
> Don’t get confused by the “other” `Link` component that is used here. The one that you’re using in the `Header`has nothing to do with the `Link` component that you wrote before, they just happen to have the same name. This [`Link`](https://github.com/ReactTraining/react-router/blob/master/packages/react-router-dom/docs/api/Link.md) stems from the `react-router-dom` package and allows you to navigate between routes inside of your application.
>
> 

> Smerth - in other words react-router-dom actually defines a link component and the props it can take.  You can call it into another component any time you need a link



### Setup routes

You’ll configure the different routes for the app in the project’s root component: `App`. 

Open the corresponding file `App.js` and update `render` to include the `Header` as well as `LinkList` and the `CreateLink` components under different routes:

**.../hackernews-react-apollo/src/components/App.js

```jsx
render() {
  return (
    <div className='center w85'>
      <Header />
      <div className='ph3 pv1 background-gray'>
        <Switch>
          <Route exact path='/' component={LinkList}/>
          <Route exact path='/create' component={CreateLink}/>
        </Switch>
      </div>
    </div>
  )
}

```

For this code to work, you need to import the required dependencies of `react-router-dom`. 

Add the following statement to the top of the file:

**.../hackernews-react-apollo/src/components/App.js

```jsx
import Header from './Header'
import { Switch, Route } from 'react-router-dom'
```

Now you need to wrap the `App` with `BrowserRouter` so that all child components of `App` will get access to the routing functionality.

Open `index.js` and add the following import statement to the top:

**.../hackernews-react-apollo/src/index.js

```jsx
import { BrowserRouter } from 'react-router-dom'
```

Now update `ReactDOM.render` and wrap the whole app with the `BrowserRouter`:

**.../hackernews-react-apollo/src/index.js

```jsx
ReactDOM.render(
  <BrowserRouter>
    <ApolloProvider client={client}>
      <App />
    </ApolloProvider>
  </BrowserRouter>
  , document.getElementById('root')
)
```

That’s it. If you run `yarn start`, you can now access two URLs. `http://localhost:3000/` will render `LinkList`and `http://localhost:3000/create` renders the `CreateLink` component you just wrote in the previous section.

![img](https://imgur.com/I16JzwW.png)

### Implement navigation

To wrap up this section, you need to implement an automatic redirect from the `CreateLink` to `LinkList` after a mutation was performed.

Open `CreateLink.js` and update `_createLink` to look as follows:

**.../hackernews-react-apollo/src/components/CreateLink.js

```jsx
_createLink = async () => {
  const { description, url } = this.state
  await this.props.createLinkMutation({
    variables: {
      description,
      url
    }
  })
  this.props.history.push(`/`)
}

```

After the mutation was performed, `react-router-dom` will now navigate back to the `LinkList` component that’s accessible on the root route: `/`.



# Authentication

In this section, you’ll learn how you can implement authentication functionality with Apollo and Graphcool to provide a login to the user.

### Prepare the React components

As in the sections before, you’ll set the stage for the login functionality by preparing the React components that are needed for this feature. You’ll start by implementing the `Login` component.

Create a new file in `src/components` and call it `Login.js`. Then paste the following code into it:

**.../hackernews-react-apollo/src/components/Login.js

```jsx
import React, { Component } from 'react'
import { GC_USER_ID, GC_AUTH_TOKEN } from '../constants'

class Login extends Component {

  state = {
    login: true, // switch between Login and SignUp
    email: '',
    password: '',
    name: ''
  }

  render() {

    return (
      <div>
        <h4 className='mv3'>{this.state.login ? 'Login' : 'Sign Up'}</h4>
        <div className='flex flex-column'>
          {!this.state.login &&
          <input
            value={this.state.name}
            onChange={(e) => this.setState({ name: e.target.value })}
            type='text'
            placeholder='Your name'
          />}
          <input
            value={this.state.email}
            onChange={(e) => this.setState({ email: e.target.value })}
            type='text'
            placeholder='Your email address'
          />
          <input
            value={this.state.password}
            onChange={(e) => this.setState({ password: e.target.value })}
            type='password'
            placeholder='Choose a safe password'
          />
        </div>
        <div className='flex mt3'>
          <div
            className='pointer mr2 button'
            onClick={() => this._confirm()}
          >
            {this.state.login ? 'login' : 'create account' }
          </div>
          <div
            className='pointer button'
            onClick={() => this.setState({ login: !this.state.login })}
          >
            {this.state.login ? 'need to create an account?' : 'already have an account?'}
          </div>
        </div>
      </div>
    )
  }

  _confirm = async () => {
    // ... you'll implement this in a bit
  }

  _saveUserData = (id, token) => {
    localStorage.setItem(GC_USER_ID, id)
    localStorage.setItem(GC_AUTH_TOKEN, token)
  }

}

export default Login

```

Let’s quickly understand the structure of this new component, which can have two major states:

- One state is **for users that already have an account** and only need to login. In this state, the component will only render two `input` fields for the user to provide their `email` and `password`. Notice that `state.login` will be `true` in this case.

- The second state is for **users that haven’t created an account yet**, and thus still need to sign up. Here, you also render a third `input` field where users can provide their `name`. In this case, `state.login` will be `false`.

  ​

The method `_confirm` will be used to implement the mutations that we need to send for the login functionality.

Next you also need to provide the `constants.js` file that we use to define keys for the credentials that we’re storing in the browser’s `localStorage`.

In `src`, create a new file called `constants.js` and add the following two definitions:

**.../hackernews-react-apollo/src/constants.js

```jsx
export const GC_USER_ID = 'graphcool-user-id'
export const GC_AUTH_TOKEN = 'graphcool-auth-token'
```

With that component in place, you can go and add a new route to your `react-router-dom` setup. 

Open `App.js` and update `render` to include the new route:

**.../hackernews-react-apollo/src/components/App.js

```jsx
render() {
  return (
    <div className='center w85'>
      <Header />
      <div className='ph3 pv1 background-gray'>
        <Switch>
          <Route exact path='/login' component={Login}/>
          <Route exact path='/create' component={CreateLink}/>
          <Route exact path='/' component={LinkList}/>
        </Switch>
      </div>
    </div>
  )
}

```

Also import the `Login` component on top of the same file: 

**.../hackernews-react-apollo/src/components/App.js

```jsx
import Login from './Login'
```

Finally, go ahead and add `Link` to the `Header` that allows the users to navigate to the `Login` page. 

Open `Header.js` and update `render` to look as follows:

**.../hackernews-react-apollo/src/components/Header.js

```jsx
render() {
  const userId = localStorage.getItem(GC_USER_ID)
  return (
    <div className='flex pa1 justify-between nowrap orange'>
      <div className='flex flex-fixed black'>
        <div className='fw7 mr1'>Hacker News</div>
        <Link to='/' className='ml1 no-underline black'>new</Link>
        {userId &&
        <div className='flex'>
          <div className='ml1'>|</div>
          <Link to='/create' className='ml1 no-underline black'>submit</Link>
        </div>
        }
      </div>
      <div className='flex flex-fixed'>
        {userId ?
          <div className='ml1 pointer black' onClick={() => {
            localStorage.removeItem(GC_USER_ID)
            localStorage.removeItem(GC_AUTH_TOKEN)
            this.props.history.push(`/new/1`)
          }}>logout</div>
          :
          <Link to='/login' className='ml1 no-underline black'>login</Link>
        }
      </div>
    </div>
  )
}

```

You first retrieve the `userId` from local storage. If the `userId` is not available, the *submit*-button won’t be rendered any more. That way you make sure only authenticated users can create new links. 

You’re also adding a second button to the right of the `Header` that users can use to login and logout.

Lastly, you need to import the key definitions from `constants.js` in `Header.js`. Add the following statement to the top of file:

**.../hackernews-react-apollo/src/components/Header.js

```jsx
import { GC_USER_ID, GC_AUTH_TOKEN } from '../constants'
```

Here is what the ready component looks like:

![img](https://imgur.com/tBxMVtb.png)

Before you can implement the authentication functionality in `Login.js`, you need to update the running Graphcool service and add authentication on the server-side.

### Enabling Email-and-Password Authentication & Updating the Schema

Authentication in the Graphcool Framework is based on [resolver](https://www.graph.cool/docs/reference/functions/resolvers-su6wu3yoo2) functions that deal with login-functionality by issuing and returning [node tokens](https://www.graph.cool/docs/reference/auth/authentication/authentication-tokens-eip7ahqu5o#node-tokens) which are used to authenticate requests.

Graphcool has a lightweight and flexible [template](https://www.graph.cool/docs/reference/service-definition/templates-zeiv8phail) system that allows to conventiently pull in predefined functionality into a service. You’ll be using the `email-password` template for authentication.

You can use the CLI’s [`add-template`](https://www.graph.cool/docs/reference/graphcool-cli/commands-aiteerae6l#graphcool-add-template) command to use a template in your Graphcool service. This command will perform two major tasks:

- Download the files from Graphcool’s [`templates` repository](https://github.com/graphcool/templates) that are required for the `email-password` template.

- Add commented lines to `graphcool.yml` and `types.graphql` that allow you to “activate” the template’s functionality by uncommenting them and then invoking `graphcool deploy` again. 

  ​


Navigate into the `server` directory inside your project and run the following command:

$../hackernews-react-apollo/server

```bash
graphcool add-template graphcool/templates/auth/email-password
```

This now downloaded six new files and placed them in the `src/email-password` directory. It also added comments to `graphcool.yml` and `types.graphql`.

Next, you have to actually “activate” the templates functionality by uncommenting these lines.

Open `graphcool.yml` and uncomment the definitions for the `signup`, `authenticate` and `loggedInUser`functions:

**.../hackernews-react-apollo/server/graphcool.yml

```yaml
functions:

# added by email-password template: (please uncomment)

  signup:
    type: resolver
    schema: src/email-password/signup.graphql
    handler:
      code: src/email-password/signup.ts

  authenticate:
    type: resolver
    schema: src/email-password/authenticate.graphql
    handler:
      code: src/email-password/authenticate.ts

  loggedInUser:
    type: resolver
    schema: src/email-password/loggedInUser.graphql
    handler:
      code: src/email-password/loggedInUser.ts

```

If you take a look at the code for these functions, you’ll notice that they’re referencing a `User` type that still needs to be added to your data model. In fact, the `add-template` command already wrote this `User` type to `types.graphql` - except that it still has comments.

Open `types.graphql` and uncomment the `User` type:

**.../hackernews-react-apollo/server/types.graphql

```
# added by email-password template: (please uncomment)
type User @model {
  id: ID! @isUnique   
  createdAt: DateTime!
  updatedAt: DateTime!
  email: String! @isUnique
  password: String!
}

```

Before you apply the changes to the running service, you’ll make another modification to your data model by adding the *relation* between the `Link` and the newly added `User` type as well as a new field `name` for the `User`.

Open your type definitions file `types.graphql` and update the `User` and `Link` types as follows:

```
type Link @model {
  id: ID! @isUnique
  createdAt: DateTime!
  updatedAt: DateTime!
  description: String!
  url: String!
  postedBy: User @relation(name: "UsersLinks")
}

type User @model {
  id: ID! @isUnique
  createdAt: DateTime!
  updatedAt: DateTime!
  name: String!
  email: String @isUnique
  password: String
  links: [Link!]! @relation(name: "UsersLinks")
}

```

You added two things to the schema:

- A new field on the `User` type to store the `name` of the user.

- A new relation between the `User` and the `Link` type that represents a one-to-many relationship and expresses that one `User` can be associated with multiple links. The relation manifests itself in the two fields `postedBy` and `links`.

  ​


Now it’s time to apply the changes by deploying your service again.

Save the file and execute the following command in the `server` directory in a terminal:

$../hackernews-react-apollo/server

```bash
graphcool deploy
```

Your GraphQL API now includes three additional operations, as specified in `graphcool.yml`:

- `signup`: Create a new user based on `email` and `password`.

- `authenticate`: Log in existing user with `email` and `password`.

- `loggedInUser`: Checks whether a user is currently logged in.

  ​

### Adding an additional Argument to the `signup` Mutation

You can see the GraphQL interface for the newly added operations in the corresponding `.graphql`-files inside the `server/src/email-password` directory. Let’s take a look at the interface of the `signup` function:

```typescript
type SignupUserPayload {
  id: ID!
  token: String!
}

extend type Mutation {
  signupUser(email: String!, password: String!): SignupUserPayload
}

```

The `signupUser`-mutation is used to create a new `User` in the database. The problem right now is that our schema requires every `User` instance to have a `name`. However, the above `signupUser`-mutation only accepts `email` and `password` as arguments. You now need to adjust the `signup` resolver so it also accepts the `name` for the new `User` as an input argument and make sure it’s saved when the `User` is created.

Open `server/src/email-password/signup.graphql` and update the extension of the `Mutation` type to look as follows:

**../hackernews-react-apollo/server/src/email-password/signup.graphql

```typescript
extend type Mutation {
  signupUser(email: String!, password: String!, name: String!): SignupUserPayload
}

```

For now you only adjusted the *interface* of the `signup` resolver. Next, you also need to make sure to update the *implementation*. 

> 
>
> Note: The `signup` resolver is implemented as a [serverless function](https://www.graph.cool/docs/reference/functions/overview-aiw4aimie9) which will be deployed for you by the Graphcool Framework. The input arguments for that function are determined by the input arguments of the corresponding GraphQL operation. In this case, this is the `signupUser`-mutation, so the function will received three string as input arguments: `email`, `password` and `name`. (Notice that these are wrapped in a single object called `event` though.)
>
> 

The goal in the new implementation is to retrieve the `name` argument from the input `event` and send it along when creating the new `User`.

Open `signup.ts` and update the definition of the `EventData` interface like so:

**../hackernews-react-apollo/server/src/email-password/signup.ts

```typescript
interface EventData {
  email: string
  password: string
  name: string
}

```

Still in `signup.ts`, adjust the implementation of the anonymous (and topmost) function to look as follows:

**../hackernews-react-apollo/server/src/email-password/signup.ts

```typescript
export default async (event: FunctionEvent) => {
  console.log(event)

  try {
    const graphcool = fromEvent(event)
    const api = graphcool.api('simple/v1')

    const { email, password, name } = event.data

    if (!validator.isEmail(email)) {
      return { error: 'Not a valid email' }
    }

    // check if user exists already
    const userExists: boolean = await getUser(api, email)
      .then(r => r.User !== null)
    if (userExists) {
      return { error: 'Email already in use' }
    }

    // create password hash
    const salt = bcrypt.genSaltSync(SALT_ROUNDS)
    const hash = await bcrypt.hash(password, SALT_ROUNDS)

    // create new user
    const userId = await createGraphcoolUser(api, email, hash, name)

    // generate node token for new User node
    const token = await graphcool.generateNodeToken(userId, 'User')

    return { data: { id: userId, token } }
  } catch (e) {
    console.log(e)
    return { error: 'An unexpected error occured during signup.' }
  }
}

```

All you do is also retrieve the `name` from the input `event` and then pass it to the `createGraphcoolUser` function a bit later.

Still in `signup.ts`, update the `createGraphcoolUser` function like so:

**../hackernews-react-apollo/server/src/email-password/signup.ts

```typescript
async function createGraphcoolUser(api: GraphQLClient, email: string, password: string, name: string): Promise {
  const mutation = `
    mutation createGraphcoolUser($email: String!, $password: String!, $name: String!) {
      createUser(
        email: $email,
        password: $password,
        name: $name
      ) {
        id
      }
    }
  `

  const variables = {
    email,
    password,
    name
  }

  return api.request<{ createUser: User }>(mutation, variables)
    .then(r => r.createUser.id)
}

```

All that’s left for you now is deploying these changes to make sure your running Graphcool service gets updated and exposes the new functionality in its API.

In your terminal, navigate to the `server` directory and run:

$.../hackernews-react-apollo/server

```bash
graphcool deploy
```

Perfect, you’re all set now to actually implement the authentication functionality in the frontend as well.

### Implementing the Login Mutations

`signupUser` and `authenticateUser` are two regular GraphQL mutations that you can use in the same way as you did with the `createLink` mutation from before.

Open `Login.js` and add the following two definitions to the bottom of the file, also replacing the current `export Login` statement:

**.../hackernews-react-apollo/src/components/Login.js

```jsx
const SIGNUP_USER_MUTATION = gql`
  mutation SignupUserMutation($email: String!, $password: String!, $name: String!) {
    signupUser(
      email: $email,
      password: $password,
      name: $name
    ) {
      id
      token
    }
  }
`

const AUTHENTICATE_USER_MUTATION = gql`
  mutation AuthenticateUserMutation($email: String!, $password: String!) {
    authenticateUser(
      email: $email,
      password: $password
    ) {
      token
      user {
        id
      }
    }
  }
`

export default compose(
  graphql(SIGNUP_USER_MUTATION, { name: 'signupUserMutation' }),
  graphql(AUTHENTICATE_USER_MUTATION, { name: 'authenticateUserMutation' })
)(Login)

```



>  **Smerth - BUG => it seems the above AUTHENTICATE_USER_MUTATION code is wrong and should be….**
>
>  ```jsx
>  const AUTHENTICATE_USER_MUTATION = gql`
>   mutation AuthenticateUserMutation($email: String!, $password: String!) {
>     authenticateUser(
>       email: $email,
>       password: $password
>     ) {
>       token
>       id
>     }
>   }
>  `
>  ```
>
>  





Note that you’re using `compose` for the export statement this time since there is more than one mutation that you want to wrap the component with.

Before we take a closer look at the two mutations, go ahead and add the required imports. 

Still in `Login.js`, add the following statement to the top of the file:

**.../hackernews-react-apollo/src/components/Login.js

```jsx
import { graphql, compose } from 'react-apollo'
import gql from 'graphql-tag'
```

Now, let’s understand what’s going in the two mutations that you just added to the component.

Both mutations look very similar to the mutations you already saw before. They take a number of arguments returns info the user’s `id` as well as a `token` that you can attach to subsequent requests to authenticate the user. You’ll learn in a bit how to do so.

All right, all that’s left to do is call the two mutations inside the code!

Open `Login.js` and implement `_confirm` as follows:

**.../hackernews-react-apollo/src/components/Login.js

```jsx
  _confirm = async () => {
    const { name, email, password } = this.state
    if (this.state.login) {
      const result = await this.props.authenticateUserMutation({
        variables: {
          email,
          password
        }
      })
      const { id, token } = result.data.authenticateUser
      this._saveUserData(id, token)
    } else {
      const result = await this.props.signupUserMutation({
        variables: {
          name,
          email,
          password
        }
      })
      const { id, token } = result.data.signupUser
      this._saveUserData(id, token)
    }
    this.props.history.push(`/`)
  }

```

The code is pretty straightforward. If the user wants to only login, you’re calling the `authenticateUserMutation`and pass the provided `email` and `password` as arguments. Otherwise you’re using the `signupUserMutation`where you additionally pass the user’s `name`. After the mutation was performed, you’re storing the returned `id`and `token` in `localStorage` and navigate back to the root route.

You can now create an account by providing a `name`, `email` and `password`. Once you did that, the *submit*-button will be rendered again:

![img](https://imgur.com/WoWLmDJ.png)

### Updating the `createLink`-mutation

Since you’re now able to authenticate users and also added a new relation between the `Link` and `User` type, you can also make sure that every new link that gets created in the app can store information about the user that posted it. That’s what the `postedBy` field on `Link` will be used for.

Open `CreateLink.js` and update the definition of `CREATE_LINK_MUTATION` as follows:

**.../hackernews-react-apollo/src/components/CreateLink.js

```jsx
const CREATE_LINK_MUTATION = gql`
  mutation CreateLinkMutation($description: String!, $url: String!, $postedById: ID!) {
    createLink(
      description: $description,
      url: $url,
      postedById: $postedById
    ) {
      id
      createdAt
      url
      description
      postedBy {
        id
        name
      }
    }
  }
`

```

There are two major changes. You first added another argument to the mutation that represents the `id` of the user that is posting the link. Secondly, you also include the `postedBy` information in the *selection set* of the mutation.

Now you need to make sure that the `id` of the posting user is included when you’re calling the mutation in `_createLink`.

Still in `CreateLink.js`, update the implementation of `_createLink` like so:

**.../hackernews-react-apollo/src/components/CreateLink.js

```jsx
_createLink = async () => {
  const postedById = localStorage.getItem(GC_USER_ID)
  if (!postedById) {
    console.error('No user logged in')
    return
  }
  const { description, url } = this.state
  await this.props.createLinkMutation({
    variables: {
      description,
      url,
      postedById
    }
  })
  this.props.history.push(`/`)
}

```

For this to work, you also need to import the `GC_USER_ID` key. 

Add the following import statement to the top of `CreateLink.js`.

**.../hackernews-react-apollo/src/components/CreateLink.js

```jsx
import { GC_USER_ID } from '../constants'
```

Perfect! Before sending the mutation, you’re now also retrieving the corresponding user id from `localStorage`. If that succeeds, you’ll pass it to the call to `createLinkMutation` so that every new `Link` will from now on store information about the `User` who created it.

If you haven’t done so before, go ahead and test the login functionality. Run `yarn start` and open `http://localhost:3000/login`. Then click the *need to create an account?*-button and provide some user data for the user you’re creating. Finally, click the *create account*-button. If all went well, the app navigates back to the root route and your user was created. You can verify that the new user is there by checking the *data browser* in the [Graphcool Console](https://console.graph.cool/) or sending the `allUsers` query in a Playground.

### Configuring Apollo with the Authentication Token

Now that users are able to login and obtain a token that authenticates them against the Graphcool backend, you actually need to make sure that the token gets attached to all requests that are sent to your service’s API.

Since all the API requests are actually created and sent by the `ApolloClient` instance in your app, you need to make sure it knows about the user’s token. Luckily, Apollo provides a nice way for authenticating all requests by using the concept of [middleware](http://dev.apollodata.com/react/auth.html#Header), implemented as an [Apollo Link](https://github.com/apollographql/apollo-link).

Open `index.js` and put the following code *between* the creation of the `httpLink` and the instantiation of the `ApolloClient`:

**.../hackernews-react-apollo/src/index.js

```jsx
const middlewareAuthLink = new ApolloLink((operation, forward) => {
  const token = localStorage.getItem(GC_AUTH_TOKEN)
  const authorizationHeader = token ? `Bearer ${token}` : null
  operation.setContext({
    headers: {
      authorization: authorizationHeader
    }
  })
  return forward(operation)
})

const httpLinkWithAuthToken = middlewareAuthLink.concat(httpLink)

```

This middleware will be invoked every time `ApolloClient` sends a request to the server. You can imagine the process of sending a request as a *chain* of functions that are called. Each function gets passed the GraphQL `operation` and another function called `forward` which needs to be called when the middleware is finished with its task to passed the `operation` to the next function in the chain. 

Now you also need to make sure `ApolloClient` gets instantiated with the correct link - update the constructor call as follows:

**.../hackernews-react-apollo/src/index.js

```jsx
const client = new ApolloClient({
  link: httpLinkWithAuthToken,
  cache: new InMemoryCache()
})

```

Then directly import the key you need to retrieve the token from `localStorage` as well as `ApolloLink` on top of the same file:

**.../hackernews-react-apollo/src/index.js

```jsx
import { GC_AUTH_TOKEN } from './constants'
import { ApolloLink } from 'apollo-client-preset'
```

That’s it - now all your API requests will be authenticated if a `token` is available.

> **Note**: In a real application you would now configure the [permissions rules](https://www.graph.cool/docs/reference/auth/authorization/overview-iegoo0heez/) of your project to define what kind of operations authenticated and non-authenticated users should be allowed to perform. See [this](https://github.com/graphcool/framework/tree/master/examples/permissions) Graphcool service definition for a practical example.





# More Mutations and Updating the Store

The next piece of functionality that you’ll implement is the voting feature! Authenticated users are allowed to submit a vote for a link. The most upvoted links will later be displayed on a separate route!

### Preparing the React Components

Once more, the first step to implement this new feature is to make your React components ready for the expected functionality.

Open `Link.js` and update `render` to look as follows:

**.../hackernews-react-apollo/src/components/Link.js

```jsx
render() {
  const userId = localStorage.getItem(GC_USER_ID)
  return (
    <div className='flex mt2 items-start'>
      <div className='flex items-center'>
        <span className='gray'>{this.props.index + 1}.</span>
        {userId && <div className='ml1 gray f11' onClick={() => this._voteForLink()}>▲</div>}
      </div>
      <div className='ml1'>
        <div>{this.props.link.description} ({this.props.link.url})</div>
        <div className='f6 lh-copy gray'>{this.props.link.votes.length} votes | by {this.props.link.postedBy ? this.props.link.postedBy.name : 'Unknown'} {timeDifferenceForDate(this.props.link.createdAt)}</div>
      </div>
    </div>
  )
}

```

You’re already preparing the `Link` component to render the number of votes for each link and the name of the user that posted it. Plus you’ll render the upvote button if a user is currently logged in - that’s what you’re using the `userId` for. If the `Link` is not associated with a `User`, the user’s name will be rendered as `Unknown`.

Notice that you’re also using a function called `timeDifferenceForDate` that gets passed the `createdAt`information for each link. The function will take the timestamp and convert it to a string that’s more user friendly, e.g. `"3 hours ago"`.

Go ahead and implement the `timeDifferenceForDate` function next so you can import and use it in the `Link`component. 

Create a new file called `utils.js` in the `src` directory and paste the following code into it:

**.../hackernews-react-apollo/src/utils.js

```jsx
function timeDifference(current, previous) {

  const milliSecondsPerMinute = 60 * 1000
  const milliSecondsPerHour = milliSecondsPerMinute * 60
  const milliSecondsPerDay = milliSecondsPerHour * 24
  const milliSecondsPerMonth = milliSecondsPerDay * 30
  const milliSecondsPerYear = milliSecondsPerDay * 365

  const elapsed = current - previous

  if (elapsed < milliSecondsPerMinute / 3) {
    return 'just now'
  }

  if (elapsed < milliSecondsPerMinute) {
    return 'less than 1 min ago'
  }

  else if (elapsed < milliSecondsPerHour) {
    return Math.round(elapsed/milliSecondsPerMinute) + ' min ago'
  }

  else if (elapsed < milliSecondsPerDay ) {
    return Math.round(elapsed/milliSecondsPerHour ) + ' h ago'
  }

  else if (elapsed < milliSecondsPerMonth) {
    return Math.round(elapsed/milliSecondsPerDay) + ' days ago'
  }

  else if (elapsed < milliSecondsPerYear) {
    return Math.round(elapsed/milliSecondsPerMonth) + ' mo ago'
  }

  else {
    return Math.round(elapsed/milliSecondsPerYear ) + ' years ago'
  }
}

export function timeDifferenceForDate(date) {
  const now = new Date().getTime()
  const updated = new Date(date).getTime()
  return timeDifference(now, updated)
}

```

Back in `Link.js`, import `GC_USER_ID` and `timeDifferenceForDate` on top the file:

**.../hackernews-react-apollo/src/components/Link.js

```jsx
import { GC_USER_ID } from '../constants'
import { timeDifferenceForDate } from '../utils'
```

Finally, each `Link` element will also render its position inside the list, so you have to pass down an `index` from the `LinkList` component. 

Open `LinkList.js` and update the rendering of the `Link` components inside `render` to also include the link’s position:

**.../hackernews-react-apollo/src/components/LinkList.js

```jsx
{linksToRender.map((link, index) => (
  <Link key={link.id} index={index} link={link}/>
))}
```

Notice that the app won’t run at the moment since the `votes` are not yet included in the query. You’ll fix that next!

### Updating the Schema

For this new feature, you also need to update the schema again since votes on links will be represented with a custom `Vote` type.

Open `types.graphql` and add the following type definition:

**.../hackernews-react-apollo/server/types.graphql

```typescript
type Vote @model {
  id: ID! @isUnique
  user: User! @relation(name: "UsersVotes")
  link: Link! @relation(name: "VotesOnLink")
}
```

Each `Vote` will be associated with the `User` who created it as well as the `Link` that it belongs to. You also have to add the other end of the relation. 

Still in `types.graphql`, add the following field to the `User` type:

**.../hackernews-react-apollo/server/types.graphql

```typescript
votes: [Vote!]! @relation(name: "UsersVotes")
```

Now add another field to the `Link` type:

**.../hackernews-react-apollo/types.graphql

```typescript
votes: [Vote!]! @relation(name: "VotesOnLink")
```

Next open up a terminal window and navigate to the directory where `types.graphql` is located. Then apply your changes by typing the following command:

$.../hackernews-react-apollo

```bash
graphcool deploy
```

Awesome! Now that you updated the data model, you can fix the issue that currently prevents you from properly running the app. It can be fixed by including the information about the links’ votes in the `allLinks` query that’s defined in `LinkList`.

Open `LinkList.js` and update the definition of `ALL_LINKS_QUERY` to look as follows:

**.../hackernews-react-apollo/src/components/LinkList.js

```jsx
const ALL_LINKS_QUERY = gql`
  query AllLinksQuery {
    allLinks {
      id
      createdAt
      url
      description
      postedBy {
        id
        name
      }
      votes {
        id
        user {
          id
        }
      }
    }
  }
`

```

All you do here is to also include information about the user who posted a link as well as information about the links’ votes in the query’s payload. You can now run the app again and the links will be properly displayed. 

![img](https://i.imgur.com/eHaPg3L.png)

Let’s now move on and implement the upvote mutation!

### Calling the Mutation

Open `Link.js` and add the following mutation definition to the bottom of the file. Once more, also replacing the current `export Link` statement:

**.../hackernews-react-apollo/src/components/Link.js

```jsx
const CREATE_VOTE_MUTATION = gql`
  mutation CreateVoteMutation($userId: ID!, $linkId: ID!) {
    createVote(userId: $userId, linkId: $linkId) {
      id
      link {
        votes {
          id
          user {
            id
          }
        }
      }
      user {
        id
      }
    }
  }
`

export default graphql(CREATE_VOTE_MUTATION, {
  name: 'createVoteMutation'
})(Link)

```

This step should feel pretty familiar by now. You’re adding the ability to call the `createVoteMutation` to the `Link`component by wrapping it with the `CREATE_VOTE_MUTATION`.

As with the times before, you also need to import the `gql` and `graphql` functions on top of the `Link.js` file:

**.../hackernews-react-apollo/src/components/Link.js

```jsx
import { graphql } from 'react-apollo'
import gql from 'graphql-tag'
```

Finally, you need to implement `_voteForLink` as follows:

**.../hackernews-react-apollo/src/components/Link.js

```jsx
_voteForLink = async () => {
  const userId = localStorage.getItem(GC_USER_ID)
  const voterIds = this.props.link.votes.map(vote => vote.user.id)
  if (voterIds.includes(userId)) {
    console.log(`User (${userId}) already voted for this link.`)
    return
  }

  const linkId = this.props.link.id
  await this.props.createVoteMutation({
    variables: {
      userId,
      linkId
    }
  })
}

```

Notice that in the first part of the method, you’re checking whether the current user already voted for that link. If that’s the case, you return early from the method and don’t actually execute the mutation.

You can now go and test the implementation! Run `yarn start` and click the upvote button on a link. You’re not getting any UI feedback yet, but after refreshing the page you’ll see the added votes. 

There still is a flaw in the app though. Since the `votes` on a `Link` don’t get updated right away, a `User` currently can submit an indefinite number of votes until the page is refreshed. Only then the protection mechanism will apply and instead of an upvote, the click on the voting button will simply result in the following logging statement in the console: *User (cj42qfzwnugfo01955uasit8l) already voted for this link.*

But at least you know that the mutation is working. In the next section, you’ll fix the issue and make sure that the cache gets updated directly after each mutation!

### Updating the Cache

One cool thing about Apollo is that you can manually control the contents of the cache. This is really handy, especially after a mutation was performed, since this allows you to determine precisely how you want the cache to be updated. Here, you’ll use it to make sure the UI displays the correct number of votes right after the `createVote` mutation was performed.

You can implement this functionality by using Apollo’s [imperative store API](https://dev-blog.apollodata.com/apollo-clients-new-imperative-store-api-6cb69318a1e3).

Open `Link.js` and update the call to `createVoteMutation` inside the `_voteForLink` method as follows:

**.../hackernews-react-apollo/src/components/Link.js

```jsx
const linkId = this.props.link.id
await this.props.createVoteMutation({
  variables: {
    userId,
    linkId
  },
  update: (store, { data: { createVote } }) => {
    this.props.updateStoreAfterVote(store, createVote, linkId)
  }
})

```

The `update` function that you’re adding as an argument to the mutation call will be called when the server returns the response. It receives the payload of the mutation (`data`) and the current cache (`store`) as arguments. You can then use this input to determine a new state of the cache. 

Notice that you’re already *destructuring* the server response and retrieving the `createVote` field from it. 

All right, so now you know what this `update` function is, but the actual implementation will be done in the parent component of `Link`, which is `LinkList`. 

Open `LinkList.js` and add the following method inside the scope of the `LinkList` component:

**.../hackernews-react-apollo/src/components/LinkList.js

```jsx
_updateCacheAfterVote = (store, createVote, linkId) => {
  // 1
  const data = store.readQuery({ query: ALL_LINKS_QUERY })
  
  // 2
  const votedLink = data.allLinks.find(link => link.id === linkId)
  votedLink.votes = createVote.link.votes
  
  // 3
  store.writeQuery({ query: ALL_LINKS_QUERY, data })
}

```

What’s going on here?

1. You start by reading the current state of the cached data for the `ALL_LINKS_QUERY` from the `store`.

2. Now you’re retrieving the link that the user just voted for from that list. You’re also manipulating that link by resetting its `votes` to the `votes` that were just returned by the server.

3. Finally, you take the modified data and write it back into the store.

4. Next you need to pass this function down to the `Link` so it can be called from there. 



Still in `LinkList.js`, update the way how the `Link` components are rendered in `render`:

**.../hackernews-react-apollo/src/components/LinkList.js

```jsx
<Link key={link.id} updateStoreAfterVote={this._updateCacheAfterVote} index={index} link={link}/>
```

That’s it! The `update` function will now be executed and make sure that the store gets updated properly after a mutation was performed. The store update will trigger a rerender of the component and thus update the UI with the correct information!



While we’re at it, let’s also implement `update` for adding new links!

Open `CreateLink.js` and update the call to `createLinkMutation` inside `_createLink` like so:

**.../hackernews-react-apollo/src/components/CreateLink.js

```jsx
await this.props.createLinkMutation({
  variables: {
    description,
    url,
    postedById
  },
  update: (store, { data: { createLink } }) => {
    const data = store.readQuery({ query: ALL_LINKS_QUERY })
    data.allLinks.splice(0,0,createLink)
    store.writeQuery({
      query: ALL_LINKS_QUERY,
      data
    })
  }
})

```

The `update` function works in a very similar way as before. You first read the current state of the results of the `ALL_LINKS_QUERY`. Then you insert the newest link to the top and write the query results back to the store.

The last thing you need to do for this to work is import the `ALL_LINKS_QUERY` into that file:

**.../hackernews-react-apollo/src/components/CreateLink.js

```jsx
import { ALL_LINKS_QUERY } from './LinkList'
```

Conversely, it also needs to be exported from where it is defined. 

Open `LinkList.js` and adjust the definition of the `ALL_LINKS_QUERY` by adding the `export` keyword to it:

**.../hackernews-react-apollo/src/components/LinkList.js

```jsx
export const ALL_LINKS_QUERY = ...
```

Awesome, now the store also updates with the right information after new links are added. The app is getting into shape. 🤓





# Filtering: Searching the List of Links

In this section, you’ll implement a search feature and learn about the filtering capabilities of your GraphQL API.

> 
>
> Note: If you’re interested in all the filtering capabilities of Graphcool, you can check out the [documentation](https://www.graph.cool/docs/reference/simple-api/filtering-by-field-xookaexai0/) on it.
>
> 

### Preparing the React Components

The search will be available under a new route and implemented in a new React component.

Start by creating a new file called `Search.js` in `src/components` and add the following code:

**.../hackernews-react-apollo/src/components/Search.js

```jsx
import React, { Component } from 'react'
import { withApollo } from 'react-apollo'
import gql from 'graphql-tag'
import Link from './Link'

class Search extends Component {

  state = {
    links: [],
    searchText: ''
  }

  render() {
    return (
      <div>
        <div>
          Search
          <input
            type='text'
            onChange={(e) => this.setState({ searchText: e.target.value })}
          />
          <button
            onClick={() => this._executeSearch()}
          >
            OK
          </button>
        </div>
        {this.state.links.map((link, index) => <Link key={link.id} link={link} index={index}/>)}
      </div>
    )
  }

  _executeSearch = async () => {
    // ... you'll implement this in a bit
  }

}

export default withApollo(Search)

```

Again, this is a pretty standard setup. You’re rendering an `input` field where the user can type a search string. 

Notice that the `links` field in the component state will hold all the links to be rendered, so this time we’re not accessing query results through the component props! We’ll also talk about the `withApollo` function that you’re using when exporting the component in a bit!

Now add the `Search` component as a new route to the app. Open `App.js` and update render to look as follows:

**.../hackernews-react-apollo/src/components/App.js

```jsx
render() {
  return (
    <div className='center w85'>
      <Header />
      <div className='ph3 pv1 background-gray'>
        <Switch>
          <Route exact path='/' component={LinkList}/>
          <Route exact path='/create' component={CreateLink}/>
          <Route exact path='/login' component={Login}/>
          <Route exact path='/search' component={Search}/>
        </Switch>
      </div>
    </div>
  )
}

```

Also import the `Search` component at the top of the file:

**.../hackernews-react-apollo/src/components/App.js

```jsx
import Search from './Search'
```

For the user to be able to comfortably navigate to the search functionality, you should also add a new navigation item to the `Header`.

Open `Header.js` and put a new `Link` between `new` and `submit`:

**.../hackernews-react-apollo/src/components/Header.js

```jsx
<div className='flex flex-fixed black'>
  <div className='fw7 mr1'>Hacker News</div>
  <Link to='/' className='ml1 no-underline black'>new</Link>
  <div className='ml1'>|</div>
  <Link to='/search' className='ml1 no-underline black'>search</Link>
  {userId &&
  <div className='flex'>
    <div className='ml1'>|</div>
    <Link to='/create' className='ml1 no-underline black'>submit</Link>
  </div>
  }
</div>

```

You can now navigate to the search functionality using the new button in the `Header`:

![img](https://imgur.com/XxPdUvo.png)

Great, let’s now go back to the `Search` component and see how we can implement the actual search.

### Filtering Links

Open `Search.js` and add the following query definition at the bottom of the file:

**.../hackernews-react-apollo/src/components/Search.js

```jsx
const ALL_LINKS_SEARCH_QUERY = gql`
  query AllLinksSearchQuery($searchText: String!) {
    allLinks(filter: {
      OR: [{
        url_contains: $searchText
      }, {
        description_contains: $searchText
      }]
    }) {
      id
      url
      description
      createdAt
      postedBy {
        id
        name
      }
      votes {
        id
        user {
          id
        }
      }
    }
  }
`

```

This query looks similar to the `allLinks` query that’s used in `LinkList`. However, this time it takes in an argument called `searchText` and specifies a `filter` object that will be used to specify conditions on the links that you want to retrieve.

In this case, you’re specifying two filters that account for the following two conditions: A link is only returned if either its `url` contains the provided `searchText` *or* its `description` contains the provided `searchText`. Both conditions can be combined using Graphcool’s `OR` operator.

Perfect, the query is defined! But this time we actually want to load the data every time the user hits the *search*-button. 

That’s the purpose of the [`withApollo`](http://dev.apollodata.com/react/higher-order-components.html#withApollo) function. This function injects the `ApolloClient` instance that you created in `index.js` into the `Search` component as a new prop called `client`.

This `client` has a method called `query` which you can use to send a query manually instead of using the `graphql` higher-order component.

Here’s what the code looks like. Open `Search.js` and implement `_executeSearch` as follows:

**.../hackernews-react-apollo/src/components/Search.js

```jsx
_executeSearch = async () => {
  const { searchText } = this.state
  const result = await this.props.client.query({
    query: ALL_LINKS_SEARCH_QUERY,
    variables: { searchText }
  })
  const links = result.data.allLinks
  this.setState({ links })
}

```

The implementation is almost trivial. You’re executing the `ALL_LINKS_SEARCH_QUERY` manually and retrieving the `links` from the response that’s returned by the server. Then these links are put into the component’s `state` so that they can be rendered.

Go ahead and test the app by running `yarn start` in a terminal and navigating to `http://localhost:3000/search`. Then type a search string into the text field, click the *search*-button and verify the links that are returned fit the filter conditions.





# Realtime Updates with GraphQL Subscriptions

This chapter is a mess on the tutorial website.

The text doesn't match the video and there seems to be major changes to how Apollo is to be implemented.  

To see how I muddled through this chapter and made is work consult the following:

1. The chapter video and text on the tutorial website
2. My code (my-subscriptions-code.md in docs)



> **SMERTH - subscriptions were working before the pagination chapter but now they are not ☹️**



# Pagination

The last topic that we’ll cover in this tutorial is pagination. You’ll implement a simple pagination approach so that users are able to view the links in smaller chunks rather than having an extremely long list of `Link`elements.

## Preparing the React Components

Once more, you first need to prepare the React components for this new functionality. In fact, we’ll slightly adjust the current routing setup. Here’s the idea: The `LinkList` component will be used for two different use cases (and routes). The first one is to display the 10 top voted links. Its second use case is to display new links in a list separated into multiple pages that the user can navigate through.

Open `App.js` and adjust the render method like so:

**.../hackernews-react-apollo/src/components/App.js

```jsx
render() {
    return (
      <div className='center w85'>
        <Header />
        <div className='ph3 pv1 background-gray'>
          <Switch>
            <Route exact path='/' render={() => <Redirect to='/new/1' />} />
            <Route exact path='/login' component={Login} />
            <Route exact path='/create' component={CreateLink} />
            <Route exact path='/search' component={Search} />
            <Route exact path='/top' component={LinkList} />
            <Route exact path='/new/:page' component={LinkList} />
          </Switch>
        </div>
      </div>
    )
  }

```

Make sure to import the Redirect component, so you don’t get any errors.

Also update the router import on the top of the file:

**.../hackernews-react-apollo/src/components/App.js

```jsx
import { Switch, Route, Redirect } from 'react-router-dom'
```

You now added two new routes: `/top` and `/new/:page`. The latter reads the value for `:page` from the url so that this information is available inside the component that’s rendered, here that’s `LinkList`.

The root route `/` now redirects to the first page of the route where new posts are displayed.

Before moving on, quickly add a new navigation item to the `Header` component that brings the user to the `/top` route.

Open `Header.js` add the following lines *between* the `/` and the `/search` routes:

**.../hackernews-react-apollo/src/components/Header.js

```jsx
<Link to='/top' className='ml1 no-underline black'>top</Link>
<div className='ml1'>|</div>
```

You also need to add quite some logic to the `LinkList` component to account for the two different responsibilities it now has. 

Open `LinkList.js` and add three arguments to the `AllLinksQuery` by replacing the `ALL_LINKS_QUERY`definition with the following:

**.../hackernews-react-apollo/src/components/LinkList.js

```jsx
export const ALL_LINKS_QUERY = gql`
  query AllLinksQuery($first: Int, $skip: Int, $orderBy: LinkOrderBy) {
    allLinks(first: $first, skip: $skip, orderBy: $orderBy) {
      id
      createdAt
      url
      description
      postedBy {
        id
        name
      }
      votes {
        id
        user {
          id
        }
      }
    }
    _allLinksMeta {
      count
    }
  }
`

```

The query now accepts arguments that we’ll use to implement pagination and ordering. `skip` defines the *offset* where the query will start. If you passed a value of e.g. `10` for this argument, it means that the first 10 items of the list will not be included in the response. `first` then defines the *limit*, or *how many* elements, you want to load from that list. Say, you’re passing the `10` for `skip` and `5` for `first`, you’ll receive items 10 to 15 from the list. 

But how can we pass the variables when using the `graphql` container which is fetching the data under the hood? You need to provide the arguments right where you’re wrapping your component with the query.

Still in `LinkList.js`, replace the current `export` statement with the following:

**.../hackernews-react-apollo/src/components/LinkList.js

```jsx
export default graphql(ALL_LINKS_QUERY, {
  name: 'allLinksQuery',
  options: (ownProps) => {
    const page = parseInt(ownProps.match.params.page, 10)
    const isNewPage = ownProps.location.pathname.includes('new')
    const skip = isNewPage ? (page - 1) * LINKS_PER_PAGE : 0
    const first = isNewPage ? LINKS_PER_PAGE : 100
    const orderBy = isNewPage ? 'createdAt_DESC' : null
    return {
      variables: { first, skip, orderBy }
    }
  }
})(LinkList)

```

You’re now passing a function to `graphql` that takes in the props of the component (`ownProps`) before the query is executed. This allows you to retrieve the information about the current page from the router (`ownProps.match.params.page`) and use it to calculate the chunk of links that you retrieve with `first` and `skip`.

Also note that you’re including the ordering attribute `createdAt_DESC` for the `new` page to make sure the newest links are displayed first. The ordering for the `/top` route will be calculated manually based on the number of votes for each link.

You also need to define the `LINKS_PER_PAGE` constant and then import it into the `LinkList` component.

Open `src/constants.js` and add the following definition:

**.../hackernews-react-apollo/src/constants.js

```jsx
export const LINKS_PER_PAGE = 5
```

Now adjust the import statement from `../constants` in `LinkList.js` to also include the new constant: 

**.../hackernews-react-apollo/src/components/LinkList.js

```jsx
import { LINKS_PER_PAGE } from '../constants'
```

### Implementing Navigation

Next, you need functionality for the user to switch between the pages. First add two `button` elements to the bottom of the `LinkList` component that can be used to navigate back and forth.

Open `LinkList.js` and update `render` to look as follows:

**.../hackernews-react-apollo/src/components/LinkList.js

```jsx
render() {

  if (this.props.allLinksQuery && this.props.allLinksQuery.loading) {
    return <div>Loading</div>
  }

  if (this.props.allLinksQuery && this.props.allLinksQuery.error) {
    return <div>Error</div>
  }

  const isNewPage = this.props.location.pathname.includes('new')
  const linksToRender = this._getLinksToRender(isNewPage)
  const page = parseInt(this.props.match.params.page, 10)

  return (
    <div>
      <div>
        {linksToRender.map((link, index) => (
            <Link key={link.id} index={page ? (page - 1) * LINKS_PER_PAGE + index : index} updateStoreAfterVote={this._updateCacheAfterVote} link={link}/>
        ))}
      </div>
      {isNewPage &&
      <div className='flex ml4 mv3 gray'>
        <div className='pointer mr2' onClick={() => this._previousPage()}>Previous</div>
        <div className='pointer' onClick={() => this._nextPage()}>Next</div>
      </div>
      }
    </div>
  )

}

```

Since the setup is slightly more complicated now, you are going to calculate the list of links to be rendered in a separate method.

Still in `LinkList.js`, add the following method implementation:

**.../hackernews-react-apollo/src/components/LinkList.js

```jsx
_getLinksToRender = (isNewPage) => {
  if (isNewPage) {
    return this.props.allLinksQuery.allLinks
  }
  const rankedLinks = this.props.allLinksQuery.allLinks.slice()
  rankedLinks.sort((l1, l2) => l2.votes.length - l1.votes.length)
  return rankedLinks
}

```

For the `newPage`, you’ll simply return all the links returned by the query. That’s logical since here you don’t have to make any manual modifications to the list that is to be rendered. If the user loaded the component from the `/top` route, you’ll sort the list according to the number of votes and return the top 10 links.

Next, you’ll implement the functionality for the *Previous*- and *Next*-buttons.

In `LinkList.js`, add the following two methods that will be called when the buttons are pressed:

**.../hackernews-react-apollo/src/components/LinkList.js

```jsx
_nextPage = () => {
  const page = parseInt(this.props.match.params.page, 10)
  if (page <= this.props.allLinksQuery._allLinksMeta.count / LINKS_PER_PAGE) {
    const nextPage = page + 1
    this.props.history.push(`/new/${nextPage}`)
  }
}

_previousPage = () => {
  const page = parseInt(this.props.match.params.page, 10)
  if (page > 1) {
    const previousPage = page - 1
    this.props.history.push(`/new/${previousPage}`)
  }
}

```

The implementation of these is very simple. You’re retrieving the current page from the url and implement a sanity check to make sure that it makes sense to paginate back or forth. Then you simply calculate the next page and tell the router where to navigate next. The router will then reload the component with a new `page`in the url that will be used to calculate the right chunk of links to load. Run the app by typing `yarn start` in a Terminal and use the new buttons to paginate through your list of links!

### Final Adjustments

Through the changes that we made to the `ALL_LINKS_QUERY`, you’ll notice that the `update` functions of your mutations don’t work any more. That’s because `readQuery` now also expects to get passed the same variables that we defined before.

> 
>
> **Note**: `readQuery` essentially works in the same way as the `query` method on the `ApolloClient` that you used to implement the search. However, instead of making a call to the server, it will simply resolve the query against the local store! If a query was fetched from the server with variables, `readQuery` also needs to know the variables to make sure it can deliver the right information from the cache.
>
> 

With that information, open `LinkList.js` and update the `_updateCacheAfterVote` method to look as follows:

**.../hackernews-react-apollo/src/components/LinkList.js

```jsx
_updateCacheAfterVote = (store, createVote, linkId) => {
  const isNewPage = this.props.location.pathname.includes('new')
  const page = parseInt(this.props.match.params.page, 10)
  const skip = isNewPage ? (page - 1) * LINKS_PER_PAGE : 0
  const first = isNewPage ? LINKS_PER_PAGE : 100
  const orderBy = isNewPage ? 'createdAt_DESC' : null
  const data = store.readQuery({ query: ALL_LINKS_QUERY, variables: { first, skip, orderBy } })

  const votedLink = data.allLinks.find(link => link.id === linkId)
  votedLink.votes = createVote.link.votes
  store.writeQuery({ query: ALL_LINKS_QUERY, data })
}

```

All that's happening here is the computation of the variables depending on whether the user currently is on the `/top` or `/new` route. 

Finally, you also need to adjust the implementation of `update` when new links are created. 

Open `CreateLink.js` and replace the current contents of `_createLink` like so:

**.../hackernews-react-apollo/src/components/CreateLink.js

```jsx
_createLink = async () => {
  const postedById = localStorage.getItem(GC_USER_ID)
  if (!postedById) {
    console.error('No user logged in')
    return
  }
  const { description, url } = this.state
  await this.props.createLinkMutation({
    variables: {
      description,
      url,
      postedById
    },
    update: (store, { data: { createLink } }) => {
      const first = LINKS_PER_PAGE
      const skip = 0
      const orderBy = 'createdAt_DESC'
      const data = store.readQuery({
        query: ALL_LINKS_QUERY,
        variables: { first, skip, orderBy }
      })
      data.allLinks.splice(0,0,createLink)
      data.allLinks.pop()
      store.writeQuery({
        query: ALL_LINKS_QUERY,
        data,
        variables: { first, skip, orderBy }
      })
    }
  })
  this.props.history.push(`/new/1`)
}

```

Since you don’t have the `LINKS_PER_PAGE` constant available in this component yet, make sure to import it on top of the file:

**.../hackernews-react-apollo/src/components/CreateLink.js

```jsx
import { GC_USER_ID, LINKS_PER_PAGE } from '../constants'
```

You have now added a simple pagination system to the app, allowing users to load links in small chunks instead of loading them all up front.





> **SMERTH - subscriptions were working before the pagination chapter but now they are not ☹️**

