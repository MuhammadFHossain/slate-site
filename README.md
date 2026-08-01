# Slate, the website

The site for **Slate**, at **slatedictation.com**. Plain static HTML and CSS.
No build step, no framework, no tracking, and nothing loaded from another
company's servers. The download sits next to `index.html`, so the button is a
relative link.

```
index.html          the landing page
support.html        setup, troubleshooting, uninstall
privacy.html        privacy policy for the app and this site
terms.html          terms of use
404.html            what GitHub Pages serves for a bad address; its links are
                    root-absolute, since it is served from any depth
Slate-1.1.1.dmg     the app itself, what the download button points at.
                    Held back by .gitignore until it is notarized, see below
appcast.xml         the update feed Sparkle in the app reads
CNAME               slatedictation.com, used by GitHub Pages
.nojekyll           stops GitHub rewriting the folder
robots.txt, sitemap.xml
og-render.html      source for assets/img/og.png; kept in the repo so the
                    card can be re-rendered, not linked from anywhere
assets/
  styles.css        the whole design system
  fonts/            Newsreader, bundled, SIL Open Font Licence
  img/              favicon, apple touch icon, og.png
```

---

## The DMG here is notarized — keep it that way

The `Slate-1.1.1.dmg` in this folder came out of
`~/Desktop/Blue/Slate/scripts/notarize.sh` on 31 July 2026: Developer ID
signed, notarized by Apple, ticket stapled. It is committed on purpose —
GitHub Pages serves whatever is in the repo, so the download button needs the
file here. Only ever replace it with a build that passes both checks:

```bash
xcrun stapler validate Slate-1.1.1.dmg
spctl -a -t open --context context:primary-signature -vv Slate-1.1.1.dmg
```

A development-signed build would be refused by Gatekeeper on every Mac but
this one, and git history keeps whatever is committed forever. The notary
one-time setup (Developer ID certificate, `slate-notary` keychain profile)
is already done on this Mac; per release, `scripts/notarize.sh` does the
whole job and prints the appcast enclosure values at the end.

---

## Putting it online (GitHub Pages)

The repo is <https://github.com/MuhammadFHossain/slate-site>.

1. **Settings &rsaquo; Pages &rsaquo; Source: Deploy from a branch**, branch `main`,
   folder `/ (root)`. Save.
2. Under **Custom domain**, type `slatedictation.com` and save. The `CNAME`
   file already holds it, so this should match straight away.
3. At the registrar for `slatedictation.com`, point the apex `A` records at
   GitHub's four addresses:
   ```
   185.199.108.153
   185.199.109.153
   185.199.110.153
   185.199.111.153
   ```
   and add a `CNAME` record for `www` pointing at `muhammadfhossain.github.io`.
4. Wait for the DNS check to go green, then tick **Enforce HTTPS**.

DNS can take anything from ten minutes to a day. There is no github.io preview
while you wait: the `CNAME` file makes Pages redirect
`muhammadfhossain.github.io/slate-site/` straight to `slatedictation.com`. To
look at the built site before DNS resolves, either delete `CNAME` for the first
deploy and add it back after, or keep using the local server.

If you would rather not use Pages, this folder works as is on Cloudflare Pages
(Workers &amp; Pages &rsaquo; Create &rsaquo; Pages &rsaquo; Upload assets) or on
Netlify Drop. All three give free HTTPS. Nothing in the site depends on the
host.

---

## Shipping a new version of the app

This is the whole release, start to finish.

**1. Build and notarize.**

```bash
cd ~/Desktop/Blue/Slate
scripts/notarize.sh
```

Bump `MARKETING_VERSION` and `CURRENT_PROJECT_VERSION` in `project.yml` first.
`CURRENT_PROJECT_VERSION` is the number Sparkle actually compares, so it has to
go up every single release.

**2. Sign the DMG for the update feed.**

```bash
SPARKLE_BIN="$HOME/Library/Developer/Xcode/DerivedData/$(ls ~/Library/Developer/Xcode/DerivedData | grep '^Slate-' | head -1)/SourcePackages/artifacts/sparkle/Sparkle/bin"
"$SPARKLE_BIN/sign_update" ~/Desktop/Blue/Slate/dist/Slate-<version>.dmg
```

The keychain asks for permission the first time. Choose **Always Allow** so it
does not ask again. It prints something like:

```
sparkle:edSignature="…" length="7879440"
```

**3. Put the new build in this folder.**

```bash
cd ~/Desktop/Blue/slate-site
rm Slate-<old>.dmg
cp ~/Desktop/Blue/Slate/dist/Slate-<new>.dmg .
```

**4. Update three lines in `index.html`.** Search for `RELEASE-PIN`; there are
exactly three, all commented:

- the hero download link
- the download-section link
- the version and size line

**5. Add an entry at the top of `appcast.xml`.** Copy the existing `<item>` and
change: `<title>`, `<pubDate>`, `<sparkle:version>` (the build number),
`<sparkle:shortVersionString>`, the release notes, and in `<enclosure>` the
`url`, the `length` and the `sparkle:edSignature` from step 2. Leave the old
items in place; Sparkle picks the newest one it can run.

**6. Commit and push.** GitHub Pages redeploys on its own.

```bash
git add -A && git commit -m "Slate <version>" && git push
```

**7. Check the signature before you push.** The in-app test only works from
one Sparkle release to the next, so for 1.1.1 check it directly:

```bash
"$SPARKLE_BIN/sign_update" --verify Slate-1.1.1.dmg "<the edSignature in appcast.xml>"
```

From the next release on, the real test is a Mac running the previous version:
Settings &rsaquo; About &rsaquo; **Check now** should offer the new one, download
it, and restart into it.

### The signing key

The private half of the update key lives in your login keychain, made by
Sparkle's `generate_keys`. The public half is in Slate's `Info.plist` as
`SUPublicEDKey`:

```
+uQooF3bNvSLpz+jQ11WXJrBr3hKh4Ry5odi0CD8JAM=
```

**Back the private key up somewhere safe.** If it is lost, nobody who already
has Slate can ever be updated again; they would all have to download the app by
hand. To export a copy:

```bash
"$SPARKLE_BIN/generate_keys" -x slate-update-key.txt
```

Keep that file out of this repo and out of anything synced publicly.

---

## Changing things

**The tip link.** Search `index.html` for `TIP-LINK`. There is one comment
telling you exactly what to wrap. Put the same URL in
`~/Desktop/Blue/Slate/Slate/Support/Tips.swift`, which is a single `nil` waiting
for it, and the tip button in Settings &rsaquo; About turns on by itself.

**Prices.** The site says Slate is free and invites a tip. If that ever becomes
a purchase, the places to change are: the chip and the tip line in the download
section, the *What does it cost?* question, the price row in the comparison
table, and sections 4 and 5 of `terms.html`.

**The stylesheet is cache-busted** with `?v=YYYYMMDD` on every page. Bump that
when you change `assets/styles.css`, or people with the page cached keep the old
one.

**The two easter eggs.** Hovering the hero download button makes its label
arrive a word at a time, at the speed someone talks, with the waveform moving.
Hovering the one at the bottom types its label in at a cursor, the way Slate
puts text into an app. Both reserve the finished label's width first, so the
buttons never change size, and the link's accessible name never changes. Both
stand down under `prefers-reduced-motion`.

**The hero demo** is the one bit of real scripting. The take it plays,
`Um, meet me at nine. Actually, meet me at ten. New paragraph. Email team at
example dot com.`, was checked against the shipping cleanup engine:

```bash
/Applications/Slate.app/Contents/MacOS/Slate --refine "Um, meet me at nine. Actually, meet me at ten. New paragraph. Email team at example dot com."
```

which prints exactly the two paragraphs the document shows. If you change the
words in the demo, run that again and make the document match, or the page is
claiming something the app does not do.

---

## House rules for the copy

Taken from `~/Desktop/Blue/Slate/docs/WEBSITE-HANDOFF.md`, which is the real
brief.

- Plain words, short sentences, grade 3 to 5.
- No em dashes. Periods and commas do the job.
- Never give the app a personality. "Slate is in the menu bar", not "Slate
  lives in your menu bar".
- No marketing slop: seamless, effortless, unlock, elevate, empower,
  supercharge, game-changer, magical, "just works", "say goodbye to".
- Every claim has to be one that can be shown. The measured ones are: speech
  runs on the Neural Engine using Parakeet, a short sentence comes back in well
  under a second, zero network connections during dictation (checked with
  `lsof`), the model download is about 600 MB, 0% CPU and about 73 MB idle, the
  clipboard survives insertion, music pauses and resumes.
- No testimonials, no invented numbers, no fake urgency.

## Notes

- Newsreader is under the SIL Open Font Licence, so it ships with the site.
- The site sets no cookies and runs no analytics. GitHub Pages keeps its own
  server logs, which `privacy.html` says plainly.
- Light theme only, on purpose. The product is a light, papery design and there
  is no dark mode to match.
