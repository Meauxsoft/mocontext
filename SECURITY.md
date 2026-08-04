# Security policy

## Supported version

Security fixes are provided through the current public Mo-Search installer.
Before reporting a problem, reproduce it with the latest public release when
practical.

## Reporting a vulnerability

Please do not open a public GitHub issue for a suspected vulnerability.
Email [Questions@meauxsoft.com](mailto:Questions@meauxsoft.com) with the
subject `[Security] MoContext`.

Include:

- the installed Mo-Search/MoContext version;
- Windows version;
- clear reproduction steps and expected impact;
- the smallest safe diagnostic excerpt needed to explain the issue; and
- a way to contact you for follow-up.

Do not send passwords, API keys, private documents, or an unredacted index,
memory folder, or support report. We will acknowledge and investigate reports,
but do not currently promise a fixed response or disclosure timetable.

## Local trust boundary

MoContext binds to `127.0.0.1` by default and currently has no authentication
token configured by default. Other processes running as the user on the same
computer may be able to call it. Treat local software and AI clients as part
of the same trust boundary.

Do not expose MoContext through a LAN bind, port-forward, reverse proxy, tunnel,
or public endpoint. Those arrangements are not currently supported and require
authentication, transport security, authorization, and threat-model work that
the default local configuration does not provide.
