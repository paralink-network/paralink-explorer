FROM phusion/baseimage:master-amd64

RUN apt-get update && apt-get install wget -y

WORKDIR /usr/local/bin

RUN wget https://raw.githubusercontent.com/paralink-network/paralink-explorer/main/backend/docker/backend/paralink-node/paralink-node \
    && chmod +x paralink-node

RUN wget https://raw.githubusercontent.com/paralink-network/paralink-explorer/main/backend/docker/backend/paralink-node/aura-testnet-raw.json

EXPOSE 30333 9933 9944

VOLUME ["/data"]

ENTRYPOINT ["paralink-node"]
