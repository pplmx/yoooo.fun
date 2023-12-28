---
categories:
    - docker
date: 2023-07-28T10:44:11Z
description: copy a wget or curl binary to distroless image to healthcheck
keywords: docker,distroless,healthcheck,wget,curl
lastmod: 2023-07-28T14:06:12Z
tags:
    - docker
    - distroless
    - healthcheck
    - wget
    - curl
title: How to healthcheck distroless image
---



# How to healthcheck in distroless image

## Dockerfile

```dockerfile
### For HEALTHCHECK
FROM busybox AS wgeter

# wget grpc-health-probe
RUN wget -O /tmp/hc https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/v0.4.19/grpc_health_probe-linux-amd64 \
    && chmod +x /tmp/hc \
    && mv /tmp/hc /bin/hc

### Deploy
# FROM gcr.io/distroless/static
FROM gcr.dockerproxy.com/distroless/static
ENV TZ=Asia/Shanghai

# copy wget and health, one for http healthcheck, one for grpc healthcheck
COPY --from=wgeter /bin/wget /bin/wget
COPY --from=wgeter /bin/hc /bin/hc

HEALTHCHECK --interval=5s --timeout=3s --start-period=5s --retries=3 CMD ["hc", "-addr=localhost:9000"]
```
