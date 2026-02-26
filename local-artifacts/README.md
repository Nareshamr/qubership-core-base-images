# Local Artifacts

This directory is used for storing and referencing local Maven artifacts during testing and development. Instead of
pulling dependencies from remote Maven repositories, you can place artifact files here to be used directly.

## Usage

1. Place artifacts in this directory (see required names below for profiler images).
2. Build the image with `QUBERSHIP_PROFILER_ARTIFACT_SOURCE=local` and `QUBERSHIP_PROFILER_VERSION=<version>` so the Dockerfile uses these files instead of downloading from Maven Central.

This approach allows testing with unpublished artifacts (e.g. a build from [qubership-profiler-agent](https://github.com/Netcracker/qubership-profiler-agent) or a fork) and speeds up development by avoiding remote downloads.

## Profiler images (java-21-prof, java-25-prof)

When `QUBERSHIP_PROFILER_ARTIFACT_SOURCE=local`, the Dockerfile expects:

- `qubership-profiler-installer-<VERSION>.zip`
- `qubership-profiler-diagtools-<VERSION>-<TARGETOS>-<TARGETARCH>.tar.gz`  
  e.g. `qubership-profiler-diagtools-3.1.3-linux-amd64.tar.gz`, `qubership-profiler-diagtools-3.1.3-linux-arm64.tar.gz`

Build these from the [qubership-profiler-agent](https://github.com/Netcracker/qubership-profiler-agent) repo (or your fork) with `./gradlew build` and copy the installer zip and diagtools tarballs from the build output into this directory.

## Best Practices

- Only use for testing/development
- Do not commit binary artifacts to version control
- Document any special artifact requirements
