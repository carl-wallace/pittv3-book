# Service and relay

Two crates serve the browser application, and they do different jobs. The distinction matters,
because it decides what leaves the browser.

## The relay

A browser can build and validate paths — `certval` compiled to WebAssembly does that in the page —
but the same-origin policy stops it from reading a CRL distribution point or posting an OCSP
request. The relay does that part on the page's behalf.

It handles the three verbs PKI retrieval needs: `GET` for certificates and CRLs named by authority
information access, subject information access and CRL distribution point extensions, and `POST` for
OCSP requests. It does **not** parse what it retrieves, does not cache by certificate identity, and
does not distinguish a CRL from any other sequence of bytes. Conditional request headers pass
through, so a caller gets the benefit of `If-Modified-Since` exactly as the command-line tool does.

What that means in practice: with the relay in play, **the URIs a certificate names leave the
browser, and an OCSP request identifies the certificate being asked about** — but the certificates
themselves stay in the page, and validation still happens there.

## The service

The service is the relay plus everything else a deployment needs: it serves the browser application
itself, offers trust stores for download, and can validate server-side for a caller that would
rather send certificates than retrieve artifacts.

### What a request that states nothing means

`POST /api/validate` applies the service's own defaults before anything the caller sent. The one
that matters is the time of interest: **a request stating no settings validates against now.** The
library cannot supply it — the service builds certval without `std`, which is a build with no clock
— and an unstated time is not a wider validity window but no validity check at all.

Settings a caller does send ride *on top of* the defaults rather than replacing them, so stating one
preference does not withdraw the rest. The report names the time that was used in
`time_of_interest`, which is the field to read first when a verdict looks wrong.

### Trust stores

`--stores` names a directory of stores, **read at startup and never written to**. Two layouts are
accepted: a folder per store holding `ta.cbor` and `ca.cbor`, or flat `<id>_ta.cbor` /
`<id>_ca.cbor` pairs, as the trust store providers generate them. A saved artifacts bundle's
`derived/built-ta.cbor` and `derived/built-graph.cbor`, copied into a folder under those two names,
serve as a store.

Stores are served from `/stores/{id}/{artifact}` with an entity tag computed over the bytes, so a
client that already holds a store revalidates with a conditional request rather than downloading it
again. The tag is per artifact rather than per store: sharing one between the two parts would tell a
client holding the anchors that its CA store was current.

A store offering anchors only answers `ca.cbor` with a 404, which is correct rather than a fault —
`webpki_tls` and `webpki_email` have no CA part.

The listing at `api/stores` describes each store to a client's selector: its identifier and name,
the URIs of its two artifacts, whether it came from a trust store provider or the configured
directory, and — for a provider store — when its material was published and when it was collected.
A store read from `--stores` reports neither date: a `.cbor` artifact holds certificates and partial
paths, and nothing about when they were gathered, so the service would be guessing.

### Turning parts off

A deployment is expected to disable what it does not want, and the defaults are conservative.

- **Refusing `POST /api/validate`** leaves the relay and the stores. A deployment that wants
  certificates never to leave the browser runs this way.
- **Chasing AIA and SIA server-side is off unless asked for.** It turns one uploaded certificate
  into a good deal of outbound traffic.
- **Server-side CRL retrieval can be stopped**, leaving revocation to whatever the caller supplied.
- **Refusing `POST /api/tls`** stops the service completing a handshake with a host named by a
  caller.
- **`--no-builtin-stores`** offers only what `--stores` names, for a deployment that means to serve
  a chosen catalog and nothing else.

### Rate limiting

On by default, per client address, in two fixed windows that count **requests, retrievals and
bytes**. Retrievals are counted apart from requests because the ratio is the client's to choose: a
single fetch costs one, while a validation that chases costs as many as its budget allows. A
validation is charged what it actually retrieved rather than what it might have.

The limits live in a `rate_limit` section of the configuration file rather than on the command
line, a zero meaning unbounded. The effective limits are printed at startup, and turning them off
prints a warning. A refused request answers 429 with `Retry-After`; the browser application treats
that as terminal and stops the run rather than meeting the limit once per remaining URI.

The address is the only key, since the service has no sessions — so an office behind one NAT shares
a budget, which is what the defaults are sized for.

### Serving the application

`--client-dir` names the directory `trunk build` produced. Cache-Control is chosen per file
according to whether its name carries a Trunk content hash — `<name>-<16 hex>.js` and the matching
`_bg.wasm` cannot change without changing their names, so they are safe to pin, while everything
else revalidates. The test looks for a hash **in the file name only**, so an unhashed asset is
never pinned by accident.