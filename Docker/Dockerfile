# Base image from the official Kuma repository
FROM kumahq/kuma-cp:2.7.1-arm64

# Temporarily switch to root to create the directory
USER root

# Create a writable directory
RUN mkdir -p /custom-dir && chmod -R 755 /custom-dir

# Switch back to the non-root user for security
USER 1001

# Drop unnecessary capabilities
ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["/kuma/bin/kuma-cp"]