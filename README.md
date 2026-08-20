# status.me — holding page

A white page with the feedmypixel mark bottom-right, linking to
[feedmypixel.com](https://feedmypixel.com). It exists to get `status.me` off the previous owner's
parking page while the real thing is built.

Two files, no build step, no dependencies. `CNAME` is what tells GitHub Pages the custom domain —
Pages rewrites it if you change the domain in the UI, so edit it there rather than here.

## Deploying

GitHub Pages, because the account already uses it and swapping away later is one DNS change.

1. Create a repo `feedmypixel/status-me` (public — Pages on a private repo needs a paid plan, and
   there is nothing here worth hiding).
2. Push this directory to it.
3. **Settings → Pages** → Source: *Deploy from a branch* → `main` / `/ (root)`.
4. **Settings → Pages → Custom domain** → `status.me` → Save.
5. Once the DNS below has propagated, tick **Enforce HTTPS**. It stays greyed out until GitHub has
   issued the certificate, which usually takes minutes but can take an hour.

## DNS at GoDaddy

The domain currently `301`s to `mybigdomainsale.com` — that redirect is the broker's, and it lives
in the DNS records. **Deleting those records is the step that actually removes the old page.**

- **Delete** any existing `A` record on `@`, any `CNAME` on `www`, and any forwarding rule under
  *Domain Settings → Forwarding*. GoDaddy's forwarding is separate from the DNS tab and is the usual
  reason a parking page survives an A-record change.
- **Add** four `A` records on `@`, pointing at GitHub Pages:
  ```
  185.199.108.153
  185.199.109.153
  185.199.110.153
  185.199.111.153
  ```
- **Add** a `CNAME` on `www` → `feedmypixel.github.io`

### The `TXT` records are the previous owner's, not ours

Worth knowing before the mail setup, because the usual advice — *leave `TXT` alone* — is wrong here.
The domain arrived carrying:

```
"v=spf1 include:secureserver.net -all"
"NETORGFT9519691.onmicrosoft.com"
```

The second is a Microsoft 365 domain-verification token belonging to whoever held the domain before.
It does nothing for us and can go.

The first matters more. `-all` is a hard fail: it tells receiving servers that **only**
secureserver.net may send as `status.me`, so the moment mail goes out through Resend it fails SPF and
lands in spam. It has to be replaced with Resend's record — not merged, not left in place. Two SPF
records on one domain is itself a misconfiguration.

Neither affects this holding page, so the `A` records can be swapped now and the mail work can follow.

## When the real app is ready

Point the apex at Fly instead (`flyctl certs create status.me -a stat-ui` prints the records it
wants) and delete these four `A` records. Do not run both at once — two services claiming the apex
means whichever answers first wins, intermittently.

Archive or delete this repo at that point rather than leaving a second thing that thinks it owns the
domain.
