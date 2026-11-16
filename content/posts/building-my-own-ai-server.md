+++
title = "Building My Own AI Server: Qwen to ComfyUI"
date = 2026-11-16
draft = false
description = "How I went from running Qwen locally to hosting chat and video generation on an Ubuntu server behind WireGuard and Caddy."
tags = ["Ubuntu", "Qwen", "Ollama", "ComfyUI", "WireGuard", "AWS", "Self-Hosting"]
+++

I started with a simple goal: get Qwen running on my own machine so I could try a local language model. By the end of the project, the same Ubuntu server was running a browser-based chat interface and a separate video-generation app. An AWS EC2 instance gave both services a public HTTPS address, while the model workloads stayed at home.

## The Ubuntu server

I run the AI workloads on `dave-bigdell`, my Dell XPS 8950 home server. It runs Ollama for Qwen, Open WebUI for chat, and ComfyUI for Wan 2.2 video experiments. The AWS instance handles public HTTPS through Caddy. Model inference runs on the Dell.

![Front view of a Dell XPS 8950 desktop tower](/images/posts/dell-xps-8950.png)

*Dell XPS 8950. [Stock product photo by Dell](https://www.dell.com/support/product-details/en-us/product/xps-8950-desktop/overview).*

| Component | Setup |
| --- | --- |
| System | Dell XPS 8950 |
| CPU | 12th Gen Intel Core i7-12700 |
| Operating system | Ubuntu 26.04 |
| GPU | NVIDIA GeForce RTX 3060 Ti with 8 GiB of VRAM |
| System memory | About 30 GiB visible to Ubuntu |
| Storage | 468 GiB NVMe system partition, plus a separate 1 TB drive reserved for storage and backups |

![A browser reaches Caddy over HTTPS. Caddy routes chat and video requests over WireGuard to Open WebUI and ComfyUI on the Ubuntu server. Open WebUI sends chat requests to Ollama, which runs Qwen.](/images/posts/ai-server-architecture.svg)

## Qwen, Ollama, and Open WebUI

Ollama runs the Qwen model. Open WebUI gives me a chat interface in the browser and sends requests to Ollama. Having those pieces separate was helpful for troubleshooting. I could check whether the model responded locally before looking at the web interface or the public connection.

I kept Open WebUI on its own hostname, [ai.doubletrees.com](https://ai.doubletrees.com/). That gave me a stable address for chat while I added other services to the same server.

## Adding video generation

I added ComfyUI as a separate application for experimenting with Wan 2.2 video workflows. In ComfyUI, the workflow is a graph. The model, prompt, sampling, and output are visible as connected steps, so I can change one part of a workflow and see what happens.

Chat and video generation are separate workloads. They share the Ubuntu host, but each app has its own upstream port and hostname. ComfyUI is at [video.doubletrees.com](https://video.doubletrees.com/).

## Connecting the home server to AWS

Caddy runs on EC2 and handles public HTTPS. A WireGuard tunnel connects EC2 to the home server, so Caddy reaches each app at the home server's tunnel address. A browser request goes to Caddy first, then across WireGuard to the requested app. Inference runs on the Ubuntu machine.

One useful test was requesting ComfyUI's `/system_stats` endpoint from EC2 over the tunnel. It returned HTTP 200. That confirmed the upstream was reachable from the machine running Caddy and gave me a concrete way to test the network path independently of public DNS or HTTPS.

## What I'd repeat

I'd bring up each app locally, test it over WireGuard from EC2, and then test the public hostname. Each step narrows down where to look when something fails. A browser error at the public address can come from the proxy or TLS even when the model and the app are both running fine.

![Test each app locally first, then from EC2 over WireGuard, and finally in a browser using its public HTTPS URL.](/images/posts/ai-server-testing.svg)

I reused that same network path when I added this Hugo blog. The [next post](/posts/hugo-behind-aws-ec2-and-wireguard/) walks through how I brought up the new domain and debugged its first HTTPS failure.
