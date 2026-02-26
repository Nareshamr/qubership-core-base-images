# Qubership Base Images

This repository contains secure and feature-rich base Alpine Linux images for containerized applications, designed with security and flexibility in mind.

## Available Images

### 1. Base Alpine Image

A minimal Alpine-based image with essential security and system utilities.

### 2. Java Alpine Images

There are three Java images based on Alpine:
* Java 21 with JDK and profiler: `qubership-java-base:21-alpine-xxx`
* Java 25 with JRE: `qubership-java-base:25-alpine-xxx`
* Java 25 with JRE and profiler: `qubership-java-base-prof:25-alpine-xxx`

## Usage

### Base Alpine Image

```dockerfile
FROM ghcr.io/netcracker/qubership-core-base:latest
```
**Note**: There is obsolete image labels named `qubership/core-base:latest`. Please, do not use it!

### Java Alpine Images

**Java 21 (JDK with profiler):**
```dockerfile
FROM ghcr.io/netcracker/qubership-java-base:21-alpine-latest
```

**Java 25 (JRE only):**
```dockerfile
FROM ghcr.io/netcracker/qubership-java-base:25-alpine-latest
```

**Java 25 (JRE with profiler):**
```dockerfile
FROM ghcr.io/netcracker/qubership-java-base-prof:25-alpine-latest
```

**Note**: There are obsolete image labels named `qubership/java-base:latest`. Please, do not use them!
**Note**: Images are available on GitHub Container Registry (`ghcr.io/netcracker/qubership/`) and support multi-platform builds (linux/amd64, linux/arm64). Use platform-specific tags if needed.

## Common Features

- Based on Alpine Linux 3.23.3
- Pre-configured with essential security settings
- Built-in certificate management (including Kubernetes service account certificates)
- User management with nss_wrapper support
- Volume management for certificates and NSS data
- Graceful shutdown handling
- Initialization script support
- UTF-8 locale configuration
- Multi-platform support (linux/amd64, linux/arm64)

## Base Alpine Image Details

- **Base Image**: `alpine:3.23.3`
- **Default User**: `appuser` (UID: 10001)
- **Default Home**: `/app`
- **Default Language**: `en_US.UTF-8`

### Dependencies

- `ca-certificates`: Latest version
- `curl`: Latest version
- `bash`: Latest version
- `nss_wrapper`: Latest version

### Volume Mounts

- `/tmp`
- `/app/nss`
- `/etc/ssl/certs`
- `/usr/local/share/ca-certificates`

## Java Alpine Image Details

### Java 21 Image

- **Base Image**: `alpine:3.23.3` (via core base image)
- **Java Version**: OpenJDK 21 (JDK)
- **Default User**: `appuser` (UID: 10001)
- **Default Home**: `/app`
- **Default Language**: `en_US.UTF-8`

#### Additional Dependencies

- `openjdk21-jdk`: Latest version
- `fontconfig`: Latest version
- `font-dejavu`: Latest version
- `procps-ng`: Latest version
- `curl`: Latest version
- `bash`: Latest version
- `libstdc++`: Latest version
- `nss_wrapper`: Latest version
- And all base Alpine dependencies

#### Java 21 Environment Variables

- `JAVA_HOME`: `/usr/lib/jvm/java-21-openjdk`
- `MALLOC_ARENA_MAX`: 2
- `MALLOC_MMAP_THRESHOLD_`: 131072
- `MALLOC_TRIM_THRESHOLD_`: 131072
- `MALLOC_TOP_PAD_`: 131072
- `MALLOC_MMAP_MAX`: 65536

### Java 25 Images

- **Base Image**: `alpine:3.23.3` (via core base image)
- **Java Version**: OpenJDK 25 (JRE)
- **Default User**: `appuser` (UID: 10001)
- **Default Home**: `/app`
- **Default Language**: `en_US.UTF-8`

#### Additional Dependencies

- `openjdk25-jre-headless`: Latest version
- `curl`: Latest version
- `bash`: Latest version
- `nss_wrapper`: Latest version
- And all base Alpine dependencies

#### Java 25 Environment Variables

- `JAVA_HOME`: `/usr/lib/jvm/default-jvm`
- `MALLOC_ARENA_MAX`: 2
- `MALLOC_MMAP_THRESHOLD_`: 131072
- `MALLOC_TRIM_THRESHOLD_`: 131072
- `MALLOC_TOP_PAD_`: 131072
- `MALLOC_MMAP_MAX`: 65536

### Qubership Profiler Integration

The Java Alpine images (Java 21 and Java 25 profiler variants) include built-in support for the Qubership profiler. The agent is built from the [qubership-profiler-agent](https://github.com/Netcracker/qubership-profiler-agent) repository.

- **Profiler Version**: 3.1.3 (configurable via build arg `QUBERSHIP_PROFILER_VERSION`)
- **Where the default version is pulled from**: **Maven Central** (`https://repo.maven.apache.org/maven2`). The [Netcracker/qubership-profiler-agent](https://github.com/Netcracker/qubership-profiler-agent) project publishes releases (e.g. v3.1.3) to Maven Central via its release workflow, and the base images download `qubership-profiler-installer` and `qubership-profiler-diagtools` from there.
- **Artifact Source**: Configurable via build arg `QUBERSHIP_PROFILER_ARTIFACT_SOURCE` (`local`, `remote`, or `github`)
- **Custom Maven repo**: Optional build arg `QUBERSHIP_PROFILER_MAVEN_REPO_URL` overrides the remote URL when using a fork or private repo (same path layout as Maven Central: `.../org/qubership/profiler/...`).
- **GitHub Releases**: Use `QUBERSHIP_PROFILER_ARTIFACT_SOURCE=github` with `QUBERSHIP_PROFILER_GITHUB_REPO` (e.g. `Nareshamr/qubership-profiler-agent`) to pull assets directly from a GitHub release tag.
- **Enable Profiler**: Set environment variable `PROFILER_ENABLED=true`
- **Profiler Directory**: `/app/diag`
- **Dump Directory**: `/app/diag/dump`
- **Multi-platform Support**: Automatically downloads platform-specific artifacts based on `TARGETOS` and `TARGETARCH` build args

#### Using a profiler fork (e.g. Nareshamr/qubership-profiler-agent)

To integrate a fork such as [Nareshamr/qubership-profiler-agent](https://github.com/Nareshamr/qubership-profiler-agent) instead of the upstream Netcracker release:

**Option A – Local artifacts (no publish step)**

1. Clone and build the fork (Java 17 required):  
   `git clone https://github.com/Nareshamr/qubership-profiler-agent.git && cd qubership-profiler-agent && ./gradlew build`
2. From the fork build output, copy into `qubership-core-base-images/local-artifacts/`:
   - `qubership-profiler-installer-<VERSION>.zip` (from the `installer` module)
   - `qubership-profiler-diagtools-<VERSION>-linux-amd64.tar.gz` and/or `...-linux-arm64.tar.gz` (from the `diagtools` module)
3. Build the profiler image with local source and the same version:
   ```bash
   docker build -f images/java-21-prof/Dockerfile \
     --build-arg QUBERSHIP_PROFILER_ARTIFACT_SOURCE=local \
     --build-arg QUBERSHIP_PROFILER_VERSION=<VERSION> \
     .
   ```
   Use `images/java-25-prof/Dockerfile` for Java 25. For multi-platform, ensure the diagtools tarballs for each `TARGETOS`/`TARGETARCH` are present and named as the Dockerfile expects.

**Option B – Custom Maven repository**

1. Publish the fork’s artifacts to a Maven-compatible repository (e.g. GitHub Packages or an internal repo) using the same group/artifact IDs and path layout as upstream (`org.qubership.profiler:qubership-profiler-installer`, `qubership-profiler-diagtools`).
2. Build the profiler image with the custom repo URL and version:
   ```bash
   docker build -f images/java-21-prof/Dockerfile \
     --build-arg QUBERSHIP_PROFILER_ARTIFACT_SOURCE=remote \
     --build-arg QUBERSHIP_PROFILER_MAVEN_REPO_URL=https://your-repo-url/maven2 \
     --build-arg QUBERSHIP_PROFILER_VERSION=<VERSION> \
     .
   ```
   Omit `QUBERSHIP_PROFILER_MAVEN_REPO_URL` to keep using Maven Central (upstream).

**Option C – GitHub Releases**

To use a release from a fork (e.g. [Nareshamr/qubership-profiler-agent releases](https://github.com/Nareshamr/qubership-profiler-agent/releases)), the release must include these **exact asset names**:

- `qubership-profiler-installer-<VERSION>.zip`
- `qubership-profiler-diagtools-<VERSION>-linux-amd64.tar.gz` (and `qubership-profiler-diagtools-<VERSION>-linux-arm64.tar.gz` for arm64 builds)

Then build with `artifact_source=github`, the repo, and the release tag (e.g. `1.0`):

```bash
docker build -f images/java-21-prof/Dockerfile \
  --build-arg QUBERSHIP_PROFILER_ARTIFACT_SOURCE=github \
  --build-arg QUBERSHIP_PROFILER_GITHUB_REPO=Nareshamr/qubership-profiler-agent \
  --build-arg QUBERSHIP_PROFILER_VERSION=1.0 \
  .
```

Use `images/java-25-prof/Dockerfile` for Java 25. The default `QUBERSHIP_PROFILER_GITHUB_REPO` is already `Nareshamr/qubership-profiler-agent`, so for your 1.0 release you can use:

```bash
docker build -f images/java-21-prof/Dockerfile \
  --build-arg QUBERSHIP_PROFILER_ARTIFACT_SOURCE=github \
  --build-arg QUBERSHIP_PROFILER_VERSION=1.0 \
  .
```

### Certificate Management

- **Certificate Location**: `/etc/ssl/certs/java/cacerts` (Java keystore)
- **Certificate Password**: Configurable via `CERTIFICATE_FILE_PASSWORD` environment variable
- **Certificate Sources**: 
  - `/tmp/cert/` directory (`.crt`, `.cer`, or `.pem` files)
  - Kubernetes service account certificates from `/var/run/secrets/kubernetes.io/serviceaccount/ca.crt`

## Directory Structure

```
/app
├── init.d/          # Initialization scripts
├── nss/             # NSS wrapper data
├── ncdiag/          # Diagnostic and troubleshooting data (base image)
├── diag/            # Profiler diagnostics (Java profiler images only)
│   ├── lib/        # Profiler libraries
│   └── dump/       # Profiler dumps
└── volumes/
    └── certs/      # Certificate storage
```

## Security Features

- Non-root user execution (UID: 10001)
- Secure certificate handling
- Proper file permissions
- Volume isolation for sensitive data
- NSS wrapper integration

## Initialization Process

The entrypoint script performs the following operations:

1. **Restores volume data**: Copies certificate data from `/app/volumes/certs/` to the appropriate certificate locations
2. **Creates user if necessary**: Uses nss_wrapper to create the appuser entry if the user doesn't exist in `/etc/passwd`
3. **Loads certificates to trust store**: 
   - Scans `/tmp/cert/` directory for certificate files
   - Automatically detects and loads Kubernetes service account certificates from `/var/run/secrets/kubernetes.io/serviceaccount/ca.crt`
   - For Java images: imports certificates into the Java keystore using `keytool`
   - For base images: copies certificates and runs `update-ca-certificates`
4. **Loads profiler bootstrap** (Java profiler images only): Sources `/app/diag/diag-bootstrap.sh` to make profiler functions available
5. **Executes initialization scripts**: Runs all `.sh` scripts from `/app/init.d/` in alphabetical order (only in non-interactive mode)
6. **Runs the main application**: Executes the provided command with proper signal handling and crash dump collection

### Adding Custom Certificates

Certificates can be added in two ways:

1. **Manual placement**: Place your certificates (`.crt`, `.cer`, or `.pem` files) in `/tmp/cert/` directory. They will be automatically loaded into the trust store.

2. **Kubernetes integration**: The image automatically detects and loads Kubernetes service account certificates from `/var/run/secrets/kubernetes.io/serviceaccount/ca.crt` if mounted.

For Java images, certificates are imported into the Java keystore. The keystore password can be customized via the `CERTIFICATE_FILE_PASSWORD` environment variable (default: `changeit`).

### Adding Initialization Scripts

Place your initialization scripts (`.sh` files) in `/app/init.d/`. They will be executed in alphabetical order before the main application starts.

### Using the Qubership Profiler

To enable the profiler in Java Alpine images (Java 21 or Java 25 profiler variants):

```bash
# Set environment variable to enable profiler
export PROFILER_ENABLED=true

# Run your Java application
java -jar your-app.jar
```

The profiler will automatically:
- Load the profiler agent from `/app/diag/lib/agent.jar`
- Set up dump directory at `/app/diag/dump`
- Configure Java tool options for profiling via `JAVA_TOOL_OPTIONS`
- Provide crash dump functionality via `send_crash_dump` function

The profiler agent is automatically loaded via `diag-bootstrap.sh` script sourced in the entrypoint.

## Signal Handling

The images include comprehensive signal handling for graceful shutdowns and proper process management. They support all standard Linux signals (SIGHUP, SIGINT, SIGQUIT, SIGTERM, etc.) and ensure proper cleanup on container termination. 

For SIGTERM signals, there is a 10-second delay to prevent 503/502 errors during deployment rollouts. The entrypoint script properly forwards all signals to the child process and handles exit codes appropriately.

**Note**: Signal handling is disabled when running in interactive shell mode (`bash` or `sh` commands) to avoid interfering with terminal signal handling.

## Logging

This project provides a helper logging function named `log` used by the entrypoint script. Below are usage examples and important interpreter limitations.
The log function is exported from entrypoint script and is available only to child processes that are Bash. For example, `bash -c 'log INFO "msg"'` works, but `sh -c 'log ...'` will not.

custom_script.sh
```bash
#!/usr/bin/env bash
log INFO Hi
```
Log output:
```bash
#> ./custom_script.sh
[2026-01-22T08:58:47.000] [INFO] [request_id=-] [tenant_id=-] [thread=-] [class=-] [custom_script.sh] Hi 
```

## Read-only mode support
If you need to run a container in a read-only host environment, you must mount the required writable paths as --tmpfs volumes or as emptyDir volumes in Kubernetes.

* `/tmp` - to persist temporary files
* `/etc/env` - to manage environment configurations
* `/app/nss` - to manage NSS (Network Security Services) data
* `/app/ncdiag` - to store diagnostic and troubleshooting data
* `/etc/ssl/certs/java` - to handle Java SSL certificates, or `/etc/ssl/certs` for non-Java images


## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct and the process for submitting pull requests.

## License

This project is licensed under the terms specified in the [LICENSE](LICENSE) file.
