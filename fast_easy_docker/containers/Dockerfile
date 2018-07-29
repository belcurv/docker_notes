FROM node:8.11.3-alpine

WORKDIR /app

ADD . /app

RUN apk --update add nano

CMD [ "node", "server.js" ]