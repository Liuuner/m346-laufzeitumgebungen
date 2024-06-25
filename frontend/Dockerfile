FROM node:18.12.1-alpine AS builder
RUN apk add --no-cache bash
WORKDIR /app

ENV PATH /app/node_modules/.bin:$PATH
RUN echo "Path: $PATH"
COPY package.json ./
RUN npm install --silent
RUN npm install react-scripts@5.0.1 -g --silent
COPY . ./
RUN npm run build
RUN ls -laR

FROM nginx:alpine
COPY --from=builder /app/build/ /usr/share/nginx/html
EXPOSE 80
