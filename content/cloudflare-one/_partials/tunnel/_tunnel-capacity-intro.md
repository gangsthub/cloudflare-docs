---
_build:
  publishResources: false
  render: never
  list: never
---

Unlike legacy VPNs where throughput is determined by the server's memory, CPU and other hardware specifications, Cloudflare Tunnel throughput is primarily limited by the number of ports configured in system software. Therefore, when sizing your `cloudflared` server, the most important element is sizing the available ports on the machine to reflect the expected throughput of TCP and UDP traffic.