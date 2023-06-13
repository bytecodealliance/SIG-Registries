# Log Based Registry

# Goals

* Reduce required trust in registry operator
* Make malicious activity detectable
* Enable package state replication (mirroring)

# Package Log

For each package, a registry maintains an append-only package log.
This log is the canonical record of the package state and contains cryptographic information that makes it possible to verify its integrity.
Information can be derived from the log and used by some consumers, but other consumers and auditors will want to directly read and verify the log itself.

## Package Entry

A package log is a sequence of typed Package Entries which represent events in the life of a package.

```protobuf
message PackageEntry {
    // The previous entry in the log.
    // First entry of a log has no previous entry.
    optional string prev = 1;
    // The warg protocol version.
    // Encoded using semver 2.0.
    // Only legal value currently is "0.1.0".
    string version = 2;
    // The time when this entry was created
    // Format is ISO 8601 timestamp
    string time = 3;
    
    // The content specific to this entry type
    oneof content {
        Init init = 1;
        UpdateAuth update_auth = 2;
        Release release = 3;
        Yank yank = 4;
    }
}
```

### Initialize Package (`init`)

Creates a new package and begins the associated log.

```protobuf
message Init {
    // The hash algorithm used by this package to link entries.
    string algorithm = 1;
    // The key for the author of this entry.
    string key = 2;
}
```

* **Validation**
  * Must always be the first entry in the log.
  * The signature on this entry must be by the key specified in the `key` field.
* **Effects**
  * Permits the author to perform Assign Role.

**Note:** Registries may choose to reject `init` entries that do not conform to some registry policy
(e.g. it could enforce ACME-based domain package namespaces).

### Update Authorization (`update-auth`)

Assigns a role to a key.

```protobuf
message UpdateAuth {
    string key = 1;
    repeated Permission allow = 2;
    repeated Permission deny = 3;
}

enum Permission {
    UpdateAuth = 1;
    Release = 2;
    Yank = 3;
}
```

* **Validation**
  * This entry was signed by a key with the `update-auth` permission.
  * The key was previously authorized for all types in the deny list
* **Effects**
  * The key is now able to create entries with the specified allow types
  * The key is no longer able to create entries with the specified deny types

### Release (`release`)

Makes a new version of the package available.

```protobuf
message Release {
    // The semver 2.0 version of the release.
    string version = 1;
    // The labeled hash of the release contents.
    string digest = 2;
}
```

* **Validation**
  * This entry was signed by a key with the `release` permission.
  * Requires that no other Release has had the same version.
* **Effects**
  * Creates a new package release.
  * The package version has state `"released"`

### Yank (`yank`)

An assertion by the maintainer that "this release is not fit for use". Does not alter the release, but may cause the release to be excluded from default query results.

```protobuf
message Yank {
    // The semver 2.0 version of the release to yank.
    string version = 1;
}
```

* **Validation**
  * This entry was signed by a key with the `yank` permission.
  * The package version had state`"released"`
* **Effects**
  * The package version has state `"yanked"`

# Operator Log

Each registry also has exactly one operator log.
This log is the canonical record of the registry operator's state, which is currently just the set of keys that are valid over time.

## Operator Entry
A package log is a sequence of typed Operator Entries which represent events in the life of the registry.

```protobuf
message OperatorEntry {
    // The previous entry in the log.
    // First entry of a log has no previous entry.
    optional string prev = 1;
    // The warg protocol version.
    // Encoded using semver 2.0.
    // Only legal value currently is "0.1.0".
    string version = 2;
    // The time when this entry was created
    // Format is ISO 8601 timestamp
    string time = 3;
    
    // The content specific to this entry type
    oneof content {
        Init init = 1;
        UpdateAuth update_auth = 2;
    }
}
```

### Initialize (`init`)

```protobuf
message Init {
    // The hash algorithm used by this package to link entries.
    string algorithm = 1;
    // The key for the author of this entry.
    string key = 2;
}
```

### Update Authorization (`update-auth`)

```protobuf
message UpdateAuth {
    string key = 1;
    repeated Permission allow = 2;
    repeated Permission deny = 3;
}

enum Permission {
    UpdateAuth = 1;
    // When
    Commit = 2;
}
```

# Registry DAG

The Registry DAG is a log-backed map. For each key that is a package name `package:foobar`, the map always points to the head entry of the log at that point. The changes to this map are stored in the backing log, which as a result contains a total ordering of all log entries.

# Appendix

## Hashes
Hashes are represented as strings in the following way.

```bnf
<hash-algorithm> ::= "SHA256"

; A hash encoded with the name of the algorithm used to make it
<labeled-hash> ::= <hash-algorithm> ":" <hash-bytes-b64>

; Used in cases where the hash algorithm can be inferred
<unlabeled-hash> ::= <hash-bytes-b64>
```

## Signatures
Signatures and keys are represented as strings in the following way.

```bnf
<sig-algorithm> ::= "ECDSA"

<signature> ::= <sig-algorithm> ":" <signature-bytes-b64>

<key> ::= <sig-algorithm> ":" <key-bytes-b64>
```

