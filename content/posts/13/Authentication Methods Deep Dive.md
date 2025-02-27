---
title: Authentication Methods - A Deep(ish) Dive
date: 2025-02-27
description: JWT or sessions?
toc: true
math: true
draft: false
categories:
  - websec
tags:
  - JWT
  - sessions
  - cookies
  - authentication
  - CSRF
  - XSS
---

> Here lies the ramblings of a madwoman; bumbling her way around in the darkness in an attempt to understand the wide world of websec...

# ... in the absolute broadest of strokes:
- **Token-based (`JWT`):**
	- Authentication state is stored **on the client** (local/session storage) in the form of a **token**.
- **Session-based:**
	- Authentication state is **stored on the server's database**.

Now, let's go a little deeper, shall we?

## - JSON Web Tokens (JWT)
![](/posts/13/Screenshot%202025-02-27%20at%209.23.47%20pm.png)

### - How it works:
1. Client sends credentials to sever 
2. Sever generates a `JWT` based on credentials, and provides it to user (following below structure). 
	- For example, using the `RS256` algorithm, the generated `JWT` is signed with the **server's private key**, and verified by the client with the **server's public key**.
	  ![](/posts/13/Screenshot%202025-02-27%20at%209.24.40%20pm.png)(`JWT` structure - https://jwt.io/)
3. The client receives the`JWT`, which is **stored** in the **client's local storage/session storage/as a cookie.** AKA, the ***state lives as a token on the client***, instead of **on the** **server** (as is with typical session-based authentication).
	- *Note:* *the client also verifies the `JWT` with the server's public key, if using the `RS256` algorithm.*


### - JWT-based Authentication Drawbacks
- **State is stored client side** & can thus be dissected & manipulated
- **Vulnerable to being accessed/stolen via XSS attacks**
- ***Can* be vulnerable to CSRF** based on how the `JWT` is stored & sent.
	- **Vulnerable to CSRF**: if the `JWT` is stored as an **`HTTP`-only cookie** that is passed to the server with **every request**.
		- to mitigate this, use `SameSite=Strict` & additional `CSRF` tokens with each request.
	- **NOT (as) vulnerable to CSRF**: if the `JWT` is stored in the `local/session storage`, meaning it's **not sent with every request**. Instead, it must be manually passed into the request header (e.g. `Authorization: Bearer <token>`) when authorising.
- **No server-side revocation** - token is valid until it expires.
- **Token expiration management** can be complex
- **Data is `base64` encoded, not encrypted** - so sensitive data should never be stored in JWTs, as anyone with the token can decode and read its contents.

---

## - Session-based (cookie) authentication:
![](/posts/13/Screenshot%202025-02-27%20at%209.26.19%20pm.png)

### - How it works:
1. Client provides credentials to the server
2. Server generates a **unique session ID** for the client and **stores the session details & state in its local database.**
3. Server sends the **session ID** back within an `HTTP-only` cookie, which is **stored in the client browser's cookie jar** (a storage for key-value pairs - *how cool is this name though-*)
4. The client sends this cookie back with subsequent requests, & each time, the server has to **check the session** against the value in the server's database.
5. Upon logout, session ID is cleared from both the **client side** and **server database**.
![](/posts/13/Screenshot%202025-02-27%20at%209.46.32%20pm.png)

### - Session-based Authentication Drawbacks:* 
- **Vulnerable to CSRF** (attackers using session IDs to perform actions on behalf of the user) as cookies are sent automatically with every request.
- **Processing power & complexity that increases with scale:** as sessions have to be generated, stored & managed on the server's database.
- **Domain Restriction:** Cookies are domain-specific, making cross-domain authentication difficult without additional configurations like `CORS` (Cross Origin Resource Sharing) or third-party cookies.
	- `CORS`: when a web app makes a cross-origin request (e.g. `example.com` to `api.example.com`), the browser sends an additional `CORS` **preflight request** to check if the server (`api.example.com`) allows the cross-origin request. If it does, it needs to respond with the appropriate `CORS` headers.

---

## A brief comparison...
![](/posts/13/Screenshot%202025-02-27%20at%2010.43.47%20pm.png)
> credit where credit is due, this *is* from ChatGPT, but it was used as a sanity check after I did the bulk of the manual research to build a basis of understanding. 
so, what am i saying by this? take... ***all of it with a grain of salt lol-***

**Helpful Resources:**
- [JWT attacks - Portswigger](https://portswigger.net/web-security/jwt#how-do-vulnerabilities-to-jwt-attacks-arise)
- [Session vs Token Authentication in 100 Seconds](https://www.youtube.com/watch?v=UBUNrFtufWo&list=TLPQMjcwMjIwMjXKogKOoZBbBQ&index=4&t=65s)
- [Web Authentication Methods Explained](https://www.youtube.com/watch?v=LB_lBMWH4-s&list=TLPQMjcwMjIwMjXKogKOoZBbBQ&index=4)
- [Session-Based Authentication vs. JSON Web Tokens (JWTs) in System Design](https://www.geeksforgeeks.org/session-based-authentication-vs-json-web-tokens-jwts-in-system-design/)
- [Exploiting it in practice, within a CTF](https://aegizz.github.io/ctfs/duckCTF2024)
