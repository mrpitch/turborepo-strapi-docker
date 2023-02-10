FROM node:16-alpine AS builder
RUN apk add --no-cache libc6-compat
RUN apk update
# Set working directory
WORKDIR /usr/src/app
RUN npm install -g turbo
COPY . .
RUN turbo prune --scope=api-cms --docker

# Add lockfile and package.json's of isolated subworkspace
FROM node:16-alpine AS installer
RUN apk add --no-cache libc6-compat
RUN apk update
RUN npm install -g pnpm@7.26.2
WORKDIR /usr/src/app

# First install the dependencies (as they change less often)
COPY .gitignore .gitignore
COPY --from=builder /usr/src/app/out/json/ .
COPY --from=builder /usr/src/app/out/pnpm-lock.yaml ./pnpm-lock.yaml
COPY --from=builder /usr/src/app/out/pnpm-workspace.yaml ./pnpm-workspace.yaml
RUN pnpm --filter=api-cms... install --shamefully-hoist

# Build the project
COPY --from=builder /usr/src/app/out/full/ .
COPY turbo.json turbo.json
RUN pnpm turbo run build:strapi --filter=api-cms

# Don't run production as root
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 apicms
USER apicms

EXPOSE 1337
CMD ["pnpm", "start:strapi"]



