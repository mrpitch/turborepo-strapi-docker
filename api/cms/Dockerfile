###
## for referece please see https://github.com/BretFisher/nodejs-rocks-in-docker/blob/main/dockerfiles/ubuntu-copy.Dockerfile
###
###
## ubuntu base with nodejs coppied in from official image, for a more secure base
###
#cache our node version for installing later
FROM node:20-bullseye-slim as node
FROM ubuntu:focal-20230126 as base

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
COPY --from=installer --chown=node:node /app/out/json/ .
COPY --from=installer --chown=node:node /app/out/pnpm*.yaml ./
RUN pnpm install --prod --filter=api-cms 

# Copy /full and build the project
COPY --from=installer --chown=node:node /app/out/full/ .
RUN pnpm build:strapi:prod

#run it
CMD ["pnpm", "start:strapi:prod"]
