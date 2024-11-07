###################
# Development
###################

FROM node:20-alpine AS development

RUN yarn global add @nestjs/cli

EXPOSE 3000


###################
# Builder
###################
FROM node:20-alpine AS builder

WORKDIR /app

COPY --chown=node:node package.json ./
COPY --chown=node:node yarn.lock ./

COPY --chown=node:node . .

# RUN yarn install
RUN yarn build

ENV NODE_ENV production

# RUN yarn install --frozen-lockfile --production && yarn cache clean
# RUN yarn install --immutable --immutable-cache --check-cache
RUN yarn install --production --immutable --immutable-cache --check-cache && yarn cache clean

USER node


###################
# PRODUCTION
###################
FROM node:20-alpine AS production

WORKDIR /app

COPY --chown=node:node --from=builder /app/node_modules ./node_modules
COPY --chown=node:node --from=builder /app/dist ./dist

CMD [ "node", "dist/main.js" ]