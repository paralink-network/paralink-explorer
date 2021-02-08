# Paralink explorer

## Install

Install mono-repo:

```
git clone https://github.com/paralink-network/paralink-explorer
cd paralink-explorer
yarn
```

### Frontend

You will need `nodejs`:

```
yarn workspace frontend dev
```

That will start a dev frontend with hot reload. 

### Backend

You will need `nodejs`, `docker` and `docker-compose`. To build and start all the required dockers:

```
yarn workspace backend docker
```
