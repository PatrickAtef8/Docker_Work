FROM ubuntu:22.04

RUN apt update && apt install -y g++

WORKDIR /app

COPY main.cpp .

RUN g++ main.cpp -o app

CMD ["./app"]

