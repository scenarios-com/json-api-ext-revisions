# JSON:API Revisions

The Revisions extension is an _opinionated_ extension to the JSON:API v1.1
specification, influenced by Google's
["Resource Revisions" API Improvement Proposal][aip/162].

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
- A representation of a revision of a Resource must remain consistent for
  the entire life of the Resource, regardless of API evolution
- An application may choose to allow Revisions to change before they become a
  part of the historical record to allow a user to _stage_ and then _commit_
  changes

[Questions][discussions], [suggestions][discussions], and [corrections][pulls]
to the specification are gladly received if they align with these principles.
Some of these principles may be burdensome for your use-case; please fork the
specification if neccessary.

## Meta

### Glossary

| Term     | Description                                                         |
| -------- | ------------------------------------------------------------------- |
| Revision | A complete representation of a Resource (attributes, relationships) |

### URI

The canonical URL of this extension's Specification is
`https://github.com/scenarios-com/json-api-ext-revisions`

```http
GET /examples HTTP/1.1
Host: api.examples.com

Content-Type: application/vnd.api+json;
  ext="https://github.com/scenarios-com/json-api-ext-revisions"
```

### Namespace

The namespace for this extension's members and query parameters is `revisions`.

## Requirements

A Revision...

- ...may be immutable
- ...must revise an earlier Revision, except the first Revision
- ...should be identified using an 8 character string of hexadecimal digits

A Revisioned Resource...

- ...has a `canonical` Revision which represents the current / latest state of
  the Resource
- ...must not have any state that independent of a Revision

A server...

- ...may permit a client to specify a Revision ID when creating a Revision
- ...must automatically create a Revision when creating a Resource if a client
  did not provide one
- ...may accept partial Revision information
- ...must include a Revision ID in a Revisioned Resource's
  `Resource Identifier Object`
- ...must treat the absence of a Revision ID as a reference to the canonical
  Revision
- ...must not persist a reference to a Revisioned Resource without a Revision ID
- ...may derive canonical state dynamically through multiple Revisions
- ...may store the complete state of each Revision
- ...should make Revisions immutable once a reference to it (implicit or
  explicit) exists
- ...must uniquely identify a Resource using `type` + `id` + `revisions:id`
- ...must treat Revisions as a supplement; a client must be able to interact
  with the server without knowledge of Revisions
- ...must include revision information in relation links
- ...must represent a Revision of a Resource the same, forever, regardless of
  how the api changes -- it _may_ allow a client to explicitly opt-out of this
  behaviour

A client...

- ...must request the revision of a Resource as described in
  `Resource Identifier Object`

### Example Resource Structure

```
{
  "type": "example",
  "id": "69fcb727-f791-4604-aa9f-1a079cd9a0d7",
  "revisions:id": "5a3960d1",
  "revisions:canonical": "953245a9",
  "revisions:history": [
    {"id": "953245a9"},
    {"id": "5a3960d1"},
  ],
  "revisions:revision": {
    "id": "5a3960d1",
    "summary": "Created example",
    "revises": null,
    "createdAt": "2020-01-01T00:00:00Z",
    "updatedAt": "2020-01-01T00:00:00Z",
  }
}
```

[aip/162]: https://google.aip.dev/162
[event-sourcing]: https://martinfowler.com/eaaDev/EventSourcing.html
[discussions]: /discussions
[pulls]: /pulls
