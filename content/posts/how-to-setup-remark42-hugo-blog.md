---
title: "How to self-host Remark42 on Debian for a Hugo blog"
date: 2025-12-09
draft: false
series: "Definitive Guide to Hosting a Blog"
tags: ["Hugo", "Remark42", "Oracle"]
---
 This post documents the exact process that worked for me when self-hosting
 Remark42 behind a Cloudflare Tunnel on my Hugo blog. No theory, no dead ends, only the
 final, functional setup that I have found after hours of debugging errors and permissions.


 ------------------------------------------------------------
 1 - This is what we are going to set up:

 - Comments powered by Remark42 on your Hugo blog
 - Cloudflare Tunnel running on a Debian server + Docker
 - No public IP
 - Access via HTTPS

----

## Setup Cloudflare Tunnel
Log in as the root user and run the command below:

```
docker run --rm -it \
  --user 0:0 \
  -v /root/.cloudflared:/root/.cloudflared \
  cloudflare/cloudflared:latest login
```

A link will appear so you can log in to your Cloudflare dashboard and authorize your domain. After authorizing your domain, you should see that a `cert.pem` file was created in your `/root/.cloudflared/`

Now you will create a config.yml in the same directory: 

`/root/.cloudflared/config.yml`

config.yml:
```yml
protocol: http2

tunnel: <YOUR-TUNNEL-UID>
credentials-file: /root/.cloudflared/<YOUR-TUNNEL-UID>.json

ingress:
  - hostname: remark42.YOUR_DOMAIN.com
    service: http://remark42:8080
  - service: http_status:404
```


Then create the tunnel with this command

```
docker run --rm -it \
  --user 0:0 \
  -v /root/.cloudflared:/root/.cloudflared \
  cloudflare/cloudflared:latest tunnel create remark42
```



All you need now is to create the DNS record for your tunnel. You also can do this through your dashboard on the Cloudflare website, but I find it easier to just run the command that creates for me

```
docker run --rm \
  --user 0:0 \
  -v /root/.cloudflared:/root/.cloudflared \
  cloudflare/cloudflared:latest \
  tunnel route dns remark42 remark42.YOUR_DOMAIN.com
```

Now you have a Cloudflare tunnel running, and the DNS is pointing exactly where it needs to point, the only remaining step is to create the [Remark42](https://remark42.com/) container

----

## Setup Remark42

Copy this docker composer file and run with `docker compose up -d`

docker-compose.yml:

```yml
 remark42:
    image: umputun/remark42:latest
    user: "0:0"
    container_name: remark42
    restart: unless-stopped
    environment:
      REMARK_URL: https://remark42.YOUR_DOMAIN.com
      SITE: remark42
      SECRET: YOUR_SECRET_HERE 
      AUTH_ANON: "true" #REMOVE THIS AFTER TESTING
      AUTH_GOOGLE_CID: YOUR_GOOGLE_CLIEND_ID_HERE
      AUTH_GOOGLE_CSEC: YOUR_GOOGLE_SECRET_KEY_HERE
      AUTH_GITHUB_CID: YOUR_GITHUB_CLIENT_ID_HERE
      AUTH_GITHUB_CSEC: YOUR_GITHUB_SECRET_KEY_HERE
    volumes:
      - ./var:/srv/var
    expose:
      - "8080"

  cloudflared:
    image: cloudflare/cloudflared:latest
    user: "0:0"
    container_name: cloudflared-remark42
    restart: unless-stopped
    command: tunnel run remark42  
    volumes:
      - /root/.cloudflared:/root/.cloudflared
    depends_on:
      - remark42

```

Enter `https://remark42.YOUR_DOMAIN.com` to check if your remark42 container is up and running. Create a comment just to test if the comments are working properly, after that you can remove the ```AUTH_ANON: "true"``` on your docker container and run it again.

--- 
## Adding Remark42 in your Blog

To add remark42 on your Blog/Hugo Blog, you can follow the [Remark42 guide](https://remark42.com/docs/getting-started/installation/), or if you are using a Hugo template, check the documentation of your template, most of them have a built-in integration that you can set up through `hugo.toml`

In the Poison Theme, that I currently use, the only thing you need to do is to add this lines on your `[params]` section:

```toml
    remark42 = true
    remark42_host = "https://remark42.YOUR_DOMAIN.com"
    remark42_site_id = "remark42"
```

---

## Alternate way of creating a tunnel

If creating a tunnel through **Docker** didn't work for you, you can do the following steps:

 1 - [Install the Cloudflare Tunnel service](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/do-more-with-tunnels/local-management/as-a-service/linux/)

 2 - Log in to Cloudflare from the host (this opens a browser auth flow):

   ```cloudflared tunnel login```

 3 - Then create the tunnel:

   ```cloudflared tunnel create remark42```

 This creates a credentials file like:
   `/root/.cloudflared/<UUID>.json`


Check if your tunnel was created with
```cloudflared tunnel list```

If it worked you can proceed with the creation of the config.yml in the `/root/.cloudflared/config.yml` directory


## Conclusion 
 Remark42 is now:
 - Self-hosted
 - HTTPS
 - No open ports
 - No public IP
 - Stable

And running on your blog :)

