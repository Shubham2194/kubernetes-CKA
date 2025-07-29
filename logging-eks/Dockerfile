# Start with your existing ARM64 Fluentd image
FROM arm64v8/fluentd:v1.18-debian-1

# Switch to root user to install the build dependencies and gems
USER root

# Install build dependencies for the native gem extensions.
RUN apt-get update && \
    apt-get install -y --no-install-recommends build-essential ruby-dev && \
    rm -rf /var/lib/apt/lists/*

# Install all required Fluentd plugins
RUN gem install fluent-plugin-kubernetes_metadata_filter --no-document && \
    gem install fluent-plugin-elasticsearch --no-document && \
    gem install fluent-plugin-multi-format-parser --no-document

# Clean up build dependencies to reduce image size
RUN apt-get purge -y --auto-remove build-essential ruby-dev

# Switch back to the non-root fluent user for security
USER fluent