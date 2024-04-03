###
## for referece please see https://github.com/BretFisher/nodejs-rocks-in-docker/blob/main/dockerfiles/ubuntu-copy.Dockerfile
###
###
## ubuntu base with nodejs coppied in from official image, for a more secure base
###
#cache our node version for installing later
FROM node:20-bullseye-slim as node
FROM ubuntu:focal-20230126 as base

# replace npm in CMD with tini for better kernel signal handling
# You may also need development tools to build native npm addons:
# apt-get install gcc g++ make
RUN apt-get update \
    && apt-get -qq install -y --no-install-recommends \
    tini \
    && rm -rf /var/lib/apt/lists/*
ENTRYPOINT ["/usr/bin/tini", "--"]

# new way to get node, let's copy in the specific version we want from a docker image
# this avoids depdency package installs (python3) that the deb package requires
COPY --from=node /usr/local/include/ /usr/local/include/
COPY --from=node /usr/local/lib/ /usr/local/lib/
COPY --from=node /usr/local/bin/ /usr/local/bin/
# ensures we fix symlinks for npx, yarn and PnPm
RUN corepack disable && corepack enable

#custom: install turbo, we need this later
RUN npm install -g turbo

# create node user and group, then create app dir
RUN groupadd --gid 1000 node \
    && useradd --uid 1000 --gid node --shell /bin/bash --create-home node \
    && mkdir /app \
    && chown -R node:node /app

# run turbo prune to create strapi instance form monorepo and copy everything in next step
FROM base as installer
WORKDIR /app
COPY  . .
RUN turbo prune --scope=api-cms --docker

# copy all stuff needed from the pruned repo, install & build
FROM base as prod
ARG NODE_ENV=production
ENV NODE_ENV=${NODE_ENV}
EXPOSE 1337
WORKDIR /app
USER node
# First copy & install the dependencies (as they change less often), be aware to use node user
COPY --chown=node:node .gitignore .gitignore
COPY --from=installer --chown=node:node /app/out/json/ .
COPY --from=installer --chown=node:node /app/out/pnpm*.yaml ./
RUN pnpm install --prod --frozen-lockfile --shamefully-hoist --filter=api-cms 

# Copy /full and build the project
COPY --from=installer --chown=node:node /app/out/full/ .
RUN pnpm build:strapi

#run it - well this is not working yet :D (could use CMD ["pnpm", "start:strapi"], but there might be an issue with PID and zombie process...)
#CMD ["node", "./app/api/cms/dist/"]
CMD ["pnpm", "start:strapi"]
