# Concepts

Public key infrastructures are used to establish trust in public keys in a variety of contexts,
including authenticating servers and end entities when using transport layer security (TLS),
supporting signed and encrypted email, and verifying signatures on documents or software. In a PKI,
there are a relatively small number of certification authorities (CAs) and a large number of end
entities. A smaller subset of CAs are explicitly trusted. These are referred to as trust anchors
(TAs).

The `certval` library uses the fact that TAs and CAs change infrequently and can be arranged to form
a map of a given PKI. The relatively expensive operation of preparing a map is performed offline
(relative to establishing trust in an end entity) with the results of that operation saved for use
during frequent validation operations.

The results are saved using the following structures described using the Common Data Description
Language (CDDL) and encoded using the Concise Binary Object Representation (CBOR) data format. The
structure holds an array of certificates alongside the partial paths found for them. A partial path
is an array of indices into that array of certificates, naming the certificates that make up the
path; the paths are grouped in a map keyed by the subject key identifier of the CA that terminates each.
Keying them that way turns path building into a simple table lookup: a certificate yields its issuer's key
in its authority key identifier, and that key indexes the paths that could serve it.

```cddl

  store = {
    "buffers"       : [* cert-file],
    "partial_paths" : partial-paths,
  }

  cert-file = {
    "filename" : tstr,   ; a filename or URI — provenance only, not identity
    "bytes"    : der,
  }

  ; One binary DER-encoded certificate.
  der = bstr

  ; One map per source. Each key is the SKID of a leaf CA as uppercase ASCII hex;
  ; each value is that CA's partial paths.
  partial-paths = [* { * skid => [* path] }]

  skid = tstr .regexp "[0-9A-F]+"

  ; Positions in "buffers", ordered trust anchor to leaf CA. Every index must
  ; exist in "buffers" — new_from_cbor validates this, because the indices are
  ; consumed without repeating the search that produced them.
  path = [* uint]
  ```

## Trust stores

A trust store is a pair of CBOR files. The trust-anchor part (`*_ta.cbor`) holds roots. The CA part
(`*_ca.cbor`) holds intermediate CA certificates together with precomputed partial certification
paths.

Discovering partial paths is the expensive step of establishing trust in an end entity certificate.
Validating an end entity certificate against a precomputed partial certification path already
discovered is cheap. A store is not simply a bag of certificates, it is a bag of certificates with
path building already done.

### How are trust stores obtained

There are four methods for obtaining a pair of trust store files. All use the same format and all
are interchangeable.

**Generated from certificates you have.** `pittv3 --generate` reads trust anchors and CA
certificates and generates the CBOR files — from folders, or through repeatable `--ta` and `--ca`
arguments, which also take an existing `.cbor` store, so several stores can be combined into one; the
Generate view does the same from material indicated on the form, in the desktop application, and from
material uploaded to the page, in the browser. This is how a custom store for your own PKI can be
created. AIA/SIA chasing, i.e., `--dynamic-build` on the command line, *Chase SIA and AIA* on the UIs,
extends the collection by following the URIs the certificates includes, so the network work happens
during generation rather than on every validation. In the browser, artifacts are retrieved by a PITTv3 
service on the page's behalf, so it is available only where one is serving the page.

The `certval-store-gen` crate in the `certval-stores` workspace includes a library and a binary that 
may be useful in generating custom trust stores for use with `pittv3` or `certval`-enabled products.

**Built into the application.** Selectable from the store dropdown on the Validate view without any
network access. There is a small ecosystem of crates that implement the necessary interfaces to
serve as selectable options.

**Served by a PITTv3 service.** Where the browser application is served by one, the stores it holds
appear in the same dropdown and are downloaded from it. The dropdown says which is which, and
whether a store came from a trust store provider or was configured by whoever runs the service —
worth knowing, because a configured store may hold chased material rather than published trust
material.

**Saved from a run.** *Save artifacts* on the Results view writes a bundle whose
`derived/built-ta.cbor` and `derived/built-graph.cbor` are the trust material a validation actually
used, in this same format. That is the way to capture a store you assembled by uploading, or by
letting a run chase for certificates it did not have.

Generating and exporting differ in direction, which is easy to miss because they produce the same
thing. Generating is prospective: it takes everything in the folders you point it at, whether or not
any of it is ever needed. An export is retrospective: it holds only the material one validation
actually touched, including anything that run fetched along the way, and nothing it did not.

### Using trust stores

Upload either part through the trust-anchor and intermediate-CA controls on the Validate view or 
select a built-in trust store. A `.cbor` upload merges all of its certificates into a pool for use, 
so stores mix freely. You can mix Web PKI roots with another collection's intermediates, or your own 
trust anchors with a built-in CA store.

Offline store-generation tooling produces the same format, so a store you build yourself uploads
exactly like a built-in one, and a store exported from a run can be handed to the command line or
the desktop application.

## Partial paths

A partial path is an ordered sequence of certificates from a trust anchor down through intermediate
CA certificates, stopping short of any end-entity certificate. Computing them is what makes a store
worth more than the certificates it contains: at validation time the work is matching an end-entity
certificate against paths already found, rather than searching the whole collection again.

This is also why a store is tied to the material it was built from. If certificates are adding to a 
CA folder, the partial paths in an existing store do not know about them; the store has to be regenerated
for the new material to take part.

The same holds for trust anchors. A partial path begins at a certificate issued by one of the
anchors loaded when the paths were found, so a CA store carries paths only to those anchors. The two
halves of a store do not have to be used together — a CA store can be paired with a different trust
anchor store at run time — but a path to an anchor that was absent when the CA store was built is
not in it, even when the certificates are. To use one CA store with several anchor sets, build it
with all of those anchors loaded.

A built-in trust store is not the only input a validation operation. Loose CA certificates given alongside 
one are searched together with it, because a path spanning the two exists only once both are considered, and
dynamic path building adds whatever it fetches to the same graph.

That whole graph is what the **graph cache** keeps, under `~/.pittv3/graphs` — including the
certificates a run had to go and fetch, which are the expensive ones to have found. A later run over
the same material neither parses nor re-searches any of it. The cache is written only when there is
something to combine: a run given nothing but a store has nothing to add to it, and generating a
store does not use it at all.

The `pittv3` suite will be periodically updates as the built-in trust stores change. The **graph cache** 
provides a means of staying current until updates are applied.