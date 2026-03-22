FROM node:20-bookworm-slim AS development
WORKDIR /app
ENV NODE_ENV=development
COPY package.json ./
RUN npm install
COPY src ./src
RUN chown -R node:node /app
USER node
EXPOSE 3000
CMD ["npm", "run", "dev"]

FROM node:20-bookworm-slim AS production
WORKDIR /app
ENV NODE_ENV=production
COPY package.json ./
RUN npm install --omit=dev
COPY src ./src
RUN chown -R node:node /app
USER node
EXPOSE 3000
CMD ["npm", "start"]
