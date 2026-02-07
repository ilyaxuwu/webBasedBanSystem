# Building a real ban system for all io games (Without being a retard like sploop.io did)
> This repo was wrote by the old sploop.io hacker called ilyax without getting ban since I have wrote antiban
> Also if you are an sploop.io player please spread this to ingame moderation team to apply this so they won't get ban over over and over again.

This guide is for people who actaully want to **learn how to make next generation of ban system who people couldn't bypass** - not those who uses the url `https://api.ipify.org` and fetch to take blacklist and call it `They got ban and they won't comeback again (like sploop did)`

In this README, you'll understand how to make a better ban system

If you wan't to ban people on your scripts, your games, or more like that this repo is for you

Star this repo if you really found this useful :D

---

## First of all, What Makes an Ban System Good?
A Real Ban System Should:
1. Server-Side Enforcement (Client Side Bans are bypassable
2. Multi-Layer Idenity (UserID Ban, FingerPrint Ban, Token Ban, Behavior Ban,)
3. Audit & Transparency (Ban reasons should be logged on DB. UserID, Fingerprint, ja3, reason, timestamp)
4. Persistence (Evercookie, LocalStorage, IndexedDB => They can get unban if they delete cookies)
5. Resilience Against Bypass (Anti VPN, Anti Detect Browser)
If your players knows this assume an hacker can know it too.

---

## Secondly, Why most io game have failed ban system?

Here are the most common ban system:

* IP Ban (Restart your modem and unban)
* Client-side Bans if (banned) { alert("banned") } (Delete the JavaScript codes from Developer Tools (F12) and unban)
* Only one layer ban (Only bans UserID or IP Make new acc or restart ur modem and unban)
* No Permanent ban (They bans by localStorage and cookie delete them and unbanned)
* Infinite tokens (Token never expires, You're banned but token still works)
* No VPN/Proxy Detection (Use VPN and comeback again)
* No Behavior Analysis (Bots plays straight line presses or clicks faster anything at same time)

---

## 3. The Architecture

Your Ban System Should Work Like this:
```
[CLIENT SIDE]
↓
[CLIENT SIDE]
1. User opens the game
2. JavaScript collects:
   - userId (e.g. Discord ID, license key)
   - fingerprint (Canvas + WebGL + Audio + RAM + GPU + Screen)
   - JA3 hash (TLS fingerprint from HTTPS handshake)
   - behavior profile (mouse path, keystroke rhythm)
   - local IP (via WebRTC leak)
3. Sends all data to server via POST /auth
[SERVER SIDE]
4. Server receives: userId, fingerprint, ja3, behavior
5. Server checks ban DB:
   - Is userId banned?
   - Is fingerprint banned?
   - Is JA3 hash banned?
   - Is behavior profile matching a banned one?
   - Is IP flagged (VPN/proxy)?
6. If any match → reject with { ok: false, reason: "banned" }
7. If clean → generate short-lived token:
   - token = crypto.randomBytes(24)
   - expires in 1 hour
   - store in DB: { token, userId, fingerprint, ja3, expires }
8. Return token to client
[CLIENT SIDE]
9. Client stores token in memory or IndexedDB
10. Client uses token to access protected routes (e.g. /load, /play, /getData)
[SERVER SIDE - Every Request]
11. Server receives token + fingerprint
12. Server checks:
   - Is token valid and not expired?
   - Does token match fingerprint?
   - Is token revoked?
   - Is userId or fingerprint or JA3 now banned?
13. If valid → allow access
    If expired → attempt silent re-authentication:
        - If fingerprint + userId + JA3 still match and not banned:
            → issue new token don't kick the player, return { ok: true, token, reauth: true }
        - Else:
            → reject with { ok: false, reason: "expired_and_reauth_failed" }
    If invalid or revoked → reject with { ok: false, reason: "invalid_or_banned_token" }
```

Since I have Explained How the new Ban system works, lets get started to do how to do it.

# 1 - Advanced Browser Fingerprinting

Not only Canvas, Its combined with all of hardware datas called "Super Hash"

Detailed Logic:

* Canvas & WebGL: It draws a invisible 3D Cube on browser. When GPU rendering the cube it uses lighting calculations, anti-aliasing techniques very slightly depending on the graphics card and driver version. 

* AudioContext: In browser it is generates a sine wave (sound) then compress them. The way the sound card processes this compression produces a unique mathmematical output.

* Hardware Concurrency & Memory: You can get core of the processor `navigator.hardwareConcurrency` and amonut of RAM `navigator.deviceMemory`

* Screen Properties: Screen size and color depth

It should collect all of the datas and combines with (example SHA-256) to generate Device_ID.

## How to bypass them? 

Canvas Blocker Extensions: The extension adds random noises when you reload the page the hash changes,

Brave Browser: It blocks your fingerprint

Anti-Detect Browserlar (Dolphin Multilogin): The program shows different of hardwares (Example: It shows my RTX 5080 into Intel HD Graphics)

## How to Solve This?

Noise Detection: If the taken Canvas doesn't show %100 clean and it is weird noise (mathematical inconsistency) it should say "This person is hiding fingerprint probably cheater" and dont take to the game

Entropy Analyzing: If they seems rarely hardware combination (Example: Linux Operating System, Old Chrome, 128GB RAM) Be suspicious, and allow them if they proved it.

---

# 2. TLS Fingerprinting (JA3 / JA3S) - Server Side

This is not JavaScript, It works on the Server Side (Backend) so it makes strong. even resetting router won't work.

Detailed Logic: 
When a Browser (Client) Sends "Hello" to the Server. It lists the encryption types, SSL versions, and extension it supports.
Chrome's hash ranking differs from Firefox's. Even the hash ranking of a Python bot is quite different. This hashing package is called a JA3 Hash.

## How to Bypass them?

Change Browsers (Using Edge instead Chrome changes JA3)
Special proxy software that manipulates encryption protocols.

## How to Solve this?

If the IP changes and IF JA3 Hash and Canvas Fingerprint matches it is the person who got banned.

--- 

# 3. "Evercookie"

Instead of saving data in one place, Evercookie writes it into every part of the browser.

When a user enters the game, a unique user_id is created and saved in:

* Standard Cookies
* LocalStorage
* SessionStorage
* IndexedDB (browser database)
* Cache Storage (file cache)
* Service Workers (background scripts)

This makes the ID very hard to delete.

## How to bypass them?

They clear all browser data, (history, cookies, storage)
They use Incognito Mode, Which deletes everything when the tab closes. 

## How to Solve this?

1. Incognito Detection
Use JavaScript to test the browser’s FileSystem API.
In Incognito mode, this API behaves differently or fails.
If detected, show a warning like:

“You’re using private mode. Please switch to normal mode to play.”

2. Re-spawning the Cookie
If the user deletes cookies but forgets to clear LocalStorage or IndexedDB,
your script can rebuild the cookie from those backups.

---

# 4. WebRTC Local IP Leak

Sometimes a banned player uses a VPN to change their IP address.
The server thinks it’s a new person. But JavaScript can still see the truth from inside the browser.

## How It Works:
WebRTC is a browser feature used for real-time voice and video.
When it connects, it can reveal:
The local IP address (like 192.168.1.25)
Sometimes the real public IP, even behind a VPN
This is called a WebRTC IP leak.

## How to bypass them?

They tun off WebRTC in browser settings
They use browser extension like uBlock Origin to block WebRTC leaks

## How to solve this?

* Send a STUN request using WebRTC when the game starts
* Compare the WebRTC IP with the HTTP request IP
* If they are different (for example: one is from Germany, one from Turkey), the user is using a VPN

Then show a warning like
> VPN Detected. Please turn off to play the game

---

# 5. Behavior Analysis

Lets say the user has changed the computer, and they want to play the game still with cheats. there is only left to trust is **human behavior**.

## How It Works: 

Mouse Path: Bots and macros usually move in straight lines.
Real humans move their mouse in curves, with small shakes and natural speed changes.

Keystroke Dynamics:  
Every person types with a unique rhythm.
For example, the time between pressing and releasing the “A” key, and then moving to “B”, is different for each user — measured in milliseconds.

## How they bypass them?

* To bypass this, the player would need to change their physical behavior, which is very hard.
* Bots can try to simulate human movement, but it’s not perfect.

## How to solve them? 

Use simple JavaScript to track:
Mouse coordinates and movement speed
Key press and release timing (keystroke delay)
Send this behavior data to the server.
Compare it with the behavior profile of banned users.
If the new player’s behavior is 95% similar to a banned one, it’s probably the same person.

---

Congrats you made unbypassable ban system.



