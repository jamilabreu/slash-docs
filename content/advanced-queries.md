+++
title = "Advanced Queries with GraphQL+-"
weight = 2
[menu.main]
    parent = "slash-graphql"
+++

*It is now possible to [embed GraphQL+- queries inside your GraphQL schema](https://dgraph.io/docs/graphql/custom/graphqlpm), which is recommended for most use cases. The rest of this document covers how to connect to connect to your Slash GraphQL backend with existing Dgraph clients.*

Slash GraphQL also supports running advanced queries with `GraphQL+-`, a query language that is unique to Dgraph. GraphQL+- should be used by advanced users who wish to make queries and mutations using existing Dgraph client libraries, either via the HTTP or gRPC endpoints. You can learn more about existing client libraries by following this [documentation](https://dgraph.io/docs/clients/).

If you are getting started with Slash GraphQL, you might want to consider using our [GraphQL APIs](https://dgraph.io/docs/graphql/overview) instead. It will get you quickly started on the basics of using Slash GraphQL before you go into advanced topics.

Please note that Slash GraphQL does not allow you to alter the schema or create new predicates via GraphQL+-. You will also not be able ta access the /alter endpoint or it's gRPC equivalent. Please add your schema through the GraphQL endpoint (either via the UI or via the Admin API), before accessing the data with GraphQL+-.

## Authentication

All the APIs documented here require an API token for access. Please see [Authentication](/admin/authentication) if you would like to create a new API token.

### HTTP

You can query your backend with GraphQL+- using the `/query` endpoint of your cluster. As an example, if your graphql endpoint is `https://frozen-mango-42.us-west-2.aws.cloud.dgraph.io/graphql`, then the admin endpoint for schema will be at `https://frozen-mango.us-west-2.aws.cloud.dgraph.io/query`.

This endpoint works identically to to the [/query](https://dgraph.io/docs/query-language/) endpoint of Dgraph, with the additional constraint of requiring authentication, as described in the Authentication section above.

You may also access the [`/mutate`](https://dgraph.io/docs/mutations/) and `/commit` endpoints.

For the given GraphQL Schema:
```graphql
type Person {
 name: String! @search(by: [fulltext])
 age: Int
 country: String
}
```

Here is an example of a cURL for `/mutate` endpoint:
```
curl -H "Content-Type: application/rdf" -H "x-auth-token: <api-key>" -X POST "<graphql-endpoint>/mutate?commitNow=true" -d $'
{
 set {
    _:x <Person.name> "John" .
    _:x <Person.age> "30" .
    _:x <Person.country> "US" .
 }
}'
```
Here is an example of a cURL for `/query` endpoint:
```
curl -H "Content-Type: application/graphql+-" -H "x-auth-token: <api-key>" -XPOST "<graphql-endpoint>/query" -d '{
   queryPerson(func: type(Person))  {
     Person.name
     Person.age
     Person.country
  }
}'
```

### gRPC

Slash GraphQL is compatible with most existing Dgraph clients. You can use the helper methods from each library to connect to your backend, passing in a Slash GraphQL endpoint and API token.

Here is an example which uses the [pydgraph client](https://github.com/dgraph-io/pydgraph) to make gRPC requests.

```python
import pydgraph

client_stub = pydgraph.DgraphClientStub.from_slash_endpoint("https://frozen-mango-42.eu-central-1.aws.cloud.dgraph.io/graphql", "<api-key>")
client = pydgraph.DgraphClient(client_stub)
```

For mutations, you can use the following example:
```python
mut = {
  "Person.name": "John Doe",
  "Person.age": "32",
  "Person.country": "US"
}

txn = client.txn()
try:
  res = txn.mutate(set_obj=mut)
finally:
  txn.discard()
```

And for a query you can use the following example:
```python
query = """
{
   queryPerson(func: type(Person))  {
     Person.name
     Person.age
     Person.country
  }
}"""
txn = client.txn()
try:
  res = txn.query(query)
  ppl = json.loads(res.json)
  print(ppl)
finally:
  txn.discard()
```

#### Connecting from Dgraph Clients

Below are snippets to connect to your Slash Backend from various dgraph clients.

**Python**
```python
import pydgraph

client_stub = pydgraph.DgraphClientStub.from_slash_endpoint("https://frozen-mango-42.eu-central-1.aws.cloud.dgraph.io/graphql", "<api-key>")
client = pydgraph.DgraphClient(client_stub)
```

**Javascript**
```javascript
const dgraph = require("dgraph-js");

const clientStub = dgraph.clientStubFromSlashGraphQLEndpoint(
  "https://frozen-mango-42.eu-central-1.aws.cloud.dgraph.io/graphql",
  "<api-key>"
);
const dgraphClient = new dgraph.DgraphClient(clientStub);
```

**Go**
```golang
// This example uses dgo
conn, err := dgo.DialSlashEndpoint("https://frozen-mango-42.eu-central-1.aws.cloud.dgraph.io/graphql", "<api-key>")
if err != nil {
  log.Fatal(err)
}
defer conn.Close()
dgraphClient := dgo.NewDgraphClient(api.NewDgraphClient(conn))
```

**Java**
```java
// This example uses dgraph4j
DgraphStub stub = DgraphClient.clientStubFromSlashEndpoint("https://frozen-mango-42.eu-central-1.aws.cloud.dgraph.io/graphql", "<api-key>");
DgraphClient dgraphClient = new DgraphClient(stub);
```

### Visualizing your Graph with Ratel

It is possible to use Ratel to visualize your Slash GraphQL backend with GraphQL+-. You may use self hosted Ratel, or using [Dgraph Play](https://play.dgraph.io/?latest#connection)

In order to configure Ratel, please do the following:

* Click the Dgraph logo in the top left to bring up the connection screen (by default, it has the caption: play.dgraph.io)
* Enter your backend's host in the Dgraph Server URL field. This is obtained by removing `/graphql` from the end of your graphql endpoint. As an example, if your graphql endpoint is `https://frozen-mango-42.us-west-2.aws.cloud.dgraph.io/graphql`, then the host for ratel will be at `https://frozen-mango.us-west-2.aws.cloud.dgraph.io`
* Click the blue 'Connect' button. You should see a green tick check next to the word connected
* Click on the 'Extra Settings' tab, and enter your API token into the 'Slash API Key" field. Please see [Authentication](/admin/authentication) if you would like to create a new API token.
* Click on the blue 'Continue' button

You may now queries and mutation via Ratel, and see visualizations of your data.

However, please note that certain functionality will not work, such as running Backups, modifying ACL or attempting to remove nodes from the cluster.

### Switching Backend Modes

For those who are interested in using DQL/GraphQL+- as their primary mode of interaction with the backend, it is possible to switch your backend to flexible mode. Please see [Backend Modes](/admin/backend-modes)
