# syntax=docker.io/docker/dockerfile:1.20
FROM ghcr.io/dependabot/dependabot-updater-core
ARG TARGETARCH

# Install JDK for Coursier and Maven repository interaction
RUN apt-get update && apt-get install -y --no-install-recommends \
    openjdk-21-jdk-headless \
    ca-certificates-java \
    && chgrp dependabot /etc/ssl/certs/java /etc/ssl/certs/java/cacerts \
    && chmod g+w /etc/ssl/certs/java /etc/ssl/certs/java/cacerts \
    && rm -rf /var/lib/apt/lists/*

COPY --chmod=755 updater/bin/java-truststore-entrypoint.sh /usr/local/bin/java-truststore-entrypoint.sh

# Install Coursier native binary (v2.1.25)
# Coursier is SBT's built-in dependency resolver and understands Maven repos,
# Scala cross-versioning (%%), and SBT plugin repositories.
ARG COURSIER_VERSION=v2.1.25
RUN case "${TARGETARCH}" in \
      amd64) COURSIER_ARCH=x86_64 ;; \
      arm64) COURSIER_ARCH=aarch64 ;; \
      *) echo "Unsupported architecture: ${TARGETARCH}" >&2; exit 1 ;; \
    esac \
    && curl -fsSL -o /tmp/cs.gz "https://github.com/coursier/coursier/releases/download/${COURSIER_VERSION}/cs-${COURSIER_ARCH}-pc-linux.gz" \
    && gunzip -c /tmp/cs.gz > /usr/local/bin/cs \
    && rm /tmp/cs.gz \
    && chmod +x /usr/local/bin/cs

ENV DEPENDABOT_NATIVE_HELPERS_PATH="/opt"
COPY --chown=dependabot:dependabot sbt/helpers /opt/sbt/helpers

USER dependabot

RUN bash /opt/sbt/helpers/build

COPY --chown=dependabot:dependabot --parents sbt maven common $DEPENDABOT_HOME/
COPY --chown=dependabot:dependabot updater $DEPENDABOT_HOME/dependabot-updater

ENTRYPOINT ["/usr/local/bin/java-truststore-entrypoint.sh"]
