<div align="center">

# Case Study: DNS Resolution in a Homelab Environment
A deep-dive into diagnosing and resolving a complex, multi-layered networking issue.
</div>

1. Problem Statement & Initial Symptom
🎯 Goal: To embed live-rendering monitoring panels from a self-hosted Grafana instance into a public GitHub project README, serving as a dynamic portfolio piece.

** Symptom:** Despite being accessible from the public internet, the embedded Grafana panels appeared as broken image icons on the public GitHub page when viewed from a client machine on the local network.

2. Troubleshooting Methodology & Narrative
The process followed a logical, layered approach, isolating the problem from the public internet down to the individual client machine. Each step narrowed the scope of the problem until the root cause was identified.

Step A: 🧪 Initial Verification (Public Accessibility)
The first step was to determine if the Grafana instance was reachable from the public internet at all.

Test: The Grafana panel URL was accessed from a smartphone using cellular data, completely bypassing the local network.

Result: The Grafana login page appeared.

Conclusion: ✅ Basic networking was correct. Public DNS, port forwarding, and the Nginx reverse proxy were all functioning. The issue was not simple connectivity but likely related to permissions or a local network conflict.

Step B: 🕵️ Diagnosing Anonymous Access & NAT Loopback
The login page indicated that an anonymous user (like GitHub's image proxy) could not access the panel.

Action: Enabled anonymous access in the Grafana docker-compose.yml configuration.

Result: The login page disappeared, but now a "Dashboard not found" error appeared when accessed locally via the public domain.

Conclusion: 💡 This revealed a classic NAT Loopback issue, compounded by an SSL certificate mismatch. The client inside the network was being blocked by the router when trying to access a local server via its public IP.

Step C: 🌐 Implementing Split-Brain DNS with Pi-hole
The professional solution to NAT Loopback is to provide a different DNS response for local clients.

Action: Deployed a Pi-hole container using Docker to act as a local DNS server.

Configuration: Added a "Local DNS Record" to Pi-hole, mapping grafana.infernalaquatics.com to the server's local IP address.

Network Update: The LAN's DHCP server (on the Spectrum router) was updated to use the Pi-hole's IP as the primary DNS server.

Step D: 🔬 The Root Cause Analysis (Client-Side DNS)
Even after setting up Pi-hole, a ping test from the primary client (a MacBook) still resolved to the public IP address, not the local one.

Test 1 (dig): A dig command executed directly against the Pi-hole container confirmed that Pi-hole itself was working correctly. This definitively isolated the problem to the client's DNS resolution path.

Test 2 (scutil --dns): This macOS command revealed the client was still using the router's IP for DNS, despite the DHCP settings being updated.

Conclusion: ❌ The ISP-provided router was failing to properly assign the custom DNS server via DHCP, or the client was ignoring it.

Step E: ✅ Final Resolution (Client-Side)
Since the router's DHCP service was unreliable, the fix had to be applied directly to the client.

Action: The Pi-hole server's IP address was manually added to the list of DNS servers in the MacBook's and Windows machine's network settings.

Result: After flushing the local DNS cache, ping immediately resolved to the correct local IP. The browser was able to connect without an SSL certificate mismatch.

Step F: 🏁 Final Hurdle & Pragmatic Solution (GitHub's Camo Proxy)
After resolving all local networking issues, the Grafana panels were confirmed to be rendering correctly when accessed directly via their public URLs. However, they still appeared as broken images on the GitHub README.

Symptom: Direct URL access to rendered panels worked, but embedded images on GitHub did not.

Root Cause: The issue was isolated to GitHub's image proxy, Camo. Camo has strict, non-public requirements for image headers, timeouts, and content types. Even with a dedicated rendering service, the output from the self-hosted Grafana instance was being rejected by the proxy.

Resolution: Recognizing that debugging an external, undocumented proxy was not feasible, a pragmatic decision was made to pivot the solution. Instead of live-embedded panels, a high-quality, up-to-date static screenshot of the fully functional Grafana dashboard was used.

Outcome: This approach successfully achieves the original goal of showcasing the monitoring dashboard's capabilities and design, while ensuring 100% reliability and demonstrating the ability to adapt solutions to work around external constraints.

3. Skills & Technologies Demonstrated
This real-world problem served as a practical validation of skills across multiple IT domains:

Skill Domain

Technologies & Concepts Applied

Network Troubleshooting

OSI Model (Layers 3, 4, 7), ping, dig, scutil, Packet Path Analysis

DNS & DHCP Administration

Split-Brain DNS, NAT Loopback, DNS Caching, DHCP Lease & DNS Assignment

Docker & Containerization

Docker Compose, Volume Mapping, Networking, Service Deployment, Plugins

Linux Systems Administration

systemd, virsh, nano, Command Line, File Permissions

Reverse Proxy Management

Nginx, SSL/TLS Termination (Certbot), Server Blocks, Proxy Headers

Technical Documentation

Articulating a complex problem and its resolution in a clear format
