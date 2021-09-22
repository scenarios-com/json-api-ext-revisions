# JSON:API Revisions ![draft]

The Revisions extension is an _opinionated_ extension to the JSON:API v1.1
specification, influenced by Google's
["Resource Revisions" API Improvement Proposal][aip/162].

:construction_worker: As an early draft, radical change is possible  
:crystal_ball: Please use milestones [v0] and [v1] to track progress towards
stability

## Design Rationale

Historical information about a Resource is often meaningful within the
application context; there are solutions to capturing this information with
varying degrees of strictness, from [Event Sourcing][event-sourcing] to activity
logs. Revisioning individual Resources is a helpful middle ground that provides
more flexibility than an activity log with lower costs than Event Sourcing.

The specification described here is an opinionated implementation
of revisioning; it may not suit all use-cases. A set of principles shaped the
design of this specification:

- Every Resource of a type within an application must have the same revisioning
  standards applied, but not every Resource type requires revisioning
- A representation of a Revision of a Resource must remain consistent for
  the entire life of the Resource, regardless of API evolution
- An application may choose to allow Revisions to change before they become a
  part of the historical record to allow a user to _stage_ and then _commit_
  changes

[Questions][discussions], [suggestions][discussions], and [corrections][pulls]
to the specification are gladly received if they align with these principles.
Some of these principles may be burdensome for your use case; please fork the
specification if necessary.

## Meta

### Glossary

Authors must use capitals for each defined term to provide clarity to readers.

| Term       | Description                                                         |
| ---------- | ------------------------------------------------------------------- |
| Resource   | A uniquely identifiable instance of a type                          |
| Revision   | A complete representation of a Resource (attributes, relationships) |
| Locked     | Immutable, no change nor is permitted                               |
| Canonical  | The current representation of a Resource                            |
| Collection | An array of zero or more Resources of a type                        |
| Fetch      | Make a request and read the response                                |
| JSON:API   | v1.1 of the JSON:API Specification                                  |

### URI

The URL of this extension's Specification is
`https://github.com/scenarios-com/json-api-ext-revisions`

```http
GET /examples HTTP/1.1
Host: api.examples.com

Content-Type: application/vnd.api+json;
  ext="https://github.com/scenarios-com/json-api-ext-revisions"
```

### Namespace

The namespace for this extension's members and query parameters is `revisions`.

## Client Use Cases

The specification must support a sane and pleasant path to implementing each of
these use cases for clients.

- [ ] Show the Canonical representation of a Resource without knowledge of its
      Revisions
- [ ] List a complete history of a Resource's Revisions
- [ ] Revert a Resource to the state of a previous Revision
- [ ] Persist changes without making them Canonical
- [ ] Create Revisions as "suggestions" that an authorized user may accept or
      reject
- [ ] Expose a user interface that _does not_ surface Revision information
      to end-users

### Client Operations

The following operations are required to support the use cases.

#### Standard Operations

All operations described in
[Creating, Updating and Deleting Resources][jsonapi/crud] must be possible
_without_ a client having any knowledge of the extension.

- [ ] Fetch Collection
- [ ] Fetch a Resource
- [ ] Create a Resource
- [ ] Update a Resource
- [ ] Update a Resource's Relationships
- [ ] Delete a Resource

#### Revision Operations

- [ ] Fetch a specific Revision of a Resource
- [ ] Create a new Resource _with_ genesis Revision information
- [ ] Update a specific Unlocked Revision of a Resource
- [ ] Promote an Unlocked Revision to the Canonical Revision
- [ ] Create a new Revision of a Resource that explicitly revises another
- [ ] Create a new Revision of a Resource _without_ explicitly revising another
- [ ] Delete an Unlocked Revision of a Resource

## Requirements

A Revision...

- ...may be Locked
- ...must revise an earlier Revision, except the first ("genesis") Revision
- ...should be identified using an 8 character string of hexadecimal digits

A Revisioned Resource...

- ...has a `canonical` Revision which represents the current / latest state of
  the Resource
- ...must not have any state that exists independent of a Revision

A server...

- ...may permit a client to specify a Revision ID when creating a Revision
- ...must automatically create a Revision when creating a Resource if a client
  did not provide one
- ...may accept partial Revision information
- ...must include a Revision ID in a Revisioned Resource's
  `Resource Identifier Object`
- ...must treat the absence of a Revision ID in a client request as a reference
  to the Canonical Revision
- ...must not persist a reference to a Revisioned Resource without a Revision ID
- ...may derive Canonical state dynamically through multiple Revisions
- ...may store the complete state of each Revision
- ...may accept changes to the Revision that is revised by another
- ...should make Revisions Locked once a reference to it (implicit or
  explicit) exists
- ...must uniquely identify a Resource using `type` + `id` + `revisions:id`
- ...must treat Revisions as a supplement; a client must be able to interact
  with the server without knowledge of Revisions
- ...must include Revision information in relation links
- ...must represent a Revision of a Resource the same, forever, regardless of
  how the api changes -- it _may_ allow a client to explicitly opt-out of this
  behavior
- ...should include a list of all Revisioned Resource types in the top-level meta
- ...may include a list of the subset of Revisioned Resource types that appear
  in the document
- ...may allow a client to "preview" the state of a Resource if a Revision were
  applied to it

A client...

- ...must Fetch the Revision of a Resource as described in its
  `Resource Identifier Object`
- ...should rely on the top-level meta to identify which Resources are
  Revisioned
- ...should not use the presence of namespaced attributes to identify which
  Resources are Revisioned

### Example Resource Structure

```json
{
  "type": "example",
  "id": "69fcb727-f791-4604-aa9f-1a079cd9a0d7",
  "revisions:id": "5a3960d1",
  "revisions:canonical": "953245a9",
  "revisions:history": [{ "id": "953245a9" }, { "id": "5a3960d1" }],
  "revisions:revision": {
    "id": "5a3960d1",
    "summary": "Created example",
    "revises": null,
    "createdAt": "2020-01-01T00:00:00Z",
    "updatedAt": "2020-01-01T00:00:00Z"
  }
}
```

### Example Revisions Top-Level Meta

```json
{
  "meta": {
    "revisions:resources": ["articles", "people", "photos"]
  },
  "data": {}
}
```

[aip/162]: https://google.aip.dev/162
[event-sourcing]: https://martinfowler.com/eaaDev/EventSourcing.html
[discussions]: https://github.com/scenarios-com/json-api-ext-revisions/discussions
[pulls]: https://github.com/scenarios-com/json-api-ext-revisions/pulls
[draft]: https://img.shields.io/badge/specification-draft-orange
[v0]: https://github.com/scenarios-com/json-api-ext-revisions/milestone/1
[v1]: https://github.com/scenarios-com/json-api-ext-revisions/milestone/2
[jsonapi/crud]: https://jsonapi.org/format/#crud
