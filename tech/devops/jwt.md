---
tags:
  - devops
  - tools
created: 2024-01-01
updated: 2025-01-23
status: active
---


- It is just a token format
  - Popular used in authorization and authentication protocol like OAuth2 and OpenID
- Use Bear Authentication Schemes
- Header, Payload, Singnature
- Can be decoded by everyone, so it is
  - Must be transmitted under HTTPS (But still not enough)
  - Not suitable for Web Authentication, usually used for API
  - Should not stored in localStorage or Cookie, but memory
    - unless with strictly CSRF protection implemented, like SameSite, keep secure and httpOnly
    - https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite
    - use Content-Security-Policy


# Why Refresh Token can be safely use

Because the Server provided /refresh_token with POST API and must have CORS Policy to prevent requests from unauthroized web/sources

# Articles

https://anil-pace.medium.com/json-web-tokens-vs-oauth-2-0-85dd0b32057d#9c7d
https://dev.to/siwalikm/what-the-heck-is-jwt-anyway--47hg
https://blog.logrocket.com/jwt-authentication-best-practices/

# Discussion, Security, Storage

https://stackoverflow.com/questions/27067251/where-to-store-jwt-in-browser-how-to-protect-against-csrf
