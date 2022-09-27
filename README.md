<!-- Logo Start -->
<!-- An HTML element is intentionally used since GitHub recommends this approach to handle different images in dark/light modes. Ref: https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#specifying-the-theme-an-image-is-shown-to -->
<!-- markdownlint-disable-next-line MD033 -->
<h1><picture><source media="(prefers-color-scheme: dark)" srcset="https://github.com/project-oak/oak/blob/main/docs/oak-logo/svgs/oak-transparent-release-negative-colour.svg?sanitize=true"><source media="(prefers-color-scheme: light)" srcset="https://github.com/project-oak/oak/blob/main/docs/oak-logo/svgs/oak-transparent-release.svg?sanitize=true"><img alt="Project Oak Logo" src="docs/oak-logo/svgs/oak-logo.svg?sanitize=true"></picture></h1>
<!-- Logo End -->

With this `hello-transparent-release` repo we showcase [Transparent Release](https://github.com/project-oak/transparent-release).

In this repo lives a [Java program](src/main/java/com/example/HelloTransparentRelease.java) printing `Hello Transparent Release` to `stdout`. 

We want to apply [Transparent Release](https://github.com/project-oak/transparent-release) on the binary of this Java program.

## [Optional] Build the binary from the command line first.

First, we need to make sure to have [Bazel set-up](https://docs.bazel.build/versions/main/tutorial/java.html#before-you-begin).

Then, we can build with

```bash
bazel build //:HelloTransparentRelease 
```

and run with

```bash
bazel run //:HelloTransparentRelease
[..]
INFO: Build completed successfully, 1 total action
Hello, Transparent Release!
```

or we run the binary:

```bash
./bazel-bin/HelloTransparentRelease
```

This is the binary we want to release: our `HelloTransparentRelease` binary.

Now, let's compute a sha256 digest of the `HelloTransparentRelease` binary:

```bash
sha256sum ./bazel-bin/HelloTransparentRelease
8a87337c16d1386510f9d3dd36a744d267945370e40c18113c78bb67e2934cae HelloTransparentRelease
```

This sha256 digest might or might not be the same on your machine.

Our goal is to make this to be the same for whoever builds the `HelloTransparentRelease` binary. 

That is why we build a builder Docker image.


## Build a builder Docker image to build the `HelloTransparentRelease` binary.

Our builder Docker image has everything installed to build the `HelloTransparentRelease` binary. 

We need to provide a [Dockerfile](Dockerfile) to build our builder Docker image. We name our Docker image `hello-transparent-release`. Our builder Docker image has to be publicly available, so we upload it to a registry: `europe-west2-docker.pkg.dev/oak-ci/hello-transparent-release/`.

We want the binary built by the builder Docker image to have the same permissions as the user. To do so, we rely on [rootless Docker](https://docs.docker.com/engine/security/rootless/).

To build our builder Docker image we run [`./scripts/docker_build`](./scripts/docker_build); to uplodad it to the registry we run [`./scripts/docker_push`] (given the right permissions).

We can now see the latest builder Docker image [here](https://pantheon.corp.google.com/artifacts/docker/oak-ci/europe-west2/hello-transparent-release?project=oak-ci). 

Uploading the Docker image will give us a manifest and a `DIGEST` to identify the image. For the _local_ builder Docker image we have no `DIGEST`:

```bash
docker images --digests gcr.io/oak-ci/hello-transparent-release:latest
REPOSITORY                                TAG       DIGEST    IMAGE ID       CREATED        SIZE
gcr.io/oak-ci/hello-transparent-release   latest    <none>    d682d6f0f2bb   20 hours ago   1.19GB
```

However, in the published registry [we find](https://pantheon.corp.google.com/artifacts/docker/oak-ci/europe-west2/hello-transparent-release/hello-transparent-release?project=oak-ci)

```bash
sha256:d682d6f0f2bbec373f4a541b55c03d43e52e774caa40c4b121be6e96b5d01f56
```


## Build the `HelloTransparentRelease` binary with the builder Docker image.

To build binary we use the [`cmd/builder` tool](https://github.com/project-oak/transparent-release#building-binaries-using-the-cmdbuilder-tool) from [project-oak/transparent-release](https://github.com/project-oak/transparent-release). 

We configure the  `cmd/builder` in [buildconfigs/hello_transparent_release.toml](buildconfigs/hello_transparent_release.toml).

From the checked out [transparent-release](https://github.com/project-oak/transparent-release) repo we call:

```bash
go run cmd/builder/main.go -build_config_path <path-to-hello-transparent-release-repo>/hello-transparent-release/buildconfigs/hello_transparent_release.toml 
```

The sha256 digest of the binary:

```bash
hello-transparent-release/out$ sha256sum HelloTransparentRelease
e8e05d1d09af8952919bf6ab38e0cc5a6414ee2b5e21f4765b12421c5db0037e  HelloTransparentRelease
```

This should now be the same on your machine! Please let us know if not!