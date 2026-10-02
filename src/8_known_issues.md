# Limitations and known issues

## Deliberate limitations

These are choices, not gaps waiting to be filled.

**Delta CRLs are not supported.** A delta CRL is never indexed or processed, and a freshestCRL
pointer is not followed, so status comes from the complete CRL alone. A certificate revoked since
that CRL was issued, and listed only on a delta, reads as good until the next complete CRL lists it
(PKITS 4.15.4 is that case). With no complete CRL available, status is not determined rather than
taken from a delta.

**CRLs scoped by onlySomeReasons are not used.** A CRL that covers only some revocation reasons
cannot show that a certificate is not revoked, since a revocation for another reason is listed
elsewhere, and coverage is not combined across such CRLs. They are discarded, so a certificate
whose only CRLs are partitioned by reason gets an undetermined status (PKITS 4.14.18 and 4.14.19
are that case).

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

**Windows machine stores need elevation to read.** The library opens CAPI stores through the
`schannel` crate, which has no way to open a store read-only, so it asks for write access even when
it only reads. An unelevated process is refused that access to `LocalMachine`, although Windows
itself lets any process read the machine stores.

## Known issues

TODO — to be filled from the issue tracker at release time.
