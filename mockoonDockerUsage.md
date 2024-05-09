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

That's it, you can query your mocked API on port `3000`.

### Running in the same `docker` network
TDB

PS: the best feature of that whole thing for me was the UI-way to design your API(s), save the result to json and them mock-deploy it using `mockoon`
