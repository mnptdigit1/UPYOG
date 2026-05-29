FROM node:18.20.4-alpine AS build
#FROM node:14.21.3-alpine AS build
#FROM upyogio/alpine-node-builder-14:yarn AS build
#FROM ghcr.io/egovernments/alpine-node-builder-14:yarn AS build
RUN apk update && apk upgrade
#RUN apk add --no-cache git>2.30.0
RUN apk add --no-cache git>2.30.0 python3 make g++
RUN ln -sf python3 /usr/bin/python
ARG WORK_DIR
WORKDIR /app
#ENV NODE_OPTIONS "--max-old-space-size=4096"
ENV NODE_OPTIONS "--max-old-space-size=4096 --openssl-legacy-provider"
ENV GENERATE_SOURCEMAP "false"
ENV REACT_APP_SECRET_KEY "PaXJBEnwt4tGSo/8"

COPY ${WORK_DIR} .
RUN ls -lah

#RUN node web/envs.js
RUN cd web/ \
    && node envs.js \
    && node -e 'console.log(v8.getHeapStatistics().heap_size_limit/(1024*1024))' \
    && cd micro-ui-internals \
    && node -e "const fs=require('fs');const p=JSON.parse(fs.readFileSync('package.json'));p.resolutions=p.resolutions||{};p.resolutions['node-sass']='8.0.0';fs.writeFileSync('package.json',JSON.stringify(p,null,2));" \
    && yarn install \
    && yarn build \
    && find . -name "node_modules" -type d -prune -exec rm -rf '{}' \; \
    && cd .. \
    && rm -rf node_modules \
    && yarn install --force \
    && yarn build 

FROM nginx:mainline-alpine
#FROM ghcr.io/egovernments/nginx:mainline-alpine
ENV WORK_DIR=/var/web/digit-ui

RUN mkdir -p ${WORK_DIR}

COPY --from=build /app/web/build ${WORK_DIR}/
COPY --from=build /app/web/docker/nginx.conf /etc/nginx/conf.d/default.conf
