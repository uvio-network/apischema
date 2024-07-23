# apischema

This is the source of truth for the Uvio RPC API. The apischema defines how RPC
calls are shaped and function using Protocol Buffer definitions. Having a
unified abstraction helps us to generate client and server code across different
languages so that we remain mostly independent of transport layer specifics.



### Resource Definitions

* The `Post` resource describes claims and comments. Claims are structured in
  trees of lifecycle specific stages that every claim may go through. Every
  claim may have comments associated with them. Every claim and comment that
  exists must have reputation staked with it. That means that every opinion
  expressed and every vote cast on the Uvio platform has an inherent amount of
  skin in the game attached to it without exception.
* The `List` resource allows users to manage custom feeds based on a set of
  associated rules. Static lists can only contain manually added claims. Dynamic
  lists contain automatically added claims based on the aggregated criteria of a
  list's rule set. Criteria for dynamic lists can be for instance labels that
  describe market categories like politics, sports or economy.
* The `Rule` resource allows users to effectively add claims to custom lists
  based on certain criteria. Criteria like specific claim IDs are static. Adding
  static rules to static lists restricts the lists' ability to only contain
  static rules. Criteria like label IDs and user IDs are dynamic. Adding dynamic
  rules to dynamic lists restricts the lists' ability to only contain dynamic
  rules.



### Query Objects

All APIs allow for multiple action specific query objects, each of which defines
a certain behaviour. Create APIs allow for multiple create query objects. Search
APIs allow for multiple search query objects. Each object represents a context
defining its own behaviour. Some search query objects may yield a union of
results, so that a multitude of search criteria can be provided in order to
receive a response containing all merged results.



##### Object.Intern

`Object.Intern` search queries reference internally managed fields of the
underlying resource objects. The search query below returns the post object
associated to the specified post ID, which in turn originated from the system at
resource creation.

```json
{
    "object": [
        {
            "intern": {
                "post": "1234"
            },
            "public": {},
            "symbol": {}
        }
    ]
}
```



##### Object.Public

`Object.Public` search queries reference publicly provided fields of the
underlying resource objects. The search query below returns a list of claim
objects associated to the specified label IDs, which were added to the claim by
the user when proposing said claims.

```json
{
    "object": [
        {
            "intern": {},
            "public": {
                "label": "4567",
            },
            "symbol": {}
        }
    ]
}
```



##### Object.Symbol

`Object.Symbol` search queries execute arbitrary logic to return resource
objects matching certain use case requirements. The search query below resolves
to the list of claims that the provided list ID resolves to.

```json
{
    "object": [
        {
            "intern": {},
            "public": {},
            "symbol": {
                "list": "551265"
            }
        }
    ]
}
```



##### Response.Reason

`Response.Reason` provides the caller with a possible explanation for the
returned result. Every API may implement specific behaviour to manage the
resources it is responsible for. Such behaviour may imply authentication,
authorization or other preconditions that the caller must comply with. Since not
all API behaviour warrants an implicit error response that ought to expresses
failure, a list of reasons for the returned result may be presented in order to
inform about the logical decisions that an API had to make when serving the
callers request as presented. We may refer to these reasons as warnings. They
are errors, but not really. They imply an error condition occurred which is
expected to happen for some users of the system. Then, instead of failing
strictly with an error response, the returned result may simply be empty,
accompanied with an additional explanation of why that was then the case.

```json
{
    "object": [],
    "reason": [
        {
            "desc": "This is why the warning occurred.",
            "kind": "someSoftError"
        }
    ]
}
```



### JSON-Patch

Complex updates require JSON-Patch operations. If a resource for instance
defines a list of strings it would require the whole list to be provided only in
order to add or remove a single item of that resource's list property. For such
operations the resource's service in question provides the option of JSON-Patch
operations.

- https://datatracker.ietf.org/doc/html/rfc6902
- https://jsonpatch.com
- https://github.com/evanphx/json-patch



### Pagination

Pagination works by providing paging pointers for search query objects. The
pointers `start` and `stop` are always **inclusive**. In the request example
below the response would contain the first batch of 50 items. Depending on the
requested resources, consecutive calls with the exact same pointers may not
result in the same objects received, given the dynamic nature of the underlying
system.

```json
{
    "filter": {
        "paging": {
            "kind": "page",
            "start": "0",
            "stop": "49"
        }
    }
}
```

Providing paging pointers in requests containing multiple search query objects
causes the system to apply paging to each search query result. Consider
resources to be searched from two lists using two search query objects. Say the
lists look like [A B C D] and [E F G H]. Searching for both results from both
lists in a single call using `start=0` and `stop=1` would return [A B] and [E
F]. If paging is desired for large lists of objects, single search query objects
may be more applicable, depending on the use case.

There are two kinds of paging pointers, `kind=page` and `kind=unix`. Kind "page"
is the default and does not have to be provided. Using kind "page" may result in
less received objects than requested, but never more. Using kind "unix" may
result in indeterministic amounts of received objects. Searching for objects
constrained to a particular hour in time may be done by defining `kind=unix`,
`start=1672531200` and `stop=1672534800`.

Further note that the hard cap of returned objects is 1000. No more than 1000
objects can be received from any endpoint in a single call.



### Formatting

```bash
clang-format -i $(find ./pbf -name "*.proto")
```
