# Bundle Builder for CDK

The `bundle` script will allow you to build CDK bundles out of bundle fragments
(found in the fragments subdirectory). Here's what building the default bundle
looks like:

```sh
./bundle -o ./bundles/cdk-flannel k8s/cdk cni/flannel
```

This will compose the fragments into a single bundle and version them according
to the given channel, which by default is `stable`. Additionally, a bundle
README.md will be generated by concatenating all the fragment README.md's in the
order given.

## Options

### Channel

You can select a channel using the `--channel` or `-c` option. Valid options are
`stable`, `unstable`, `edge`, and `local`.

#### Example
```sh
./bundle -o ./bundles/core-flannel -c edge k8s/core cni/flannel
```

> Note: for this to work, the fragment charms must be public for the given channel.

The `local` channel can be used to generate a bundle wherein all the charms
are referenced locally. The `--localpath` or `-l` option can be used to set the
path for the location of all the charms. *All the charms must be located in the
same path for the local channel to work*. By default, `--localpath` is set to
`/home/ubuntu/builds`.

### Output directory

You can set the location for the resulting `bundle.yaml` and `README.md` with the
`--outputdir` or `-o` flag.

#### Example

```sh
./bundle -o ~/foo_dir k8s/core cni/flannel
```

> Note: `bundle` will not overwrite existing bundle.yaml and README.md files. If
> you provide a location that contains these files, `bundle` will abort.



## Building all the bundles

This repo includes a makefile that will produce all known bundles.
The bundles will be placed in the `bundles` directory of the root of this
repo.

```sh
make
```

## Fragments

The `fragments` directory is laid out so:

```plaintext
fragments
├── cni
│   └── flannel
│       ├── bundle.yaml
│       └── README.md
├── k8s
│   ├── cdk
│   │   ├── bundle.yaml
│   │   └── README.md
│   └── core
│       ├── bundle.yaml
│       └── README.md
└── monitor
    └── elastic
        ├── bundle.yaml
        └── README.md
```

When the user adds the `k8s/cdk` fragment, they're grabbing the `bundle.yaml`
and `README.md` from `fragments/k8s/cdk`.

Each fragment is simply a bundle with two constraints:

- It needs to interoperate with a `k8s` fragment by setting up relations.
- It must not define versions for charms.

Here's a simple example from the `flannel` fragment:

```yaml
series: bionic
services:
  "flannel":
    charm: "cs:~containers/flannel"
relations:
  - - "flannel:etcd"
    - "etcd:db"
  - - "flannel:cni"
    - "kubernetes-master:cni"
  - - "flannel:cni"
    - "kubernetes-worker:cni"
```
