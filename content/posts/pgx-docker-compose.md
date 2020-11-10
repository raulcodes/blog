+++
title = "Settin up Go's pgx package with docker-compose"
date = "2020-10-16"
author = "Raul Camacho"
authorTwitter = "_raulcodes" #do not include @
cover = ""
tags = ["development", "go"]
keywords = ["golang", "go", "docker", "postgres", "pgx", "docker-compose"]
description = "A simple docker-compose set up to get you up and running with Go's pgx package"
showFullContent = false
+++

Go's [pgx package](https://github.com/jackc/pgx) goes above being a postgres driver and actually implements its own interface that boasts better performance than the standard `database/sql` package. That, along with it being a well-maintained and actively developed project, make it an attractive option for using postgres in your next Go project. This guide will hopefully help you get up and running with `pgx` and docker-compose. 

## 1. Go Project

Create a new Go project according to `pgx`'s [Getting Started guide](https://github.com/jackc/pgx/wiki/Getting-started-with-pgx) . Once you have a working Go program with a local install of postgres, make sure to stop your local postgres instance, to avoid any possible conflicts later. With postgres installed via brew on a mac, I ran 
```bash
brew services stop postgresql
```

### 1a. Create Postgres volume

In your terminal, create a directory that the docker postgres service will use as a data volume.
```bash
mkdir -p /path/to/postgres/database-data
```

## 2. Go Dockerfile

At the root of your new Go project, Add a `Dockerfile` file with the below contents:
```docker
FROM golang:1.14-alpine AS build

WORKDIR /src/
COPY main.go go.* /src/
RUN CGO_ENABLED=0 go build -o /bin/hello-world

FROM scratch
COPY --from=build /bin/hello-world /bin/hello-world
ENTRYPOINT ["/bin/hello-world"]
```

This is a pretty generic dockerfile that you might see variants of in other Go projects that use Docker. It pulls the golang docker image, `version 1.14-alpine`, builds your app, and moves the built binary into a `bin/` directory, readying it for us to run.

### 2a. Verify Dockerfile

With your dockerfile in place, verify it works by building your app with:
```bash
docker build
```
in your terminal.

## 3. docker-compose

At the root of your project, create a file called `docker-compose.yml` and paste in the following:

```yaml
version: '3'
services:
  app:
    build: .
    depends_on:
      - db
    environment: 
      DATABASE_URL: "postgres://postgres:postgres@db:5432/postgres"
  db:
    image: "postgres" 
    volumes:
      - /path/to/postgres/database-data:/var/lib/postgresql/data/

volumes:
  database-data: 
```

We are declaring that we have two services: `app` and `db`. 

### `db` service

```yaml
  db:
    image: "postgres" 
    volumes:
      - ~/Documents/database-data:/var/lib/postgresql/data/
```

The `db` service pulls the latest postgres docker image, and declares a single volume to store data in. 

### `app` service

```yaml
  app:
    build: .
    depends_on:
      - db
    environment: 
      DATABASE_URL: "postgres://postgres:postgres@db:5432/postgres"
```

The `app` service defined uses our dockerfile that we added in step 2 (`build .` defines where docker-compose should look for a dockerfile). We are also saying that this service `dependes_on` the `db` service.

### 4. First run

Run:
```bash
docker-compose build
```
in your terminal and watch docker-compose pull images and build your services.

With your services built, run:
```bash
docker-compose up
```
You should hopefully see logs being outputted in your terminal, with different prefixes for `app` and `db`, depending on which service the log is from. Unfortunately, you will probably see a database connection error coming from `app`. One caveat of docker-compose is that while the `app` service may depend on the `db` service, this dependency is at a container level. So there is a good chance that postgres may not be running when your go program tries connecting to it, since the idea of postgres being 'ready' at an application level isn't clear. Read on to see how to fix this.


### 5. Waiting for Postgres

Docker compose's own [documentation](https://docs.docker.com/compose/startup-order/) encourages developers to add logic for waiting for services at the application level, instead of relying on scripts like [`wait-for-it.sh`](https://github.com/vishnubob/wait-for-it) (which is great btw). Below, I've adapted a [`wait-for-it.sh` port for go](https://github.com/alioygur/wait-for) to wait for postgres to be running and accepting connections on the default port of 5432. 


```go
func WaitForPostgres(service string, timeOut time.Duration) error {
	var pgChan = make(chan struct{})
	var wg sync.WaitGroup
	wg.Add(1)
	go func() {
			go func(s string) {
				defer wg.Done()
				for {
					_, err := net.Dial("tcp", service)
					if err == nil {
						return
					}
					time.Sleep(1 * time.Second)
				}
			}(service)
		wg.Wait()
		close(pgChan)
	}()

	select {
	case <-pgChan:
		return nil
	case <-time.After(timeOut):
		return fmt.Errorf("postgres isn't ready in %s", timeOut)
	}
}
```

Using this function before connecting to postgres with `pgx` makes our go program wait for postgres. Try running 
```bash
docker-compose up
```
again and watch your `app` service log "Hello world"!

Thanks for reading.