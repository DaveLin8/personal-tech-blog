+++
title = "How I Hosted My Hugo Blog Behind AWS EC2 and WireGuard"
date = 2026-11-16
draft = false
description = "How I deploy Hugo with Jenkins and Nginx, then serve it through an AWS Caddy proxy over WireGuard."
tags = ["Hugo", "Jenkins", "Nginx", "Caddy", "WireGuard", "AWS", "Self-Hosting"]
+++

When I set up this blog, I already had Caddy running on an AWS EC2 instance and a WireGuard tunnel back to my Ubuntu server. I used that same connection for the blog. The home server builds and serves the pages, and EC2 handles public HTTPS.

![Jenkins builds the Hugo site from GitHub and syncs the generated HTML to the home server. Browser requests pass through Caddy on EC2 and over WireGuard to Nginx, which reads the generated files.](/images/posts/hugo-blog-architecture.svg)

## From Markdown to the live site

The Hugo source lives in a GitHub repository. Posts are Markdown files under `content/posts/`, and PaperMod is checked out as a Git submodule. Jenkins checks out the repository and its submodules, runs `hugo --minify`, checks that the build produced `public/index.html`, and syncs the generated files into the directory Nginx serves.

Publishing a post starts with a commit to the Hugo repository. The Caddyfile comes into play when a hostname or upstream changes.

## Bringing up the first domain

I set up [www.davelin.io](https://www.davelin.io/) first. Its DNS record pointed to EC2. In Caddy, I added a site block that proxied to Nginx on port 8082 over WireGuard.

```caddyfile
www.davelin.io {
    reverse_proxy <home-server-wireguard-ip>:8082
}
```

The upstream address above is a placeholder for the home server's WireGuard IP. I kept the existing Open WebUI and ComfyUI routes in place while bringing up the blog domain.

## Debugging the HTTPS error

Chrome initially showed `ERR_SSL_PROTOCOL_ERROR`. The DNS lookup returned the expected IPv4 address, and the Hugo block was present in the Caddyfile. To test TLS without public DNS in the path, I ran this on EC2:

```bash
curl -v --resolve www.davelin.io:443:127.0.0.1 https://www.davelin.io/
```

The local TLS handshake failed too. That narrowed the investigation to the HTTPS listener on EC2, before any request could reach Nginx. The next checks were whether the running Caddy configuration contained the new hostname and whether Caddy's journal showed certificate activity for it. The file on disk and the configuration the process has loaded are two separate things to check.

## Switching the second domain

Once I could open the Hugo homepage on `www.davelin.io` over HTTPS, I moved [www.doubletrees.com](https://www.doubletrees.com/) from its old Open WebUI upstream to the Hugo upstream. I backed up the Caddyfile, reviewed the upstream change, validated the config, and reloaded Caddy. Afterward I confirmed that both blog domains worked and checked the dedicated hostnames for Open WebUI and ComfyUI.

![Add and test www.davelin.io, verify its Hugo homepage, then back up and validate Caddy before switching www.doubletrees.com. Check both blog domains and the AI app routes after the switch.](/images/posts/hugo-domain-rollout.svg)

Having a second hostname gave me a way to verify the new end-to-end path before switching the existing one. For the next post, the route is already in place. I only need to update the Markdown and deploy the Hugo build.
