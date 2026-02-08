# Building a Real Ban System for IO Games (Without being retarded)
> **Note:** This repository is a guide for developers if they wish to create a 'Next Generation' ban system. IP banning is a very common method in most .io games, and it is extremely easy to circumvent. This system is intended to be **server-sided** and **multi-layered**.

However,

## 1. What Makes a Ban System Good?

First, a real ban system should not be based on just one factor. It must be a variety of things.
1.  **Server-Side Control:** Never trust the client (browser). The final decision must always be on the server.
2.  **Multi-Layer Identity:** UserID + Fingerprint + TLS Hash + Behavior.
3.  **Logs & Transparency:** You should save *why* you banned someone (UserID, JA3, Reason, Date).

4.  **Persistence:** In case they delete the cookies, it will automatically rest the ID (Evercookie).

5.  **Anti-Bypass:** It should be able to detect use of VPNs and

The actors
## 2. Why Do Most IO Games Fail?
The following are the most common weak points found in other games:

*   **IP Ban:** The player restarts their router and gets a new IP, returning to the game immediately.

*   **Client Side Ban:** If you write if (banned) { alert("Bye") }, the cheater will simply remove the code from the browser console (F12).

*   **Simple LocalStorage:** Just delete the browser's history/data, and the ban will be lifted.

*   **No Behavior Check:** The bots move the mouse in perfect straight lines, which most games do not take into account.
The binary
## 3. The Architecture (How It Works)
The system operates with a loop between Client and Server.
### [CLIENT SIDE]
1.  **User opens the game.**

2.  **JavaScript collects data:**
*   **Fingerprint:** (Canvas + WebGL + Audio + RAM + Screen Resolution).
*   **Local IP:** (via the WebRTC leak if possible).
*   **Behavior:** (Mouse movements, typing rhythm)
3.   **Sends this data to the server** via `POST /auth`.
### [SERVER SIDE]
4.  *The server receives the data.*
5.  **Crucial Step:** The server will determine the **JA3 Hash** based on the connection (TLS Handshake).
*   *Note: JavaScript is invisible to JA3, whereas the server is not.*
6.  **The server checks the Database:**
*   Is **UserID** banned?

*   Is this **Fingerprint** banned?

*   Is this **JA3 Hash** suspicious (i.e., Python bot hash)?

*   Is the **IP** a known VPN?
7.  **Decision
*   If **BANNED**: Return `{ ok: false, reason: "Security Violation" }`.

*   If **CLEAN**: Generate a temporary **Token** and have them play.
#Image

### 4. Advanced Browser Fingerprinting

We not only use Canvas, we develop a "Super Hash" by hashing various hardware information.

**How it works:**
*   **Canvas & WebGL:** The browser creates an invisible image in 3D. Every Graphics Card (GPU) displays this slightly differently.
*   **AudioContext:** An audio context is created by the browser. The actual math of the sound compression varies depending on your hardware.
*   **Hardware Info:** RAM, number of CPU Cores, Screen size.

**How to bypass & Fix:**
*   *Cheaters use:* Brave browser or "Canvas Blocker" extensions to add noise (random) values.

*   *Solution:* If the canvas data looks "noisy" -- looks mathematically fake -- mark this user **Suspicious**, but don't ban them at this time; send them a CAPTCHA.

Being a

### 5. TLS Fingerprinting (JA3) - Server Side
This is the strongest because it happens at the network layer, not the browser.
**How it works:**
So, henceforth, when your browser establishes a connection with your server using https, your browser says "hello." However, different ways of saying "hello" are referred to as Cipher suites, SSL version, etc. All these combinations are represented as a unique
*   "Chrome has a specific JA3."

*   Firefox uses a different one.
*   The Python script, i.e., the Bot has a totally different one.

**Strategy

*   User claims to be "Google Chrome" but has a "Python" JA3 hash -> **Ban immediately.**

*   The banned user changes their IP, clears their cookies, but his **JA3 + Fingerprint** is the same.-> **Ban again.**
---
## 6. "Evercookie"

Cheaters typically delete cookies to avoid the ban. To prevent this, UserID must be saved always.

**Storage Locations:**

1. LocalStorage
2.  SessionStorage

3. IndexedDB (Browser Database)
4.  Cache Storage
5. Service Workers

**Logic

The script checks all these locations when the user visits the site. *   If the user deletes the *Cookies*, but forgets the *IndexedDB*, the script locates the ID and **re-saves (respawns)** it back to Cookies. The ban is reinstated.* “BACKGROUND ## 7. IP & VPN Detection (WebRTC) Cheaters use VPN to hide their IP. **How it works:** "WebRTC is used in voice chats, however it leaks **Local IP**, also known as the **Real IP**." *    **Check:** Compare the HTTP Request IP (VPN IP) to the WebRTC IP. *   **Result:** When they do not match (e.g., HTTP is Germany, WebRTC is Turkey), it's a VPN. Block or validate. * By **8. Behavior Analysis** If the player skips all that, his **behavior** will give him away. **How it works:** *   **Mouse Path:** The movement of the mouse is curved for humans, with a few errors. However, for bots, this movement is perfectly linear. *   **Keystroke Rhythm:** All humans have individual typing speeds. For example, the time taken in between 'W' and 'A' key press. **Solution Record the last 10 seconds of movement. *   If `Movement_Variance == 0`: Perfect line -> **It is a Bot.** *   If the rhythm matches a previously banned player's rhythm, flag it for **Suspicious.** -,*, ### Conclusion "With **Server-Side JA3**, **Browser Fingerprinting**, and **Behavior Analysis**, you're creating a system which is very annoying to circumvent. For the cheater to play again, they must change their IP, Browser, Hardware, and Playstyle all at once."
