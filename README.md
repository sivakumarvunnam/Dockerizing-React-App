This project was bootstrapped with [Create React App](https://github.com/facebook/create-react-app).

## Production
Let’s create a Dockerfile for use in production called Dockerfile:
```
# build environment
FROM node:13.12.0-alpine as build
MAINTAINER sivakumarvunnam1@gmail.com
WORKDIR /app
ENV PATH /app/node_modules/.bin:$PATH
COPY package.json ./
COPY package-lock.json ./
RUN npm ci --silent
RUN npm install react-scripts@3.4.1 -g --silent
COPY . ./
RUN npm run build

# production environment
FROM nginx:stable-alpine
COPY --from=build /app/build /usr/share/nginx/html
# new
COPY nginx/nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

```

