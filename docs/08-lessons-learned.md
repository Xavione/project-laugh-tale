# Lessons Learned

## 1. Validate from the bottom up

When a service is unreachable, start with physical/link state, then MAC learning, IP routing/reachability, and finally the application. This prevented me from treating every failure as a Proxmox problem.

## 2. A cluster is more than a collection of servers

The Sanji recovery demonstrated that cluster membership is distributed state. Quorum, votes, Corosync membership, authentication, locks, and configuration synchronization all influence whether a change is safe.

## 3. Avoid changing multiple variables at once

Moving Zoro and its cable to a known-good switch port helped isolate the fault domain. That is stronger troubleshooting than randomly changing host, switch, and network configuration together.

## 4. Do not invent a root cause

Port 3 eventually worked after inspection/link cycling, but the exact original cause was not conclusively proven. The documentation therefore records what was observed and validated without pretending certainty.

## 5. Documentation is part of the build

A working homelab proves that something worked once. A runbook proves I can explain, repeat, troubleshoot, and hand off the process.

## 6. The GUI is an interface, not the architecture

Proxmox made cluster creation easy, but troubleshooting required understanding `pvecm`, `pmxcfs`, Corosync, Linux services, quorum, and network behavior beneath the GUI.

## 7. Build complexity in layers

The environment is intentionally starting with a simple Layer 2 network and three hypervisors. Kubernetes, VLANs, observability, automation, storage, and local AI will be added as independent layers with their own validation and documentation.
