# JSON:API Revisions

Revisions is an extension to the JSON:API v1.1 specification, inspired by
Google's [Resource Revisions AIP][aip/162].

### Meta

#### Glossary

| Term     | Description                                                         |
| -------- | ------------------------------------------------------------------- |
| Revision | A complete representation of a Resource (attributes, relationships) |

#### URI

The canonical URL of this extension's Specification is
`https://github.com/scenarios-com/json-api-ext-revisions`

```http
GET /examples HTTP/1.1
Host: api.examples.com

Content-Type: application/vnd.api+json;
  ext="https://github.com/scenarios-com/json-api-ext-revisions"
```

#### Namespace

The namespace for this extension's members and query parameters is `revisions`.

## Requirements

The following list is a draft set of requirements for Revisioning.

A Revision...

- ...may be immutable
- ...must revise an earlier Revision, except the first Revision
- ...should be identified using a 8 character string of hexadecimal digits

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

### TODO

- [ ] Represent the mutability of a Revision in `revisions:revision`
- [ ] Define language to communicate the difference between a revision that has
      never been used and a revision that has been canonical at some point

[aip/162]: https://google.aip.dev/162
