ARG BASE_IMAGE=ghcr.io/netcracker/qubership-java-base:25-alpine-latest
FROM ${BASE_IMAGE}
ARG TARGETOS
ARG TARGETARCH
ARG UID=10001
ARG GID=root
ARG BASE_IMAGE_TAG=java-prof:25-alpine-unknown

LABEL maintainer="qubership"

USER root

RUN echo $BASE_IMAGE_TAG > /etc/base-image-release

ENV NC_DIAGNOSTIC_MODE=off

# renovate: datasource=maven depName=qubership-profiler-installer packageName=org.qubership.profiler:qubership-profiler-installer versioning=maven
ARG QUBERSHIP_PROFILER_VERSION=3.1.3
ARG MAVEN_CENTRAL_URL=https://repo.maven.apache.org/maven2
# Optional: override to pull from a custom repo (e.g. fork published to GitHub Packages). Same path layout as Maven Central: .../org/qubership/profiler/...
ARG QUBERSHIP_PROFILER_MAVEN_REPO_URL=
# Supported values: local, remote, github. Use github to pull from GitHub Releases (e.g. Nareshamr/qubership-profiler-agent).
ARG QUBERSHIP_PROFILER_ARTIFACT_SOURCE=remote
# For artifact_source=github: GitHub repo in owner/repo form (e.g. Nareshamr/qubership-profiler-agent). Version is used as release tag.
ARG QUBERSHIP_PROFILER_GITHUB_REPO=Nareshamr/qubership-profiler-agent

RUN --mount=type=bind,source=local-artifacts,target=/build/artifacts mkdir /app/diag && \
    cd /app/diag && \
    if [ "$QUBERSHIP_PROFILER_ARTIFACT_SOURCE" = "local" ]; then \
      cp "/build/artifacts/qubership-profiler-installer-$QUBERSHIP_PROFILER_VERSION.zip" .; \
      cp "/build/artifacts/qubership-profiler-diagtools-$QUBERSHIP_PROFILER_VERSION-$TARGETOS-$TARGETARCH.tar.gz" .; \
    elif [ "$QUBERSHIP_PROFILER_ARTIFACT_SOURCE" = "remote" ]; then \
      PROFILER_REPO="${QUBERSHIP_PROFILER_MAVEN_REPO_URL:-$MAVEN_CENTRAL_URL}" && \
      curl --fail-with-body -OL "$PROFILER_REPO/org/qubership/profiler/qubership-profiler-installer/$QUBERSHIP_PROFILER_VERSION/qubership-profiler-installer-$QUBERSHIP_PROFILER_VERSION.zip" && \
      curl --fail-with-body -OL "$PROFILER_REPO/org/qubership/profiler/qubership-profiler-diagtools/$QUBERSHIP_PROFILER_VERSION/qubership-profiler-diagtools-$QUBERSHIP_PROFILER_VERSION-$TARGETOS-$TARGETARCH.tar.gz"; \
    elif [ "$QUBERSHIP_PROFILER_ARTIFACT_SOURCE" = "github" ]; then \
      GITHUB_BASE="https://github.com/$QUBERSHIP_PROFILER_GITHUB_REPO/releases/download/$QUBERSHIP_PROFILER_VERSION" && \
      curl -fSL -o "qubership-profiler-installer-$QUBERSHIP_PROFILER_VERSION.zip" "$GITHUB_BASE/qubership-profiler-installer-$QUBERSHIP_PROFILER_VERSION.zip" && \
      curl -fSL -o "qubership-profiler-diagtools-$QUBERSHIP_PROFILER_VERSION-$TARGETOS-$TARGETARCH.tar.gz" "$GITHUB_BASE/qubership-profiler-diagtools-$QUBERSHIP_PROFILER_VERSION-$TARGETOS-$TARGETARCH.tar.gz"; \
    else \
      echo "Unsupported QUBERSHIP_PROFILER_ARTIFACT_SOURCE=$QUBERSHIP_PROFILER_ARTIFACT_SOURCE. Supported values are local, remote, github"; \
      exit 127; \
    fi && \
    unzip "qubership-profiler-installer-$QUBERSHIP_PROFILER_VERSION.zip" && \
    mv lib/qubership-profiler-agent.jar lib/agent.jar && \
    rm "qubership-profiler-installer-$QUBERSHIP_PROFILER_VERSION.zip" && \
    tar -xzf "qubership-profiler-diagtools-$QUBERSHIP_PROFILER_VERSION-$TARGETOS-$TARGETARCH.tar.gz" --strip-components=1 && \
    rm "qubership-profiler-diagtools-$QUBERSHIP_PROFILER_VERSION-$TARGETOS-$TARGETARCH.tar.gz" && \
    chown -R ${UID}:${GID} /app/diag && \
    mkdir /app/diag/dump && \
    chmod ug+rw -R /app/diag && \
    chown -R ${UID}:${GID} /app/diag

USER ${UID}:${GID}

