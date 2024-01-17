# Stage 1: Build Angular application
FROM node:14-alpine as build-stage

RUN mkdir -p /app
WORKDIR /app

COPY package*.json ./

RUN apk add --no-cache nodejs npm

#ENV PATH="${PATH}:/app/node_modules/.bin"

RUN npm install

COPY .. .
RUN npm run build

EXPOSE 4200

CMD ["npm", "start"]

# Stage 2: Nginx to serve the Angular app
FROM nginx:alpine as nginx
COPY --from=build-stage /app/dist/angular-app /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
