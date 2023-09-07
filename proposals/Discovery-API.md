# Discovery API

> This API focuses on the querying and discovery of "components", but should be generalizable to other "entity types" (e.g. profiles & interfaces)

## Discovering Components

`GET /components`

### Parameters

`signatures`: comma-delimited set of signing identities (e.g. public-keys)

`package`: fully-qualified package id 

`imports`:  comma-delimited set of interface names

`exports`: comma-delimited set of interface names

`license`: license with which the entity was published

`versions`: semver query (default value = `latest`)
> `latest` means return only latest version of matched packages

`pre-release`: defaults to false; include semver pre-release versions of packages

> Omitting pagenation (e.g. `page` query param, `nextPage` result field) for brevity.
> Omitting full-text search, e.g. `search`.
> Query by profile could be added later once defined.

### Examples

* I want to see all components signed by set of signing identities
    `/components?signatures="1deadbeef...","2deadbeef"`
* I want to see all components with MIT license:
    `/components?license=MIT`
* I want to get the latest `package` with the ID `bytecodealliance:url-parser` that satisfies the semver query `^1.2`:
    `/components?package=bytecodealliance:url-parser&version=^1.2`
* I want to see all components that implement (i.e. exports) the `wasi:cache` interface: `/components?exports=wasi:cache`
* I want to see all components that depend on the `wasi:cache` interface: `/components?imports=wasi:cache`
* I want to see _all versions_ of the component ID `bytecodealliance:url-parser` component that satisfy the semver query `^1`:
  `/components?package=bytecodealliance:url-parser&versions=^1`

> TODO: Add more examples

#### Pagination Example
`GET /components?...&page=defcba`
```jsonc
{
    "items": [
        {
            "description": "something",
            "version": "0.1.0"
            // ...           
        }
    ],
    "nextPage": "/components?...&page=abcdef"
}
```