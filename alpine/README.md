# Docker image with cron (Alpine-based)

Docker image to run cron inside the container

## Adding cron jobs

Suppose you have folder `crontabs` with your cron scripts. The only thing you have to do is copying this folder into the Docker image:

```Dockerfile
# Dockerfile

FROM sorx00/cron-base:alpine

COPY crontabs /etc/crontabs
```

Then build and create container:

```bash
docker build --tag my_cron .
docker run --detach --name cron my_cron
```

## Logs

To view logs use Docker [logs](https://docs.docker.com/engine/reference/commandline/logs/) command:

```bash
docker logs --follow cron
```

*All you cron scripts should write logs to `/var/log/cron.log`. Otherwise you won't be able to view any log using this way.*

You can set the logging level using a optional argument `-l`/`--loglevel`. Most verbose: 0, default: 8.

## Passing cron jobs by arguments

Additionally you can pass any cron job by argument(s) using custom command at the moment of container creation (providing optional user with `-u`/`--user` option):

```bash
docker run --detach --name cron sorx00/cron-base:alpine --user www-data \
    "0 1 \* \* \* echo '01:00 AM' >> /var/log/cron.log 2>&1" \
    "0 0 1 1 \* echo 'Happy New Year!!' >> /var/log/cron.log 2>&1"
```

## Passing additional entrypoint by argument

You can pass own script or another executable file to run before `crond`.

```bash
docker run sorx00/cron-base:alpine init-script.sh --loglevel 0
```

## Environ variables

Almost any environ variable you passed to the Docker will be visible to your cron scripts. With the exception of `$SHELL`, `$PATH`, `$PWD`, `$USER`, etc.

```bash
docker run --tty --rm --interactive --env MY_VAR=foo sorx00/cron-base:alpine \
    "\* \* \* \* \* env >> /var/log/cron.log 2>&1"
```

## Special Environ variables

### `CRONTABS_DIR`

This Environ variable let you provide custom `crontabs` directory location which will be used instead of default one (`/etc/crontabs`):

```bash
docker run --detach --name cron --env CRONTABS_DIR=/etc/my_app/crontabs sorx00/cron-base:alpine
```

This may be very useful when you create more then one Docker container from a single image with different cron jobs per container.
