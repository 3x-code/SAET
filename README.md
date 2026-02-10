# Designing a Stable Adaptive Encrypted Tunnel Between Inside-Iran and Outside-Iran Servers

## Abstract

This article presents a full technical design for building a **custom, highly stable, and secure tunnel** between servers located inside Iran and servers outside Iran. The goal is to overcome real-world issues such as **Deep Packet Inspection (DPI)**, **packet shaping**, **active probing**, **connection resets**, and **route instability**, which cause most traditional VPNs and packet-based tunnels to fail.

Rather than relying on classic VPN architectures, this design introduces an **Adaptive Encrypted Transport Tunnel (AETT)** — a system that focuses on *behavioral camouflage*, *adaptive networking*, and *non-signature-based encryption*.

This document is written for engineers and developers who want full control over their tunnel stack and are willing to implement a protocol from the ground up.

---

## 1. Problem Statement

Most existing tunneling solutions (OpenVPN, IPsec, WireGuard, V2Ray-like stacks) fail under restrictive network environments because:

* They use **known protocol fingerprints** (TLS, DTLS, IKE, ESP)
* Their **handshake patterns** are detectable
* Packet sizes and timing are predictable
* They rely on a **single transport path**
* They assume a relatively neutral network

In environments with advanced censorship and traffic manipulation, the network actively:

* Identifies encrypted tunnels by behavior, not just headers
* Injects TCP RST packets
* Drops or delays UDP selectively
* Applies QoS throttling after classification

Therefore, stability cannot be achieved by encryption alone.

---

## 2. Core Design Philosophy

The tunnel must:

1. Avoid all known protocol signatures
2. Look statistically similar to **legitimate high-volume traffic**
3. Adapt its behavior in real time
4. Survive partial packet loss and forced resets
5. Rotate cryptographic material frequently

Instead of asking *"How do we encrypt traffic?"*, we ask:

> **"How do real applications behave on this network, and how can we look identical?"**

---

## 3. High-Level Architecture

```
[ Client (Inside Iran) ]
        |
        |  Adaptive Encrypted Transport
        |
[ Relay / Server (Outside Iran) ]
        |
        |  Local forwarding / VPN / Proxy
        |
[ Internet / Target Services ]
```

The tunnel itself is **transport-agnostic** and only responsible for delivering a secure byte stream.

---

## 4. Transport Layer Strategy

### 4.1 Why TCP Alone Fails

* Predictable congestion control
* Easy RST injection
* Head-of-line blocking

### 4.2 Why Raw UDP Alone Fails

* Easy to drop or throttle
* Many ISPs apply rate limits

### 4.3 Hybrid Transport Model (Recommended)

* **Primary:** UDP-based custom transport
* **Fallback:** Stealth TCP mode
* **Optional:** Port hopping

The system dynamically switches transport **without breaking the session**.

---

## 5. Encryption and Handshake Design

### 5.1 Avoiding TLS Completely

TLS has:

* Known ClientHello fingerprints
* Cipher suite patterns
* Extension ordering

### 5.2 Noise Protocol Framework

Use **Noise** to design a custom handshake:

* No certificates
* Minimal packet count
* Fully customizable pattern

Recommended pattern:

* One-round-trip handshake
* Ephemeral key exchange
* No static identifiers

### 5.3 Symmetric Encryption

* Algorithm: **XChaCha20-Poly1305**
* Advantages:

  * Nonce misuse resistance
  * Fast on all CPUs

### 5.4 Key Rotation

* Rekey every N megabytes or N minutes
* Rekey packets indistinguishable from data packets

---

## 6. Traffic Shaping and Obfuscation

This is the **most critical component**.

### 6.1 Packet Size Randomization

Instead of fixed MTU-sized packets:

* Random fragment sizes
* Non-uniform distribution

Example:

```
137 bytes
891 bytes
54 bytes
1200 bytes
301 bytes
```

### 6.2 Timing Behavior (Anti-Behavioral DPI)

* Random jitter
* Burst + idle cycles
* Artificial pauses

The tunnel must sometimes **do nothing**, just like real apps.

### 6.3 Padding Strategy

* Random padding
* Sometimes no padding at all
* Padding size not correlated with payload

---

## 7. Traffic Mimicry (Behavioral, Not Header-Based)

The tunnel should statistically resemble:

* HTTP/3 video streaming
* WebRTC data channels
* CDN-backed services

Key metrics to mimic:

* Packet inter-arrival times
* Burst duration
* Idle frequency

Not:

* HTTP headers
* Fake TLS handshakes

---

## 8. Connection Maintenance (Keep-Alive)

Traditional keep-alives are detectable.

Instead:

* Random heartbeat intervals
* Heartbeats indistinguishable from data
* Silence windows allowed

---

## 9. Multi-Path and Failover Design

### 9.1 Path Diversity

* Multiple IPs
* Multiple ports
* Multiple transports

### 9.2 Seamless Switching

State is maintained independently from the transport:

```
Session ID
Crypto State
Sequence Numbers
```

Transport failure does not kill the session.

---

## 10. Reliability Layer

Since UDP is unreliable:

* Lightweight ACKs
* Selective retransmission
* Forward error correction (optional)

Avoid TCP-like behavior — stay unpredictable.

---

## 11. Implementation Language

### Why Go is Ideal

* Native concurrency
* Strong crypto libraries
* Easy static binaries
* Precise network control

Alternative languages:

* Rust (excellent but slower iteration)
* C (maximum control, high risk)

---

## 12. Deployment Considerations

### Inside Iran

* Prefer residential-like IP behavior
* Avoid data center patterns if possible

### Outside Iran

* Use stable providers
* Multiple IP addresses
* Avoid known VPN-hosting ASNs if possible

---

## 13. Testing Methodology

* Long-lived connections (hours/days)
* Forced packet loss simulation
* Manual route changes
* DPI-trigger testing

Metrics to track:

* Session survival time
* Reconnect success rate
* Throughput stability

---

## 14. Security Considerations

* No static identifiers
* No reusable fingerprints
* Memory zeroing after session end
* Replay protection

---

## 15. Conclusion

A stable tunnel in restrictive environments is **not a VPN problem**, but a **systems and behavior problem**.

By combining:

* Custom cryptography
* Adaptive transport
* Behavioral camouflage
* Real-time adaptation

it is possible to build a tunnel that significantly outperforms traditional solutions in hostile networks.

This approach requires engineering effort, but the result is a **long-term, resilient, and controllable communication channel**.
