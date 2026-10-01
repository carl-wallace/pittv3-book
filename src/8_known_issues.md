# Limitations and known issues

## Deliberate limitations

These are choices, not gaps waiting to be filled.

**Delta CRLs are not supported.** They are neither indexed nor considered. A scope requiring one
fails closed rather than being approximated, because approximating revocation is worse than
declining to answer.

**FN-DSA is not verified.** The identifiers are declared; verification waits on a FIPS 206
implementation.

**The browser gathers revocation data before validating rather than while it validates.** certval
tries sources in a fixed order and stops at the first that answers — a cached determination, a
no-check extension, stapled data, local CRLs, OCSP from the AIA, then a remote distribution point. A
page cannot fetch from inside that walk, so the browser works out everything the paths could need,
retrieves it, and supplies it as local data before validation begins. The consequence is that it may
retrieve an artifact a command-line run would never have asked for, having been answered earlier in
the order — which is why a browser run has a retrieval budget and a native one does not.

**This is not a TLS verifier.** Web PKI-specific behavior — name matching rules, policy
requirements particular to the CA/Browser Forum baseline — is out of scope by design. PITTv3
validates certification paths per RFC 5280 and RFC 5937; it does not decide whether a certificate is
acceptable for a TLS connection.

## Things worth knowing

**A CRL folder is written as well as read.** CRLs fetched during a run, and the last-modified map
that makes later fetches conditional, are saved into `--crl-folder`. Indexing only skips a CRL that
does not cover the time of interest, and nothing on the command line removes one. The desktop's
*Remove stale* button on the CRL index does remove them, and a superseded CRL generally cannot be
fetched again, so using it forecloses validating as of a time it covered.

**Cleanup moves or deletes, depending on one field.** `--cleanup` moves certificates that cannot
contribute to the folder `--error-folder` names; with no error folder they are deleted. Use
`--report-only` first. The applications do not clean folders — what corresponds there is marking
rows in the Inspect tables, which writes a new store and leaves the one it read alone.

**A store does not know about certificates added after it was built.** Partial paths are computed
when the store is generated; adding certificates to a CA folder afterwards does not change an
existing store. Regenerate it.

**A time of interest of `0` disables validity period checks** rather than meaning the epoch.

**Setting a time of interest is almost always wrong** unless you mean to validate relative to a past
moment — code signing, or replaying an archived run. A past time also discards revocation data
published since, which is correct and reads as a changed verdict.

**A browser on iOS or iPadOS cannot select a folder.** Safari there does not honor the folder
mode other browsers offer, so certificates are chosen file by file; a set of several hundred is a
long selection rather than one pick. Nothing else differs — large sets do validate from an iPad —
and the desktop application takes folders through the platform's own dialogs.

**Windows machine stores need elevation to read.** The library asks for write access when opening a
CAPI store, because the wrapper it uses offers no read-only open, so `LocalMachine` fails for an
unelevated process even when only reading. Windows itself does not require that; the constraint is
ours and is being addressed upstream.

## Known issues

TODO — to be filled from the issue tracker at release time.
