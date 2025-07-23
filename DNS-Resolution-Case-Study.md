Case Study: Solving a Complex DNS Resolution Issue in a Homelab Environment
1. Problem Statement
The primary goal was to embed live-rendering monitoring panels from a self-hosted Grafana instance into a public GitHub project README. This would serve as a dynamic, real-time portfolio piece showcasing server administration skills.

Initial Symptom: Despite being accessible from the public internet, the embedded Grafana panels appeared as broken image icons on the public GitHub page when viewed from a client machine on the local network.

2. Troubleshooting Methodology & Narrative
The troubleshooting process followed a logical, layered approach, isolating the problem from the public internet down to the individual client machine.

Step A: Initial Verification (Public Accessibility)

The first step was to determine if the Grafana instance was reachable from the public internet.

Test: The Grafana panel URL was accessed from a smartphone using cellular data (bypassing the local network).

Result: The Grafana login page appeared.

Conclusion: This confirmed that basic networking was correct: DNS was resolving to the public IP, and port forwarding on the router was working. The issue was not simple connectivity but likely related to permissions or how the content was being served.

Step B: Diagnosing Anonymous Access

The login page indicated that an anonymous user (like GitHub's image proxy) could not access the panel.

Action: Enabled anonymous access in the Grafana docker-compose.yml configuration (GF_AUTH_ANONYMOUS_ENABLED=true).

Result: The login page disappeared, but now a "Dashboard not found" error appeared.

Conclusion: This led to the discovery of a NAT Loopback issue. While the public could reach the server, clients inside the network were being blocked by the router when trying to access a local server via its public IP address. The browser was also likely blocking the connection due to an SSL certificate mismatch (connecting to a local IP but receiving a cert for a public domain).

Step C: Implementing Split-Brain DNS with Pi-hole

The professional solution to NAT Loopback is to provide a different DNS response for local clients.

Action: Deployed a Pi-hole container using Docker to act as a local DNS server.

Configuration: Added a "Local DNS Record" to Pi-hole, mapping grafana.infernalaquatics.com to the server's local IP address (e.g., 192.168.1.100).

Network Update: The LAN's DHCP server (on the Spectrum router) was updated to use the Pi-hole's IP as the primary DNS server.

Step D: The Root Cause Analysis (Client-Side DNS)

Even after setting up Pi-hole, a ping test from the primary client (a MacBook) still resolved to the public IP address, not the local one.

Test 1: dig command. A dig command executed directly against the Pi-hole container (docker exec pihole dig ...) confirmed that Pi-hole itself was working correctly. This definitively isolated the problem to the client's DNS resolution path.

Test 2: scutil --dns on macOS. This command revealed the client was still using the router's IP for DNS, despite DHCP settings being updated. This pointed to a failure in the DHCP handshake or a client-side override.

Step E: Final Resolution

Since the router's DHCP service was unreliable, the fix had to be applied directly to the client.

Action: The Pi-hole server's IP address (192.168.1.27) was manually added to the list of DNS servers in the MacBook's network settings.

Result: After flushing the local DNS cache, ping immediately resolved to the correct local IP. The browser was able to connect without an SSL certificate mismatch, and the Grafana panels rendered correctly on GitHub.

3. Skills & Technologies Demonstrated
This real-world problem served as a practical validation of skills across multiple IT domains:

Network Troubleshooting: Systematically diagnosing a complex issue from the application layer down to the network infrastructure and client configuration.

DNS & DHCP Administration: Deep understanding of DNS resolution, split-brain DNS architecture, and DHCP server configuration.

Docker & Containerization: Deploying and managing multiple networked services (Grafana, Prometheus, Pi-hole) using Docker Compose.

Linux Systems Administration: Navigating the command line, editing configuration files, and managing services (systemd, virsh).

Reverse Proxy & Web Server Management: Configuring Nginx to handle public traffic, proxy requests, and manage SSL certificates.

Documentation: Clearly articulating a complex technical problem and the steps taken to resolve it.
