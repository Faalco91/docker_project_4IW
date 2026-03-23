FROM node:20-bookworm-slim AS development
WORKDIR /app
ENV NODE_ENV=development
COPY package.json ./
RUN npm install
COPY . .
RUN chown -R node:node /app
USER node
EXPOSE 8080
CMD ["npm", "run", "dev"]

FROM node:20-bookworm-slim AS build
WORKDIR /app
COPY package.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:stable-bookworm AS production
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 8080
