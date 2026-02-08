# Building a Real Ban System for .io Games (Stop Using IP Bans)

> **Heads up:** This repo is for devs who actually want to stop cheaters. If you are just using `socket.remoteAddress` to ban people, you are doing it wrong. This guide explains how to build a **server-sided**, **multi-layered** system that is genuinely annoying to bypass.

---

## 1. The Philosophy (How not to suck)
A good ban system isn't about one magic trick. It's about layering traps. If you trust the client (the browser), you lose. Always.

**Your system needs to be:**
*   **Server-Authoritative:** The client sends data, but the Server decides the truth.
*   **Sticky:** If they clear cookies, the ban should stay.
*   **Noisy:** Log everything. You need to know *why* someone got banned (JA3? Fingerprint? Behavior?).
*   **Smart:** Don't just ban IPs. Ban the identity.

---

## 2. Why Most Games Fail
Let's be real, most .io games have terrible security.
*   **IP Bans:** Kid restarts router -> New IP -> Back in game. Useless.
*   **Client-Side Checks:** If you have code like `if (isBanned) window.close()`, any script kiddie can delete that line in the F12 console.
*   **Ignoring Behavior:** Bots walk in perfect straight lines. If you aren't checking for that, you're inviting them in.

---

## 3. The Architecture
Here is the logic. It works in a loop between the Client and your Backend.

### [CLIENT SIDE]
1.  **User loads the game.**
2.  **JS grabs the fingerprints:**
    *   Canvas, WebGL, AudioContext, RAM, Screen Res.
    *   Tries to leak Local IP via WebRTC (if the browser allows it).
    *   Records mouse movement variance (is it robotic?).
3.  **Sends it to `/auth`.**

### [SERVER SIDE - The Brain]
4.  **Receive the data.**
5.  **The Secret Weapon (JA3):** While the handshake happens, the server calculates the **JA3 Hash** from the TLS connection.
    *   *Note: You can't do this in JS. It has to be done on the backend (Node, Go, Nginx, etc).*
6.  **Check the DB:**
    *   UserID banned?
    *   Fingerprint hash matches a banned one?
    *   JA3 hash looks like a Python script instead of Chrome?
    *   IP belongs to a VPN provider?
7.  **Verdict:**
    *   **BANNED:** Send a generic error (don't tell them exactly why).
    *   **CLEAN:** Give them a temporary token and let them play.

---

## 4. Fingerprinting (Make it stick)
Canvas fingerprinting alone is too easy to fake now. You need a "Super Hash". Combine WebGL renderer info, AudioContext latency, and hardware concurrency.

**The "Brave" Problem:**
Browsers like Brave or extensions like "Canvas Blocker" add random noise to the data.
*   **My advice:** If the fingerprint looks mathematically "noisy" or fake, don't ban them instantly (might be a privacy nut, not a cheater). Just flag them as **Suspicious** and force a CAPTCHA.

---

## 5. TLS Fingerprinting (JA3)
This is your best defense against bots.
When a browser connects to your HTTPS server, the "Client Hello" packet has a specific structure.
*   Chrome's structure is unique.
*   Firefox is different.
*   A Node.js or Python bot is **completely different**.

**Strategy:**
If the User-Agent says "I am Google Chrome" but the JA3 hash says "I am a Python Script" -> **Instant Ban.**

---

## 6. "Evercookie" (Persistence)
Cheaters love clearing cookies. We can make that harder.
Don't just save the UserID in `document.cookie`. Save it in:
1.  LocalStorage
2.  SessionStorage
3.  IndexedDB
4.  Cache Storage
5.  Service Workers

** The Logic:**
When they visit the site, check *all* of these. If they cleared Cookies but forgot IndexedDB? **Respawn the cookie.** The ban is back.

---

## 7. Detecting VPNs & Proxies
WebRTC is useful here. Even if they use a VPN, WebRTC can sometimes leak the real IP or at least show a mismatch.
*   **Check:** Does the HTTP Request IP match the WebRTC IP?
*   **Result:** If HTTP says "Germany" but WebRTC says "Turkey" (or fails completely), it's likely a proxy. Flag it.

---

## 8. Behavior Analysis
If they spoof everything else, they can't easily spoof being human.
*   **Mouse Path:** Humans make curved movements with variable speed. Bots move in straight lines (`variance == 0`).
*   **Rhythm:** Track the delay between key presses. If it's perfectly constant (e.g., exactly 50ms every time), it's a macro.

---

### Final Thoughts
There is no such thing as "100% unbannable." But if you combine Server-Side JA3, hardware fingerprinting, and behavior checks, you force the cheater to change their IP, Browser, Hardware, and playstyle all at once to get back in.

Most of them will just give up. Good luck.
