FROM node:16-bullseye-slim AS packages

WORKDIR /app
COPY ./backstage-app/packages/backend/package.json ./backstage-app/yarn.lock ./
COPY ./backstage-app/packages packages
# Comment this out if you don't have any internal plugins
COPY ./backstage-app/plugins plugins

RUN find packages \! -name "package.json" -mindepth 2 -maxdepth 2 -exec rm -rf {} \+

# Stage 2 - Install dependencies and build packages
FROM node:16-bullseye-slim AS build

WORKDIR /app
COPY --from=packages /app .

ENV TZ="UTC"
ENV NEXUS_REPO_URL="https://****.ru"

ARG DEBIAN_FRONTEND=noninteractive
ARG NEXUS_REPO_URI="${NEXUS_REPO_URL}/repository"

ADD https://****.ru/cert.pem /usr/local/share/ca-certificates/

RUN bash -c 'echo -e "deb ${NEXUS_REPO_URI}/debian-proxy bullseye main contrib non-free\ndeb ${NEXUS_REPO_URI}/debian-security bullseye-security main contrib non-free\ndeb ${NEXUS_REPO_URI}/debian-proxy bullseye-updates main contrib non-free" > /etc/apt/sources.list' \
    && bash -c 'echo -e "APT::Install-Recommends \"0\";" > /etc/apt/apt.conf.d/' \
    && mkdir -p /etc/ssl/certs/ \
    && cat /usr/local/share/ca-certificates/cert.crt > /etc/ssl/certs/ca-certificates.crt\
    && apt-get update \
    && apt-get install --yes --no-install-recommends --fix-missing \
        ca-certificates \
        openssl \
        wget \
        python3 \
        build-essential \
    && update-ca-certificates --fresh \
    && apt-get autoremove -yqq --purge \
    && apt-get clean \
    && rm -rf \
        /var/lib/apt/lists/* \
        /tmp/* \
        /var/tmp/* \
        /usr/share/man \
        /usr/share/doc \
        /usr/share/doc-base \
    && yarn config set python /usr/bin/python3 \
    && yarn config set "strict-ssl" false \
    && yarn config set registry "${NEXUS_REPO_URL}/repository/npm-all/"  \
    && cd /tmp && wget ${NEXUS_REPO_URL}/repository/nodejs-headers/v16.18.1/node-v16.18.1-headers.tar.gz \
    && npm config set tarball "/tmp/node-v16.18.1-headers.tar.gz"

ENV CYPRESS_INSTALL_BINARY=${NEXUS_REPO_URL}/repository/raw-download.cypress.io-proxy/
ENV CYPRESS_DOWNLOAD_MIRROR=${NEXUS_REPO_URL}/repository/raw-download.cypress.io-proxy/
ENV NODE_EXTRA_CA_CERTS=/etc/ssl/certs/ca-certificates.crt

COPY ./backstage-app/ .

RUN yarn install --frozen-lockfile --network-timeout 600000 \
    && rm -rf "$(yarn cache dir)" \
    && yarn tsc \
    && yarn --cwd packages/backend build

# Stage 3 - Build the actual backend image and install production dependencies
FROM node:16-bullseye-slim

LABEL VERSION=0.0.1

WORKDIR /app

ENV TZ="UTC"
ENV NEXUS_REPO_URL="https://****.ru"

ARG DEBIAN_FRONTEND=noninteractive
ARG NEXUS_REPO_URI="${NEXUS_REPO_URL}/repository"

ADD https://****.ru/cert.pem /usr/local/share/ca-certificates/
RUN bash -c 'echo -e "deb ${NEXUS_REPO_URI}/debian-proxy bullseye main contrib non-free\ndeb ${NEXUS_REPO_URI}/debian-security bullseye-security main contrib non-free\ndeb ${NEXUS_REPO_URI}/debian-proxy bullseye-updates main contrib non-free" > /etc/apt/sources.list' \
    && bash -c 'echo -e "APT::Install-Recommends \"0\";" > /etc/apt/apt.conf.d/' \
    && mkdir -p /etc/ssl/certs/ \
    && cat /usr/local/share/ca-certificates/cert.crt > /etc/ssl/certs/ca-certificates.crt\
    && apt-get update \
    && apt-get install --yes --no-install-recommends --fix-missing \
        ca-certificates \
        openssl \
        wget \
        python3 \
        build-essential \
    && update-ca-certificates --fresh \
    && apt-get autoremove -yqq --purge \
    && apt-get clean \
    && rm -rf \
        /var/lib/apt/lists/* \
        /tmp/* \
        /var/tmp/* \
        /usr/share/man \
        /usr/share/doc \
        /usr/share/doc-base \
    && yarn config set python /usr/bin/python3 \
    && yarn config set "strict-ssl" false \
    && yarn config set registry "${NEXUS_REPO_URL}/repository/npm-all/" \
    && cd /tmp && wget ${NEXUS_REPO_URL}/repository/nodejs-headers/v16.18.1/node-v16.18.1-headers.tar.gz \
    && npm config set tarball "/tmp/node-v16.18.1-headers.tar.gz"

ENV NODE_EXTRA_CA_CERTS=/etc/ssl/certs/ca-certificates.crt

# Copy the install dependencies from the build stage and context
COPY --from=build /app/yarn.lock /app/package.json /app/packages/backend/dist/skeleton.tar.gz ./
RUN tar xzf skeleton.tar.gz && rm skeleton.tar.gz \
    && yarn install --frozen-lockfile --production --network-timeout 600000 \ 
    && rm -rf "$(yarn cache dir)"

# Copy the built packages from the build stage
COPY --from=build /app/packages/backend/dist/bundle.tar.gz .
RUN tar xzf bundle.tar.gz && rm bundle.tar.gz

# Copy any other files that we need at runtime
COPY ./backstage-app/app-config.yaml ./

CMD ["node", "packages/backend", "--config", "app-config.yaml"]
