# Case Study: Systemic Server Failure & Resolution

* Author: Christopher Clendening (Netherwarlord)
* Date: July 28, 2025
* Tags: Nginx, SSL, Docker, LetsEncrypt, Linux, Networking, DNS, NAT, Penpot, Mailcow, System Administration, Troubleshooting

1. Executive Summary

This document details the end-to-end troubleshooting process for a critical, multi-layered infrastructure failure on a bare-metal home server. The incident began as a simple failed SSL certificate renewal and cascaded through the entire stack, revealing: systemic Nginx configuration errors, missing symbolic links, a network-level NAT loopback problem, persistent application-level backend failures in a Penpot stack, and fundamental conflicts between a new Mailcow stack and the host server's core DNS and networking services. After numerous, increasingly complex fixes failed to restore stability and instead introduced new, unpredictable errors, the ultimate root cause was determined to be a corrupted and unstable state of the host operating system's networking and containerization layers. The final resolution was the strategic decision to perform a full backup and execute a "bare-metal nuke"—a complete reinstallation of the operating system and a structured, methodical redeployment of all services.

2. The Troubleshooting Journey: A Cascade of Failures

The investigation followed a logical but frustrating progression, with each fix revealing a deeper, more complex problem.

Layer 1: Nginx Configuration: The initial investigation, prompted by a failed nginx -t command, uncovered a systemic misconfiguration across multiple site files. The root cause was a copy-paste error where several Nginx server blocks were referencing another service's SSL certificate. This was corrected by systematically editing the invalid files.

Layer 2: Docker Networking & DNS: Even after correcting Nginx, services were unreachable. This was traced to a combination of a missing symbolic link for a site in /etc/nginx/sites-enabled/ and a classic NAT loopback problem. The use of ss and openssl proved that the Nginx service was running correctly, isolating the fault to the network. This was initially resolved by implementing a split-brain DNS configuration using a Pi-hole container.

Layer 3: Application-Specific Failures (Penpot): With the network seemingly stable, the Penpot application still failed with a 502 Bad Gateway error. A deep dive into the container logs (docker logs penpot-backend) revealed a series of internal application errors, including a typo in the database connection string, an incorrect public URI, and a suspected bug in the :latest Docker image tag. Each fix led to a new, different crash.

Layer 4: Inter-Service Conflicts (Mailcow): To resolve Penpot's final dependency on an SMTP server, a production Mailcow stack was deployed. This introduced a host of new, severe conflicts. The nginx-mailcow container fought for ports with the host Nginx, and the unbound-mailcow container was caught in a crash loop due to a conflict with the host's systemd-resolved DNS service.

Layer 5: Host System Instability: Attempts to resolve the DNS conflict by modifying the host's /etc/resolv.conf file proved catastrophic, breaking the previously stable split-brain DNS setup and rendering the entire local network unreliable. Even after reverting the changes, the system remained in an unpredictable state. "Ghost" docker-proxy processes were found to be holding network ports hostage, a situation that even a full Docker daemon restart and a host reboot could not reliably clear.

3. Root Cause Analysis & Final Resolution

The cascading nature of the failures, where each fix led to a new and often unrelated problem, pointed to a single, overarching root cause: the host operating system's state had become unpredictably corrupt. The repeated, forceful restarts of services, manual network configuration changes, and conflicting container setups had damaged the stability of the Docker networking layer and the host's DNS resolution to a point where it could no longer be trusted.

Final Resolution: Bare-Metal Rebuild

The decision was made to execute a controlled "nuclear option." Continued attempts to patch the unstable system were deemed inefficient and unreliable. The only way to guarantee a return to a known-good state was to start fresh. The resolution was to back up all essential data and follow a structured plan to:

Reinstall a minimal Debian 12 server OS.

Configure the system from the ground up with industry-standard tools (Docker Compose V2, UFW).

Methodically redeploy the core services (Nginx Proxy Manager, Portainer, Pi-hole) one by one on a clean, stable foundation.

4. Conclusion & Key Takeaways

Recognize the Point of No Return: While methodical troubleshooting is critical, it's equally important to recognize when a system's state has become so unstable that patching is no longer a viable option. The time and effort spent fighting unpredictable behavior can quickly outweigh the cost of a clean rebuild.

Infrastructure as Code: This incident highlights the immense value of defining infrastructure in code (e.g., using Docker Compose files). A full rebuild was a viable option precisely because the services could be redeployed quickly and consistently from their configuration files.

Minimalism Reduces Conflict: The initial conflicts (especially with systemd-resolved) were caused by default services running on the host OS. Starting with a minimal server installation reduces the potential attack surface and eliminates sources of service conflict.

The Reboot is a Valid Tool, But Not a Panacea: While a reboot can clear many transient issues, it cannot fix underlying configuration corruption. When a reboot fails to restore stability, it is a strong indicator of a deeper problem.

Documentation is a Recovery Tool: The detailed, step-by-step record of this troubleshooting process (the case study itself) became the foundation for the successful rebuild plan, ensuring that past mistakes were not repeated.

