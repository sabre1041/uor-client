# Universal Runtime (UR) Client

Note: The UR Client is being actively developed. Please consider joining the UOR Community to participate!

## Participate

Please join us in the discussion forum and feel free to ask questions about the UOR-Framework or UR Client.

## About

The Universal Runtime Client interacts with UOR artifacts and is aware of the runtime instruction
embedded in UOR artifacts.

To learn more about Universal Runtime visit the UOR Framework website at <https://universalreference.io>.

> WARNING: The repository is under active development and the API is subject to change.

## Development

### Requirements

- `go` version 1.17+

### Build

```
make
```

### Test

```
make test-unit
```

### Run

```
./bin/uor-client-go help
```

## Basic Usage

### Version

```
uor-client-go version
```

### User Workflow

1. Use `uor-client-go build schema` to build a schema to be used with a collection.
2. Use the `uor-client-go build collection` command to build the workspace as an OCI artifact in build-cache The default location is ~/.uor/cache. It can be set with the `UOR_CACHE` environment variable`.
3. Use the `uor-client-go push` command to publish to a registry as an OCI artifact.
4. Use the `uor-client-go pull` command to pull the artifact back to a local workspace.
5. Use the `uor-client-go inspect` command to inspect the build cache to list information about references.

### Build a schema into an artifact

A schema can be optionally created prior to building a collection. Collections can then reference an already built schema or no schema at all.

```shell
uor-client-go build schema schema-config.yaml localhost:5000/myschema:latest
```

### Build workspace into an artifact

Execute the following command to build a workspace into an an artifact:

```shell
uor-client-go build collection my-workspace localhost:5000/myartifacts:latest
```

An optional dataset configuration file can be provided using the `--dsconfig` flag:

```shell
uor-client-go build my-workspace localhost:5000/myartifacts:latest --dsconfig dataset-config.yaml
```

### Push workspace to a registry location

Push a workspace to a remote registry

```shell
uor-client-go push my-workspace localhost:5000/myartifacts:latest
```

### Pull UOR collection to a location

Pull a collection from a remote registry:

```shell
uor-client-go pull localhost:5000/myartifacts:latest -o my-output-directory
```

### Pull subsets of a UOR collection to a location by attribute

Pull a portion of a collection by filtering for a set of attribute:

```shell
uor-client-go pull localhost:5000/myartifacts:latest -o my-output-directory --attributes key=value
```

## Getting Started

This guide will walk through a basic workflow of using the UOR Client

Set an environment variable called `UOR_CLIENT_GO_REPO` with the current working directory within the repository. It will be used when working through the scenarios below.

```shell
export UOR_CLIENT_GO_REPO=$(pwd)
```

Create a directory called `examples` and change into this directory to begin working on the exercises.

```shell
mkdir ${UOR_CLIENT_GO_REPO}/exercises
cd ${UOR_CLIENT_GO_REPO}/exercises
```

### Basic Collection Publishing

The first exercise creates and publishes a basic collection to a remote repository

1. Create a new directory for this exercise called `basic` and change into this directory

```shell
mkdir -p ${UOR_CLIENT_GO_REPO}/exercises/basic
cd ${UOR_CLIENT_GO_REPO}/exercises/basic
```

2. Create a new _workspace_ directory for the collection called `basic-collection` and change into this directory

```shell
mkdir basic-collection
cd basic-collection
```

3. Add the content to the workspace. For example, two images of fishes an a text file.

Add the first photo to the current directory

```shell
cp ${UOR_CLIENT_GO_REPO}/test/fish.jpg .
```

Next, create a directory called `subdir1` to contain several additional files:

```shell
mkdir subdir1
```

Now, add a text file and another photo:

```shell
cp ${UOR_CLIENT_GO_REPO}/test/level1/fish2.jpg subdir1/
cp ${UOR_CLIENT_GO_REPO}/test/level1/file.txt subdir1/
```

4. Create a json document where the value of each kv pair is the path to each file within the directory. Multiple json documents can be used to create deep graphs, but a graph must only have one root. Multiple json docs in a build directory is for advanced use cases which will not be covered in this basic example and most just need one json document.

Create a json document called `basic.json` in the current directory with the following content:

```json
{
    "fish": "fish.jpg",
    "text": "subdir1/file.txt",
    "fish2": "subdir1/fish2.jpg"
}
```

5. A Dataset Configuration can be use to assign _attributes_ to the various resources within a collection. This file must be located outside of the content directory and refer to the relative paths within the content directory. Add user defined key value pairs as subkeys to the `annotations`section. Each file should have as many attributes as possible. Multiple files can be referenced by using the `*` wildcard.

Navigate up one directory and create a file called `dataset-config.yaml` with the following content:

```shell
cd ..
```

```yaml
kind: DataSetConfiguration
apiVersion: client.uor-framework.io/v1alpha1
collection:
  files:
    - file: "fish.jpg"
      attributes:
        animal: "fish"
        habitat: "ocean"
        size: "small"
        color: "blue"
    - file: "subdir1/file.txt"
      attributes:
        fiction: true  
        genre: "science fiction"
    - file: "*.jpg"
      attributes:
        custom: "customval"
```

6. Run the UOR client _build_ command referencing the dataset config, the content directory, and the destination registry location to the local cache. Each of the examples in this document will make use of a registry located at `localhost:5000`. Be sure to modify as appropriate for your environment including making use of the parameters related to insecure registries or communicating over HTTP.

```shell
uor-client-go build collection basic-collection localhost:5000/exercises/basic:latest --dsconfig dataset-config.yaml 
```

7. Run the UOR _push_ command to publish the collection to the remote repository.

```
uor-client-go push localhost:5000/exercises/basic:latest
```

8. Inspect the OCI manifest of the published collection. The `jq` tool can be used to format the response to make it more readable. Once again, if the remote registry is exposed using HTTP or non trusted certificates, adjust the curl command below accordingly:

```shell
curl -H "Accept: application/vnd.oci.image.manifest.v1+json" https://localhost:5000/v2/exercises/basic/manifests/latest
```

Notice that each of the files in the workspace are represented as _Layers_ within the Manifest. In addition, the relative location within the workspace along with the _attributes_ are added as _annotations_.

```json
...
  "layers": [
    {
      "mediaType": "text/plain; charset=utf-8",
      "digest": "sha256:8b8843c2c23a94efafa834c7b52547aa2cba63ed517c7891eba5b7386330482b",
      "size": 4,
      "annotations": {
        "org.opencontainers.image.title": "subdir1/file.txt",
        "uor.attributes": "{\"fiction\":true,\"genre\":\"science fiction\"}"
      }
...
```

9. The UOR _inspect_ subcommand can be used to view the contents of the local cache. By default, the cache is located at `~/.uor/cache/`.

```shell
uor-client-go inspect
```

Notice the collection built previously is now present within the cache

```
Listing all references:  
localhost:5000/exercises/basic:latest
```

10. The collection can be pulled from the remote registry to verify the content. Use the UOR _pull_ subcommand to a directory called _my-output-directory_ using the `-o` flag:

```shell
uor-client-go pull localhost:5000/exercises/basic:latest -o my-output-directory
```

11. Instead of retrieving an entire collection, a subset can be retrieved by creating a `AttributeQuery` resource.

Create a file called `attribute-query.yaml` with the following content:

```yaml
kind: AttributeQuery
apiVersion: client.uor-framework.io/v1alpha1
attributes:
  fiction: true
```

Now use the UOR _pull_ subcommand along with the `--attributes` option placing the contents in a directory titled `my-filtered-output-directory`:

```shell
uor-client-go pull localhost:5000/exercises/basic:latest -o my-filtered-output-directory --attributes attribute-query.yaml
```

Since the `fiction=true` attribute was associated with only the _file.txt_ file it was the only resource retrieved from the collection.

```shell
tree my-filtered-output-directory

my-filtered-output-directory
└── subdir1
    └── file.txt

1 directory, 1 file
```

### Collection Publishing with Schema

A _Schema_ can be used to define the attributes associated with a collection along with linking multiple collections.

Be sure that the `UOR_CLIENT_GO_REPO` environment variable is defined as described at the beginning of the exercises along with the `examples` directory.

1. Create a new directory called `schema` within the _exercises_ directory and change into this directory

```shell
mkdir -p ${UOR_CLIENT_GO_REPO}/exercises/schema
cd ${UOR_CLIENT_GO_REPO}/exercises/schema
```

2. Create the Schema Configuration in a file called `schema-config.yaml` to define attribute keys and types for corresponding collections:

```yaml
kind: SchemaConfiguration
apiVersion: client.uor-framework.io/v1alpha1
schema:
  attributeTypes:
    "animal": string
    "size": string
    "color": string
    "habitat": string
    "mammal": boolean
```

3. Use the UOR _build_ subcommand to build and save the schema within the local cache:

```shell
uor-client-go build schema schema-config.yaml localhost:5000/exercises/myschema:latest
```

4. Push the schema to the remote registry:

```
uor-client-go push localhost:5000/exercises/myschema:latest
```

5. Create a new directory called `schema-collection` to contain a workspace to demonstrate how a schema can be used

```shell
mkdir schema-collection
cd schema-collection
```

6. Add a picture of a fish and a picture of a dog to a subdirectory called `subdir1`:

```shell
mkdir subdir1

cp ${UOR_CLIENT_GO_REPO}/test/fish.jpg .
cp ${UOR_CLIENT_GO_REPO}/cli/testdata/uor-template/dog.jpeg subdir1/dog.jpg
```

7. Create a json document describing the two resources created within the workspace in a file called `schema-collection.json`

```json
{
    "fish": "fish.jpg",
    "dog": "subdir1/dog.jpg",
}
```

8. Navigate up one directory so that the Dataset Configuration file can be created:

```shell
cd ..
```

Create the `dataset-config.yaml` for the collection with the following content. Notice that the schema previously published is referenced in the `schemaAddress` property:

```yaml
kind: DataSetConfiguration
apiVersion: client.uor-framework.io/v1alpha1
collection:
  schemaAddress: "localhost:5000/exercises/myschema:latest"
  files:
    - file: "fish.jpg"
      attributes:
        animal: "fish"
        habitat: "ocean"
        size: "small"
        color: "blue"
        mammal: "false"
    - file: "subdir1/dog.jpg"
      attributes:
        animal: "dog"
        habitat: "house"
        size: "medium"
        color: "brown"
        mammal: "true"
```

9. Use the UOR client _build_ subcommand referencing the dataset config, the content directory, and the destination registry location. The attributes specified will be validated against the schema provided.

```shell
uor-client-go build collection schema-collection localhost:5000/exercises/schemacollection:latest --dsconfig dataset-config.yaml 
```

A validation error occurred since the _mammal_ attribute in the Dataset Configuration specified a string value instead of a boolean as defined in the schema.

In order to be able to build the schema, modify the _mammal_ attribute of the `dataset-config.yaml` file by removing the surrounding quotes as shown below in the updated Dataset Configuration:

```yaml
kind: DataSetConfiguration
apiVersion: client.uor-framework.io/v1alpha1
collection:
  schemaAddress: "localhost:5000/exercises/myschema:latest"
  files:
    - file: "fish.jpg"
      attributes:
        animal: "fish"
        habitat: "ocean"
        size: "small"
        color: "blue"
        mammal: false
    - file: "subdir1/dog.jpg"
      attributes:
        animal: "dog"
        habitat: "house"
        size: "medium"
        color: "brown"
        mammal: true
```

With a valid Dataset Configuration now in place, the collection should build successfully:

```shell
uor-client-go build collection schema-collection localhost:5000/examples/schemacollection:latest --dsconfig dataset-config.yaml 
```

10. Use the UOR client _push_ subcommand to publish the collection to the remote repository

```shell
uor-client-go push localhost:5000/exercises/schemacollection:latest
```

11. Inspect the OCI manifest of the published dataset

```shell
curl -H "Accept: application/vnd.oci.image.manifest.v1+json" https://localhost:5000/v2/exercises/schemacollection/manifests/latest
```

Note that the schema is an annotation within the manifest:

```json
...
"annotations": {
  "uor.schema": "localhost:5000/myschema:latest"
}
...
```

With the collection pushed, you can also revisit some of the other ways of interacting with the collection including _inspecting_ the local cache or _pulling_ either the entire collection or a subset by defining an _AttributeQuery_. Refer back to the _basic_ exercise for examples of how these steps can be achieved.

### Collection Publishing with Links

Collections can also refer to other collection; known as _Linked Collections_. It is important to note that a Linked Collection **must** have an attached schema.

Be sure that the `UOR_CLIENT_GO_REPO` environment variable is defined as described at the beginning of the exercises along with the `examples` directory.

1. Create a new directory called `linked` within the _exercises_ directory and change into this directory

```shell
mkdir -p ${UOR_CLIENT_GO_REPO}/exercises/linked
cd ${UOR_CLIENT_GO_REPO}/exercises/linked
```

2. Create a `schema-config.yaml` file with the following contents to define the schema for the collection:

```yaml
kind: SchemaConfiguration
apiVersion: client.uor-framework.io/v1alpha1
schema:
  attributeTypes:
    "animal": string
    "size": string
    "color": string
    "habitat": string
    "type": string
```

3. Build and push the schema to the remote registry

```bash
uor-client-go build schema schema-config.yaml localhost:5000/exercises/linkedschema:latest
uor-client-go push localhost:5000/exercises/linkedschema:latest
```

4. To demonstrate Linked Collections, start creating a leaf collection by creating a workspace directory called `leaf-workspace`.

```bash
mkdir leaf-workspace
```

5. Create a simple file called `leaf.txt` containing a single word within the workspace:

```bash
echo "leaf" > leaf-workspace/leaf.txt
```

6. Create the Dataset Configuration within a file called `leaf-dataset-config.yaml` with the following content:

```yaml
kind: DataSetConfiguration
apiVersion: client.uor-framework.io/v1alpha1
collection:
  schemaAddress: localhost:5000/exercises/linkedschema:latest
  files:
    - file: "*.txt"
      attributes:
        animal: "fish"
        habitat: "ocean"
        size: "small"
        color: "blue"
        type: "leaf"
```

7. Build and push the leaf collection to the remote registry

```bash
uor-client-go build collection leaf-workspace localhost:5000/exercises/leaf:latest --dsconfig leaf-dataset-config.yaml
uor-client-go push localhost:5000/exercises/leaf:latest
```

8. Build a Root collection and link the previously built collection

Create a new directory for the root collection called `root-workspace`

```bash
mkdir root-workspace
```

9. Create a simple file called `root.txt` containing a single word within the workspace:

```bash
echo "root" > root-workspace/root.txt
```

10. Create the Dataset Configuration within a file called `root-dataset-config.yaml` with the following content:

```yaml
kind: DataSetConfiguration
apiVersion: client.uor-framework.io/v1alpha1
collection:
  linkedCollections:
  - localhost:5000/exercises/leaf:latest
  schemaAddress: localhost:5000/exercises/linkedschema:latest
  files:
    - file: "*.txt"
      attributes:
        animal: "cat"
        habitat: "house"
        size: "small"
        color: "orange"
        type: "root"
```

11. Build and push the root collection to the remote registry

```bash
uor-client-go build collection root-workspace localhost:5000/exercises/root:latest --dsconfig root-dataset-config.yaml
uor-client-go push localhost:5000/exercises/root:latest
```

12. Pull the collection into a directory called `linked-output`

```bash
uor-client-go pull localhost:5000/exercises/root:latest -o linked-output
```

13. Inspect the contents of the `linked-output` directory

```bash
ls linked-output

root.txt
```

14. Retrieve the contents of the root and leaf collection

Notice that only the contents of the root collection was retrieved in the prior step. To pull the content of both the root and any leaf collections, use the `--pull-all` flag of the UOR client `pull` subcommand into a directory called `all-linked-output`.

```bash
uor-client-go pull localhost:5000/exercises/root:latest --pull-all -o all-linked-output
```

15. Inspect the content of the `all-linked-output` directory

```bash
ls all-linked-output

leaf.txt root.txt
```

16. Pulling by attributes can also be specified when referencing Linked Collections. Create a file called `color-query.yaml` to content that has the attribute `color=orange`

```yaml
kind: AttributeQuery
apiVersion: client.uor-framework.io/v1alpha1
attributes:
  "color": "orange"
```

17. Pull the contents from the linked collection using the Attribute Query into a directory called `color-output`:

```bash
uor-client-go pull localhost:5000/root:latest --pull-all --attributes color-query.yaml -o color-output
```

18. Inspect the content of the `color-output` directory

```bash
ls color-output

root.txt
```

Notice how only the _root.txt_ file was retrieved as only this file contained the attribute `color=orange`

# Glossary

`collection`: a collection of linked files represented as on OCI artifact
`schema`: the properties and datatypes that can be specified within a collection
