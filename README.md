# PITTv3 User's Guide

Source for the PITTv3 manual, built with [mdBook](https://rust-lang.github.io/mdBook/).

**Read it here:** <https://pittv3.redhoundsoftware.com/pittv3-book/>

**Offline:** <https://pittv3.redhoundsoftware.com/pittv3-book/pittv3-book.pdf> — one file, for the
disconnected case the desktop build exists to serve.

The guide covers all of PITTv3's frontends: the command line, the desktop application, the browser
application, and the service the browser reaches repositories through. It is hosted beside the
browser application at the same origin, so the link the application offers adds no second host to a
page whose point is that nothing leaves it.

## Building

```text
mdbook build      # renders to book/
mdbook serve      # renders and watches, at http://localhost:3000
```

The PDF is produced by `mdbook-pdf` from the same sources, so the two artifacts cannot disagree.

## Chapters

`src/SUMMARY.md` is the table of contents and the list mdBook builds from; a chapter not named there
is not in the book. Chapters are numbered by the order they are read in rather than by topic, so
inserting one means renaming its neighbours.
