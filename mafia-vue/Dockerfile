FROM node:24.14-trixie-slim@sha256:c319bb4fac67c01ced508b67193a0397e02d37555d8f9b72958649efd302b7f8 as build-stage
WORKDIR /app
COPY package*.json ./
RUN npm ci --ignore-scripts
COPY ./ .
RUN npm run build

FROM nginx:stable-alpine3.23-slim@sha256:470297f0c1a833c3b3089542cb38d72c83a4beb2a449ef807d33267c0a063d7c as artifact-stage
ARG CI_ENV=noci
ARG GIT_COMMIT=git_commit_undefined
ARG GIT_BRANCH=git_branch_undefined
ARG VERSION=not_versioned
COPY --from=build-stage /app/dist /usr/share/nginx/html
RUN echo "version=$VERSION" > /usr/share/nginx/html/version.html && echo "commit=$GIT_COMMIT" >> /usr/share/nginx/html/version.html && echo "branch=$GIT_BRANCH" >> /usr/share/nginx/html/version.html
RUN chmod 0755 /usr/share/nginx/html && chmod 0644 -R /usr/share/nginx/html/*
RUN find /usr/share/nginx/html -type d -exec chmod 0755 {} \;
COPY nginx/default.conf.template /etc/nginx/templates/
COPY nginx/nginx.conf /etc/nginx/
ENV HUB_HOST 172.17.0.1
ENV HUB_PORT 3000
LABEL git_commit $GIT_COMMIT
LABEL git_branch $GIT_BRANCH
LABEL ci_environment $CI_ENV
LABEL version $VERSION