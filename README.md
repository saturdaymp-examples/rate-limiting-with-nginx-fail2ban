# BSides Edmonton 2022 - Rate Limiting Using Nginx and Fail2Ban

An demonstration of how to set up rate limiting using [nginx](https://nginx.org/) and [Fail2Ban](http://www.fail2ban.org).  This repo has the Docker container used for demo and the slides.  The Docker container should NOT be used for production.  It is setup for the demo and to mimic setting up rate limiting on a server.

## Setup

To run the Docker container and follow along during the demo follow the steps below.  This demo uses a [nginx Docker](https://hub.docker.com/_/nginx) container that has some customization such as installing Fail2Ban, iptables, and configuring the Docker network corretly.

1) Install [Docker](https://docs.docker.com/get-docker/).

2) Build the container:

    ```
    docker compose build demo-start
    ```

3) Run the container and login into it:

    ```
    docker compose run --rm --service-ports demo-start bash
    ```

4) Start nginx.  Do this from inside the Docker container.

    ```
    nginx
    ```

5) Open up a browser on your host machine and navigate to [http://localhost:8080](http://localhost:8080).  You should see the nginx welcome page.

You will need to restart nginx and Fail2Ban if you make changes to their config files.  If you stop the container you will lose all your work.  If you want to keep your work run the container without the `--rm` flag.

The completed demo, with everything enabled, is in the `demo-end` container instead of `demo-start`.  Completeded configuration files can be found at:

- nginx config: [nginx/conf.d/default.conf](nginx/conf.d/default.conf)
- Fail2Ban jail: [fail2ban.jail.d/nginx.conf](fail2ban.jail.d/nginx.conf)

Additional informaton about setting up rate limiting in nginx and Fail2Ban can be found at:

- Nginx rate limiting: https://www.nginx.com/blog/rate-limiting-nginx/
- Fail2Ban: https://rtcamp.com/tutorials/nginx/fail2ban/

## Useful Commands

### nginx: start, stop, check config, and reload config

```
nginx
```

```
nginx -s stop
```

```
nginx -t
```

```
nginx -s reload
```

### Fail2Ban: start, stop, view jail status, and unban an IP

```
fail2ban-client start
```

```
fail2ban-client stop
```

```
fail2ban-client status
```

```
fail2ban-client status ngxin-limit-req
```

```
fail2ban-client set nginx-limit-req unbanip 172.22.0.1
```

### iptables: view current rules

```
iptables -L
```