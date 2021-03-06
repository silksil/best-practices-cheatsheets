> Note: this configuration doesn't make use of Next.js

### Introduction
The purpose of GraphQl client is a bonding layer between React and the GraphQl graph. Some major GraphQl clients are Lokka, Apollo Client and Relay. </br>
<img src="../images/graphql-client.png?" width="600">

Apollo consists out of the Apollo store and provider. The store is a client-side piece of tech that directly communicate with the server and saves the data in the store. It can be used across different technologies, not just React. Nonetheless, the Apollo provider glues React with the Apollo store by taking data from the store in injecting it in the client-side. Most set-up is related to the Apollo provider. </br>
<img src="../images/apollo-store-provider.png?" width="400">

### Set-up
To set-up Apollo we have to install dependencies::
 ```
 - apollo-client
 - react-apollo
 ```

Next, we set-up the config of Apollo:
```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import ApolloClient from 'apollo-client';
import { ApolloProvider } from 'react-apollo';

// Create a new instance of ApolloClient and pass it to ApolloProvider
// We pass an empty config object => it then assumes the back-end is set-up on the route /graphql
const client = new ApolloClient({});

const Root = () => {
  return (
    <ApolloProvider client={client}>
      <div>Lyrical</div>
    </ApolloProvider>
  );
};

ReactDOM.render(
  <Root />,
  document.querySelector('#root')
);
```

### How to write GraphQl queries in React
<img src="../images/graphql-apollo-react.png?">

GraphQl queries are not valid javascript. In order to write GraphQl queries we need to import 'graphql-tag'. We can refer to it, but have to make sure we include backticks. Next, we have to bond the queries with a component (`export default graphql(query)(SongList)`). The data is eventually stored in the `props` object under the same key as defined in the query. This leads to the following code:
```jsx
import React, { Component } from 'react';
import gql from 'graphql-tag';
import { graphql } from 'react-apollo';

class SongList extends Component {
  render() {
    console.log(this.props);

    return (
      <div>
        SongList
      </div>
    );
  }
}

// the query
const query = gql`
  {
    songs {
      title
    }
  }
`;

// bond component with query
export default graphql(query)(SongList)
```
### Flow React Component & GraphQL
When we first render the component, the query we write will automatically send to our back-end server. This as async; when the data is being rendered the component will re-render with the data.
<img src="../images/flow-component-graphql.png?" width="600">

## Mutations
To include a mutation we can create a const that includes the query:
```jsx
const mutation = gql`
  mutation AddLyricToSong($content: String, $songId: ID) {
    addLyricToSong(content: $content, songId: $songId) {
      id
      lyrics {
        content
      }
    }
  }
`;
```
We then have to associate the mutation with the component:
```jsx
export default graphql(mutation)(LyricCreate);
```

To initialize the mutations we have to call `this.props.mutate`

### Mutations Query with Param
If you have to create mutations that includes a parameter, it is not possible to just refer to the params through `this.state` in the query, as it is not part of the component. Instead you can make use of query variables. It allows you to insert variables outside the query. A query like this would include the following:

<img src="../images/mutations-query.png?" width="600">

To include the parameter into the query, you have to you have to pass a variable to `this.props.mutate`:
```jsx
this.props.mutate({
  variables: {
    title: this.state.title
  }
});
```

### Apollo store: storing data:
The Apollo store, or client, has internal buckets of data. So in this case, it has an internal bucket with songs and lyrics. It will fetch data from the GraphQl server and store data in one of these buckets. It will know in which bucket the data has to be stored because every response of the server will add a field that defines the type (`__typename`) of data.

<img src="../images/apollo-data-storing.png?"

### Apollo store: refetch after mutation
The shortcoming is that Apollo has no idea what data or which properties exist in the bucket. As a result, if new data is rendered, Apollo cannot detect this, even though it is actually changed and included in the bucket.

##### Solution 1
As a solution, we can add a piece of configuration that will take all records that are being fetched and provide it with an id. By giving an id, Apollo is being able to communicate to React when any piece of data is updated, and when as result a component should be re-rendered. Note: this is based on the assumption that the DB returns and id. In order to do this we do the following in the root React file:

```jsx
const client = new ApolloClient({
  // go fetch every piece of data and use the id to identify every record
  dataIdFromObject: o => o.id
});
```
Below are two alternative solutions, non the less these solutions require you to conduct a second network request. In some cases this may be preferred, but in general the solution above is more desirable.

##### Solution 2
Let's say you added an item and directly afterwards go to a page that shows the items. It's possible that the store doesn't include the item yet, because it's not included in the Apollo store yet. In order to solve this, you can use `refetchQueries` and include the query you want to refetch:
```jsx
onSubmit(event) {
  event.preventDefault();

  this.props.mutate({
    variables: { title: this.state.title },
    /*
      We customize the mutate function.
      Trough refetchQueries we say to rerun the query the list of sings after the mutation is complete
    */
    refetchQueries: [{ query }]
  }).then(() => hashHistory.push('/'));
}
```
##### Solution 3
Alternatively, we can use a function that is assigned to `this.props.database` by the `react-apollo` library. It will automatically re-execute the queries that are associated with the component:
```jsx
onSongDelete(id) {
  this.props.mutate({ variables: { id } })
    .then(() => this.props.data.refetch());
}
```
`this.props.data.refetch()` only works for the data that is associated with that component! If you call refetch(), but you go to a different component, it's not assured that the data shows up in that component.

An another alternative is by assigning an id to every piece of data. This is discussed above in the section `Storing Data Apollo Store`.


### Queries in a seperate folder
To prevent duplicating code and keep organised it is convention to put the queries in a seperate folder (queries) and file. For example, we create a file `fetchSongs` in queries:
```jsx
import gql from 'graphql-tag';

export default gql`
  {
    songs {
      id
      title
    }
  }
`;
```
Then, we import it: `import query from '../queries/fetchSongs';`


### Extract param, query with param
When you pass info through a paramater, you can find it automatically as `props.paramater`.

In order to make sure that we create query that includes a parameter, we have to do some configuration. Note, that query are automatically executed for us, but mutations we have to be done manually. As a result, for queries there is not a clear location where to pass variables. Fortunately, this is covered by the Apollo team:
```jsx
export default graphql(fetchSong, {
  // this.props in React is the same as the props here. Passed to GraphQl through ReactRouter
  options: (props) =>
  { return { variables: { id: props.params.id } }
}
})(SongDetail);
```
