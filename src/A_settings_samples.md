# Appendix: settings samples

Three starting points for a settings file: **Standard**, **Offline** and **Strict**. They are
samples to copy and edit rather than modes the application offers — nothing in PITTv3 knows these
names.

Use one by saving it to a file and pointing an interface at it:

```sh
pittv3 -s standard.json -t ta_folder -b ca.cbor -e target.der
```

The desktop application reads and writes the settings file in its own folder, and the browser
application imports and exports the same format, so a file written here is portable between all
three.

**A settings file need only carry what differs from the defaults.** A key that is absent takes its
default value, which is also why the Standard sample below is short and why editing a sample down to
the two or three lines you care about is the normal thing to do. Every sample here was generated
from certval itself and loaded back, so each one is valid as printed.

## Standard

The shipped defaults, written out so they can be read rather than looked up. Revocation is checked,
CRLs and OCSP responses are both consulted, and material named by AIA and SIA may be retrieved.

```json
{
  "psCheckCrlDpHttp": { "Bool": true },
  "psCheckCrls": { "Bool": true },
  "psCheckOcspFromAia": { "Bool": true },
  "psCheckRevocationStatus": { "Bool": true },
  "psRetrieveFromAiaSiaHttp": { "Bool": true }
}
```

Saving this file changes nothing on its own — that is the point of including it. It is the baseline
the other two are edits of.

One behaviour is **not** in this file and cannot be: whether path building chases AIA and SIA URIs
after exhausting the graph it was given. That is an argument rather than a setting —
`--dynamic-build` (`-y`) on the command line, the corresponding control in the desktop and browser
applications. `psRetrieveFromAiaSiaHttp` above governs whether retrieval is permitted when chasing
happens; the argument governs whether it happens at all. Standard plus `-y` is graph-first,
then-chase.

## Offline

Judge a certificate with the material you were given. Revocation is still considered — a CRL you
supply, or an OCSP response stapled into the run, is used exactly as before — but nothing is
fetched, so a run makes no network requests at all.

```json
{
  "psCheckCrlDpHttp": { "Bool": false },
  "psCheckCrls": { "Bool": true },
  "psCheckOcspFromAia": { "Bool": false },
  "psCheckRevocationStatus": { "Bool": true },
  "psRetrieveFromAiaSiaHttp": { "Bool": false }
}
```

Expect more results of *revocation status could not be determined* than with Standard: that outcome
is the honest one when the only way to learn a status was to go and ask. Pair this profile with
supplied revocation material — the `--rev` argument, or the upload slots in the applications — when
a determination is wanted without a network.

## Strict

Tighter than the defaults in two directions: certificate policy processing, and how fresh revocation
information has to be.

```json
{
  "psCheckCrlDpHttp": { "Bool": true },
  "psCheckCrls": { "Bool": true },
  "psCheckOcspFromAia": { "Bool": true },
  "psCheckRevocationStatus": { "Bool": true },
  "psCrlGracePeriodsAsLastResort": { "Bool": false },
  "psEnforceTrustAnchorConstraints": { "Bool": true },
  "psForbidSelfSignedEE": { "Bool": true },
  "psInitialExplicitPolicyIndicator": { "Bool": true },
  "psInitialInhibitAnyPolicyIndicator": { "Bool": true },
  "psInitialPolicyMappingInhibitIndicator": { "Bool": true },
  "psOcspAiaNonceSetting": { "OcspNonceSetting": "SendNonceRequireMatch" },
  "psRevocationMaxAge": { "Duration": { "secs": 86400, "nanos": 0 } }
}
```

What each line buys:

| setting | effect |
|---|---|
| `psInitialExplicitPolicyIndicator` | a path must end with a non-empty policy set; a certificate chain that satisfies 5280 but carries no acceptable policy is rejected |
| `psInitialInhibitAnyPolicyIndicator` | `anyPolicy` no longer satisfies a policy requirement |
| `psInitialPolicyMappingInhibitIndicator` | policy mappings are not followed, so a policy must be asserted rather than mapped into |
| `psEnforceTrustAnchorConstraints` | RFC 5937 constraints carried by the anchor are applied to the path |
| `psForbidSelfSignedEE` | a self-signed end entity is rejected rather than validated as its own island |
| `psCrlGracePeriodsAsLastResort` | a CRL past its next-update is no longer accepted as a last resort |
| `psRevocationMaxAge` | revocation information older than a day is not used, however it arrived |
| `psOcspAiaNonceSetting` | a nonce is sent and a response that does not echo it is rejected |

Each of these makes some certificates that Standard accepts fail, which is the intent. The policy
settings in particular are strict in the RFC 5280 sense rather than the colloquial one: they change
what a valid path *is*, not just how hard the run looks.

## A note on the browser

The browser application cannot fetch on its own — retrieval goes through a PITTv3 service acting as
a relay, and a deployment without one has no way to chase AIA or SIA, or to collect a CRL from a
distribution point. Standard and Strict therefore describe intent there rather than behaviour: the
settings are honoured where retrieval is possible, and the browser reports what it could not do
rather than pretending otherwise.
