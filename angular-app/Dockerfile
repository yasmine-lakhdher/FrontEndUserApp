# Stage 1: Build Angular application
FROM node:18-alpine as build-stage

RUN mkdir -p /app
WORKDIR /app

COPY .. .
#COPY package*.json ./
RUN npm install

RUN npm run build

# Stage 2: Nginx to serve the Angular app
FROM nginx:alpine as nginx
COPY --from=build-stage /app/dist/frontend /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf