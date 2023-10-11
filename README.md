# Rate Limiting Using Nginx and Fail2Ban

A demonstration of how to set up rate limiting using [nginx](https://nginx.org/) and [Fail2Ban](http://www.fail2ban.org) that I orginally presented at [BSides Edmonton](https://www.bsidesedmonton.ca/) in 2022.  A recorded version of the demo can be found [here](https://youtu.be/gR4w9trH9pA).

This repo has the Docker container used for demo and the slides.  The Docker container should NOT be used for production as it is setup for the demo and to mimic setting up rate limiting on a server.

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

The completed demo, with everything enabled, is in the `demo-end` container instead of `demo-start`.  Completed configuration files can be found at:

- nginx config: [nginx/conf.d/default.conf](nginx/conf.d/default.conf)
- Fail2Ban jail: [fail2ban/jail.d/nginx-limit-req.conf](fail2ban/jail.d/nginx-limit-req.conf)

Additional informaton about setting up rate limiting in nginx and Fail2Ban can be found at:

- Nginx rate limiting: https://www.nginx.com/blog/rate-limiting-nginx/
- Fail2Ban: https://rtcamp.com/tutorials/nginx/fail2ban/

A video of the demonstation:

https://youtu.be/gR4w9trH9pA

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
fail2ban-client status nginx-limit-req
```

```
fail2ban-client set nginx-limit-req unbanip 172.22.0.1
```

### Send a burst of requests

You can send a bunch of requests by running the [send-requests.sh](send-requests.sh) file.  You will have to make it executable first via `chmod +x send-requests.sh`.  The script will send 10 curl requests by default but you can tweak the number as needed.

```
./send-requests.sh
```

Without Fail2Ban enabled all requests should return a 200:

```
HTTP/1.1 200 OK
Server: nginx/1.25.2
Date: Sun, 24 Sep 2023 16:40:14 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 15 Aug 2023 17:03:04 GMT
Connection: keep-alive
ETag: "64dbafc8-267"
Accept-Ranges: bytes

HTTP/1.1 200 OK
Server: nginx/1.25.2
Date: Sun, 24 Sep 2023 16:40:14 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 15 Aug 2023 17:03:04 GMT
Connection: keep-alive
ETag: "64dbafc8-267"
Accept-Ranges: bytes

<... More requests ...>
```

With nginx rate limiting enabled the requests will start timing out with 503 errors:

```
HTTP/1.1 200 OK
Server: nginx/1.25.2
Date: Sun, 24 Sep 2023 16:52:30 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 15 Aug 2023 17:03:04 GMT
Connection: keep-alive
ETag: "64dbafc8-267"
Accept-Ranges: bytes

HTTP/1.1 503 Service Temporarily Unavailable
Server: nginx/1.25.2
Date: Sun, 24 Sep 2023 16:52:30 GMT
Content-Type: text/html
Content-Length: 497
Connection: keep-alive
ETag: "64dbafc8-1f1"

<... More requests ...>
```

Once Fail2Ban has blocked the IP address curl will return "Empty reply from server":

```
curl: (52) Empty reply from server

<... More requests ...>
```

### iptables: view current rules

```
iptables -L
```

### Log Files

- nginx: `/var/log/nginx/access.log`
- nginx: `/var/log/nginx/error.log`
- fail2ban: `/var/log/fail2ban.log`

## Acknowledgements
Thank you to [BSides Edmonton](https://www.bsidesedmonton.ca/) for allowing me to present.  Also thank you to the [Corgibytes](https://corgibytes.com/) client that inspired this presentation.
