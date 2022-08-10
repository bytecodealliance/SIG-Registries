# Log Based Registry

# Goals

* Reduce required trust in registry operator
* Make malicious activity detectable
* Enable package state replication (mirroring)

# Design

## Package Log

For each package, a registry maintains an append-only package log.
This log is the canonical record of the package state and contains cryptographic information that makes it possible to verify its integrity.
Information can be derived from the log and used by some consumers, but other consumers and auditors will want to directly read and verify the log itself.

## Log Entries

A package log is a sequence of typed *entries*. Each entry contains the following fields.

| Field | Value Type | Description |
| ----- | ---------- | ----------- |
| prev | hash or `null` | the previous entry in the log |
| version | always "0.1.0" | the registry log protocol version |
| time | ISO 8601 timestamp | when the entry was created |
| author | public key | the key of the entry author |
| kind | [Entry Type](#Entry-Types) | this entry's type |
| contents | object | entry type-specific fields |

e.g.

```json
{
    "prev": <hash>,
    "version": "0.1.0",
    "time": "2022-05-11T11:08:00Z",
    "author": <public key>,
    "type": "release",
    "contents": {
        "version": "1.14.0",
        "digest": <hash>
    }
}
```

## Log Structure

Each *entry* is submitted as the payload of a JWS signed with the author's private key.
The registry operator then signs the JWS to indicate acceptance of the entry.

<p align="center"><img src="https://i.imgur.com/HYcgqMy.png" width="500px"></p>

## Transparency Logs

Registries should with some frequency publish the payload digest of the log head and their signature to a transparency log (e.g. [Rekor](https://github.com/sigstore/rekor)).

## Entry Types

*Note: none of the "Entry Types" have yet been assigned final names/mnemonics for use in the `kind` field.*

With the exception of the Initialize Log entry, which may be issued by any key, entries are only valid if the key which signed them is granted permission for that entry type by the log.

### Initialize Package (`init-package`)

Creates a new package with a specific name and begins the associated log.

* **Fields**
  * `name` - string
* **Validation**
  * Must always be the first entry in the log.
  * Requires that no package with that name exists in this registry.
* **Effects**
  * Registry guarantees exclusive ownership of the package name.
  * Permits the author to perform Assign Role.

**Note:** Registries may choose to reject `Initialize Log` entries that do not conform to some registry policy
(e.g. it could enforce ACME-based domain package namespaces).

### Update Authorization (`update-auth`)

Assigns a role to a key.

* **Fields**
  * `key` - public key
  * `allow` - list of entry type strings
  * `deny` - list of entry type strings
* **Validation**
  * The key was previously authorized for all types in the deny list
* **Effects**
  * The key is now able to create entries with the specified allow types
  * The key is no longer able to create entries with the specified deny types

### Release (`release`)

Makes a new version of the package available.

* **Fields**
  * `version` - a semantic version string
  * `digest` - content hash
* **Validation**
  * Requires that no other Release has had the same version.
* **Effects**
  * Creates a new package release.
  * The package version has state `"released"`

### Yank (`yank`)

An assertion by the maintainer that "this release is not fit for use". Does not alter the release, but may cause the release to be excluded from default query results.

* **Fields**
  * `version` - a semantic version string
* **Validation**
  * The package version had state`"released"`
* **Effects**
  * The package version has state `"yanked"`

# Open Questions

## Hashing Scheme

Define the way that payloads/envelopes/etc. are hashed and canonical digests are obtained. Define any mechanisms for identifying and migrating hash algorithms.

Define a clear recovery plan for the scenario where a hash is discovered to be weak enough to enable [pre-image attacks](https://en.wikipedia.org/wiki/Preimage_attack).

## Hash Indirection

There may be places where some entry content is large enough that directly including it significantly affects the size of the overall JWS.
In these cases, it may make sense to instead include only a hash of the content and provide some way of retrieving the content.

## Arbitrary Metadata

Maintainers may want to attach arbitrary metadata to released versions. This would be supported by an entry type (tentatively called annotate) that allows arbitrary contents ignored by the registry protocol.

Maintainers may want to grant permission to other entities to attach metadata. For example, a validator that reproduces their component build.

### Use Cases

1. Security Scanning - the log grants access to a security scanning tool which appends CVE information to the log.
2. Association - the log grants access to a software publisher who can choose whether or not to publicly “own” the project.
3. Certification - the log grants access to a certification body (i.e. FIPS) who can evaluate the software and grant certifications.
4. Reproducibility - the log grants access to a reproducible build tool who can use the instructions in the log to rebuild the software packages and testify on the log that the resulting builds had the same hashes.

### Options

* Allow annotation entries in the log for the latest and other versions (in-log annotations)
* Allow each release to begin a branch containing only annotations and potentially delegations (in-branch annotations)
* Add a role that can be granted for making annotations only (role-based annotation)
* Add an entry type that grants one-time permission to annotate a version (delegation)

## Operator Entries

Annotation, and potentially other future entries, may need to be performed by operators. This can be represented by a role `"operator"`. Registries will publicly identify valid operator keys and timestamps will be used to identify whether a key is/was valid when it signed an entry.

## Re-Assigning Package Names

Registry operators may want to re-assign a package name to new owners if a package is not updated for a long length of time. This must be considered a reset of trust in the package name and automatically upgrading across these boundaries must be prevented. This boundary may be used to reset the effective log length since it may obsolete prior package state.

## Registry Health

As new contents are added to the registry over time, the proportion of contents that contains some vulnerability will tend to increase. It is worth considering if anything can be done to ameliorate this.

### Options

* Enable registries to associate a time-to-live with each action on a package or release, so that inactive packages are removed automatically.
* Enable registries to determine some "health" score for each package or release, so that users are alerted to and can address the health issues of their dependencies

## Protocol Versioning

To support future iterations of this protocol, this proposal contains a versioning field. In the future, this can be used to introduce a new version and allow logs to monotonically increase in version to adopt new features. Another approach would be to encode the version into the entry type names in a standard way. This has the advantage that it is harder for implementations to accidentally ignore the version information there.
