FROM registry.access.redhat.com/ubi10/go-toolset:latest AS builder

ARG TARGETARCH=amd64

USER root

RUN OPERATOR_SDK_VERSION=$(curl -fsSL https://api.github.com/repos/operator-framework/operator-sdk/releases/latest | grep '"tag_name"' | sed 's/.*"v\([^"]*\)".*/\1/') && \
    curl -fLo /tmp/operator-sdk \
    "https://github.com/operator-framework/operator-sdk/releases/download/v${OPERATOR_SDK_VERSION}/operator-sdk_linux_${TARGETARCH}" && \
    chmod +x /tmp/operator-sdk

RUN YQ_VERSION=$(curl -fsSL https://api.github.com/repos/mikefarah/yq/releases/latest | grep '"tag_name"' | sed 's/.*"v\([^"]*\)".*/\1/') && \
    curl -fLo /tmp/yq \
    "https://github.com/mikefarah/yq/releases/download/v${YQ_VERSION}/yq_linux_${TARGETARCH}" && \
    chmod +x /tmp/yq

RUN curl -fLo /tmp/oc.tar.gz \
    "https://mirror.openshift.com/pub/openshift-v4/${TARGETARCH}/clients/ocp/stable/openshift-client-linux.tar.gz" && \
    tar -xzf /tmp/oc.tar.gz -C /tmp oc kubectl && \
    chmod +x /tmp/oc /tmp/kubectl

RUN GOPATH=/tmp/go go install github.com/onsi/ginkgo/v2/ginkgo@latest && \
    mv /tmp/go/bin/ginkgo /tmp/ginkgo

FROM registry.access.redhat.com/ubi10/go-toolset:latest

USER root

RUN dnf -y install \
    jq \
    skopeo \
    make \
    && dnf clean all

COPY --from=builder /tmp/operator-sdk /usr/local/bin/operator-sdk
COPY --from=builder /tmp/oc /usr/local/bin/oc
COPY --from=builder /tmp/kubectl /usr/local/bin/kubectl
COPY --from=builder /tmp/yq /usr/local/bin/yq
COPY --from=builder /tmp/ginkgo /usr/local/bin/ginkgo

ENV PATH="/usr/local/bin:${PATH}"
