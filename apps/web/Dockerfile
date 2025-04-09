ARG NODE_VERSION=22-alpine
FROM node:${NODE_VERSION} AS base
WORKDIR /app

FROM base AS pruner
RUN npm i -g turbo
COPY . .

RUN turbo prune --scope=@devopsktm/web --docker

FROM base AS installer
RUN npm i -g pnpm turbo
COPY --from=pruner /app/out/json/ .
COPY --from=pruner /app/out/pnpm-lock.yaml ./pnpm-lock.yaml
RUN pnpm install --frozen-lockfile

COPY --from=pruner /app/out/full/ .
COPY turbo.json turbo.json

ARG TURBO_TOKEN
ARG TURBO_TEAM

ENV NEXT_SHARP_PATH=/app/node_modules/sharp
ENV NEXT_TELEMETRY_DISABLED=1
ENV NEXT_PRIVATE_STANDALONE=true
ENV TURBO_TOKEN=$TURBO_TOKEN
ENV TURBO_TEAM=$TURBO_TEAM
    
RUN turbo run build --filter=@devopsktm/web

# Final Image
FROM base AS runner

# Don't run production as root
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

USER nextjs

# Copy necessary files from installer stage
COPY --from=installer /app/apps/web/next.config.js .
COPY --from=installer /app/apps/web/package.json .
 
# Automatically leverage output traces to reduce image size
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/.next/standalone ./
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/.next/static ./apps/web/.next/static
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/public ./apps/web/public


ENV NODE_ENV=production
# ENV NEXT_SHARP_PATH=/app/node_modules/sharp
ENV NEXT_TELEMETRY_DISABLED=1
ENV NEXT_PRIVATE_STANDALONE=true
ENV HOSTNAME="0.0.0.0"

# Use the entrypoint script to dynamically inject environment variables at runtime
CMD exec node /app/apps/web/server.js
