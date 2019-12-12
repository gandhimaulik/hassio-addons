ARG BUILD_FROM
FROM $BUILD_FROM

ARG NODE_VERSION=10
FROM node:${NODE_VERSION}-alpine
RUN apk add --no-cache make gcc g++ python
ARG version=latest
WORKDIR /home/theia
ADD $version.package.json ./package.json
ARG GITHUB_TOKEN

RUN yarn --pure-lockfile && \
    NODE_OPTIONS="--max_old_space_size=4096" yarn theia build && \
    yarn --production && \
    yarn autoclean --init && \
    echo *.ts >> .yarnclean && \
    echo *.ts.map >> .yarnclean && \
    echo *.spec.* >> .yarnclean && \
    yarn autoclean --force && \
    yarn cache clean


FROM node:${NODE_VERSION}-alpine
# See : https://github.com/theia-ide/theia-apps/issues/34

RUN chmod g+rw /home && \
    mkdir -p /home/theia 
RUN apk add --no-cache git openssh bash
ENV HOME /home/theia
WORKDIR /home/theia
COPY --from=0 /home/theia /home/theia
EXPOSE 8099
ENV SHELL /bin/bash
ENV USE_LOCAL_GIT true
#ENTRYPOINT [ "node", "/home/theia/src-gen/backend/main.js", "/home/project", "--hostname=0.0.0.0" ]

# Setup base
ARG HASSIO_AUTH_VERSION
RUN apk add --no-cache \
      git \
      nginx \
      nginx-mod-http-lua \
      lua-resty-http \
      openssh-client \
    && git clone --depth 1 -b ${HASSIO_AUTH_VERSION} "https://github.com/home-assistant/hassio-auth" \
    && cp -f hassio-auth/nginx-frontend/ha-auth.lua /etc/nginx/ \
    && cp -f hassio-auth/nginx-frontend/example/nginx-ingress.conf /etc/nginx/ \
    && rm -fr /usr/src/hassio-auth

# Copy data
COPY data/run.sh /

CMD ["/run.sh"]
