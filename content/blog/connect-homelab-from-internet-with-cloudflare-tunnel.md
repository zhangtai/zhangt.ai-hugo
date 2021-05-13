---
title: Connect Homelab From Internet with Cloudflare Tunnel
date: 2021-05-13T21:20:48+08:00
---

Cloudflare had a [recent announcement](https://blog.cloudflare.com/tunnel-for-everyone/) that everyone can use the free Tunnel now, you can access a local service from the internet with your own domain, no public IP address is required. It's a good news for me since in my area ISP don't open 443/80 port for my public dynamic IP, every time when I want to expose a service I need to manual port forwarding in my OpnSense server and config the TLS cert for https, and I need to remember/bookmark lots of ports of my domain...

The steps are quick easy to follow by [this guide][1]. After the initial, you'd better install the [cloudflared service][2] which will be easier for manage the process and keep it running at the background. I just want to share my config for allowing multiple hosts and services been shared with single tunnel, with multiple domains.

I assume you have already installed the service, edit the config file similar to this:

`/etc/cloudflared/config.yml`

```yaml
tunnel: abcd1234-1234-5678-9012-1a2b3c4d5e
credentials-file: /var/cloudflared_secrets/abcd1234-1234-5678-9012-1a2b3c4d5e.json

ingress:
  - hostname: gke.example.com
    service: http://192.168.3.30
    noTLSVerify: true
  - hostname: ide.example.com
    service: http://localhost:12345
  - service: http_status:404
```

In you Cloudflare's domain DNS settings, set both **gke.example.com** and **ide.example.com** CNAME to the same tunnel address, e.g. `abcd1234-1234-5678-9012-1a2b3c4d5e.cfargotunnel.com`. Restart the service and check the status, if no issue you should see you side alive.

One disadvantage for me is that the tunnel is a bit slow compared to the port forwarding, I guess it's due to my location 🥲.

[1]: https://developers.cloudflare.com/cloudflare-one/tutorials/share-new-site
[2]: https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/run-tunnel/run-as-service#macos
