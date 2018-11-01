### What is it?
GraphQl provides a way to define complete description of data in schema and allows client to ask for what they need. It simply returns the response in JSON.

### Advantages compared to REST API
- Versioning: To avoid multiple versioning of your rest API.
- Ask for what you need: Client has a provision to ask only those fields which they needs. There would be no handling on server side specific to the platform.
- Get many API’s response in single request: Client has to call one query to get data from multiple rest API’s.

## Graph: notes & Edges
<img src="images/graphql-1.png?" width="600">
A graph is a data structure that includes notes (which are the rectangles)  and the edges (which are the relations). The way data is stored is not different; SQL or NoSQL can still be used. Once have put the data into graph, we query it through GraphQL. For example, let's imagine we want to start with user 23, find all their friends and all the companies those friend work at. This is how we would write the query:
```
query {
  // first find user with id 23.
  user(id: "23")
    users {
    // find all friends associated with id 23 {
      company {
      // find the company associated with all of those friends.
      }
    }
  }
}
```

## The GraphQL tool
It's a tool provided in the browser by the GraphQL Express library. We can add a query in the tool and see the result on the right. On the top-right there is a documentation explorer to understand the data structure and data fetching possibilities.