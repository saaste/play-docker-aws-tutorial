FROM openjdk:8-jdk-alpine

RUN apk add --update bash

COPY ./target/universal/dist /app

EXPOSE 8080

CMD bash /app/bin/my-app -Dhttp.port=8080
