# How to use `mockoon` as a service (powered by `docker`)

In the era of (in)famous interconnected services, we often want to build a fast prototype. So, the need for mocking some APIs is there. 
Especially, when we develop something in a local dev environment and then need to properly deploy it up to some cloud.
The pain of providing all the required accesses/tokens/auth is there, so mocking can help us paving our pathway to a somehow-working-prototype faster.

## `mockoon` tool
This is `mockoon`: https://mockoon.com/

I found this tool (both CLI and UI) pretty awesome. I am not any kind of power user but it gives an incredible power to design your APIs 
and test them through.

Yes, `postman` does provide simimlar functionality and maybe even more than that; however, `mockoon` is a more lightweight alternative.
Additionally, they are open-source and the CLI is just awesome.

One can use CLI just as-is (in a local environment) however I am pretty sure many developers would like to containerize it to isolate and 
ease the future pain of deployment (for example, as a part of test stage on your ci/cd).

### `mockoon` official `docker` image
The official docker image resides here: https://hub.docker.com/r/mockoon/cli
So, given the `mockoon_config.json` ("environment" configuration in `mockoon` terms) in the current folder, you can run it as follows:
```
docker run --mount type=bind,source=./mockoon_config.json,target=/data,readonly -p 3000:3000 mockoon/cli:latest --data data --port 3000
```
(add `-d if you want detached mode)

One can also `docker compose` for it:
```
version: '3.8'
services:
  mockoon:
    image: mockoon/cli:latest
    command: --data data --port 3000
    volumes:
      - type: bind
        source: ./mockoon_config.json
        target: /data
        read_only: true
    ports:
      - "3000:3000"
```

That's it, you can query your mocked API on `localhost:3000` (change the port in `command` and in `ports` according to your needs).

IMPORTANT note: sometimes you can find that this `docker compose` does not really work. In that case check the access rights to `mockoon_config.json`.
**Insecure** alternative:
```
version: '3.8'
services:
  mockoon:
    image: mockoon/cli:8.1.1
    user: root
    command: --data data --port 3000
    volumes:
      - type: bind
        source: ./ocr-service-v3-mock.json
        target: /data
        read_only: true
    ports:
      - "3000:3000"
```

### Running in the same `docker` network
If you want to interconnect several services, you can put them into one `docker-compose.yml` file (so that the `docker` creates a new network on its own). Or you can create a new network yourself using `docker network create` and then attach the services to it.

For example like this:
```
docker network create testnetwork
```

We can list all the `docker` networks like this:
```
$ docker network ls

NETWORK ID     NAME          DRIVER    SCOPE
8a77a2ad75b8   bridge        bridge    local
d19f6dd1841f   host          host      local
85e9272c56cd   none          null      local
bf781fb92d59   testnetwork   bridge    local
```

If we want `mockoon` to run in the same `docker` network as other containers:
```
version: '3.8'
services:
  mockoon:
    image: mockoon/cli:8.1.1
    user: root
    command: --data data --port 8083
    volumes:
      - type: bind
        source: ./ocr-service-v3-mock.json
        target: /data
        read_only: true
    ports:
      - "8083:8083"
    networks:
      - testnetwork

networks:
  testnetwork:
    external: true
```

PS: the best feature of that whole thing for me was the UI-way to design your API(s), save the result to json and them mock-deploy it using `mockoon`
