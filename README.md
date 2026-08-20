# status.me holding page

A white page with the feedmypixel mark bottom-right, linking to
[feedmypixel.com](https://feedmypixel.com). It exists to hold the domain while the real thing is
built.

Two files, no build step, no dependencies.

## Live

GitHub Pages, served from `main` at the repo root. `CNAME` carries the custom domain. Pages rewrites
that file when the domain changes in the UI, so change it there rather than here.

## DNS, as it stands

At GoDaddy. Nine records, and every one of them earns its place:

| Type  | Name             | Data                                                            |
| ----- | ---------------- | --------------------------------------------------------------- |
| A ×4  | `@`              | `185.199.108.153`, `.109.153`, `.110.153`, `.111.153` (GitHub Pages) |
| CNAME | `www`            | `feedmypixel.github.io`                                          |
| CNAME | `_domainconnect` | GoDaddy's setup helper                                           |
| NS ×2 | `@`              | `ns69` / `ns70.domaincontrol.com`                                |
| SOA   | `@`              |                                                                  |

**There is deliberately no `MX` record.** Mail to `@status.me` bounces, which is correct until Resend
is set up. The domain arrived pointing at the previous owner's Microsoft 365 tenant, so until that
record was deleted, anything sent to `@status.me` was landing in a stranger's mailbox. Eighteen
inherited records went with it: `secureserver.net` mail plumbing, Skype for Business `SRV` records,
and an `auth` A record. Stale records pointing at services you do not control are how subdomain
takeovers start.

Same reasoning applies to the two inherited `TXT` records, now gone. The SPF was
`include:secureserver.net -all`, a hard fail naming someone else's mail servers as the only valid
senders. Left in place, the first Resend message would have failed SPF and gone to spam.

## Retiring this

When the app is ready, `status.me` moves to Fly:

```bash
flyctl certs create status.me -a stat-ui   # prints the records it wants
```

Then delete the four Pages `A` records. **Never run both at once.** Two services claiming the apex
means whichever answers first wins, intermittently, and it looks like a caching bug rather than a
config one.

Archive this repo at that point. A second thing that still believes it owns the domain is the
problem, not the leftover HTML.
