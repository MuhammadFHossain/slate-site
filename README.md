# Slate, the website

The site for **Slate**, at **slatedictation.com**. Plain static HTML and CSS.
No build step, no framework, no tracking, and nothing loaded from another
company's servers. The download sits next to `index.html`, so the button is a
relative link.

Slate is sold by **Blue Tomato Studio LLC** for **$27, once**. Lemon Squeezy is
the merchant of record: it takes the payment, handles sales tax and VAT, and
emails the receipt and the license key. The site never loads anything from
Lemon Squeezy. Its Buy buttons are plain links to the checkout.

The look follows `DESIGN.md` in the app repo: one emerald accent, glass cards,
a black island grown out of the notch as the hero object, system rounded type,
light or dark with the system. Where that file and the older
`docs/WEBSITE-HANDOFF.md` disagree, the design spec wins on looks and the
handoff still wins on voice.

The two repos sit side by side:

```
~/Desktop/Blue Tomato Studio/Slate/Slate        the app
~/Desktop/Blue Tomato Studio/Slate/slate-site   this site
```

The folder names have spaces, so quote them in the shell.

```
index.html          the landing page: hero, pricing, download, and the FAQ
support.html        setup, the trial and license keys, moving Macs, refunds,
                    uninstalling
privacy.html        privacy policy for the app and this site
terms.html          terms of sale and use, including the refund promise
404.html            what GitHub Pages serves for a bad address; its links are
                    root-absolute, since it is served from any depth
Slate-<version>.dmg every build shipped so far. The newest is what the
                    download buttons point at; the older ones stay on
                    purpose, and .gitignore says why
appcast.xml         the update feed Sparkle in the app reads
CNAME               slatedictation.com, used by GitHub Pages
.nojekyll           stops GitHub rewriting the folder
robots.txt          also keeps this README out of search results
sitemap.xml         one <lastmod> per page; change it when the page changes
og-render.html      source for assets/img/og.png, the card shown when someone
                    shares a link; not linked from anywhere
store-render.html   source for assets/img/store-1mac.png, the product image
                    on the Lemon Squeezy checkout; not linked from anywhere
assets/
  styles.css        the whole design system, tokens at the top, dark mode below them
  img/              favicon, apple touch icon, og.png, store-1mac.png
```

---

## The DMG here is notarized. Keep it that way.

Every DMG in this folder came out of the app repo's `scripts/notarize.sh`:
Developer ID signed, notarized by Apple, ticket stapled. It is committed on
purpose. GitHub Pages serves whatever is in the repo, so the download button
needs the file here. Only ever replace it with a build that passes both checks:

```bash
xcrun stapler validate Slate-<version>.dmg
spctl -a -t open --context context:primary-signature -vv Slate-<version>.dmg
```

A development-signed build would be refused by Gatekeeper on every Mac but
this one, and git history keeps whatever is committed forever. The notary
one-time setup (Developer ID certificate, `slate-notary` keychain profile)
is already done on this Mac; per release, `scripts/notarize.sh` does the
whole job and prints the appcast enclosure values at the end.

Older DMGs stay. Every file here is a live download, and a build from before
the paid release never asks for a key; that is a known, accepted trade-off
(decided 2026-09-24), not an oversight.

---

## Selling Slate

### The offer

Said the same way on every page, in the app, and in the terms:

- **14-day free trial** with every feature. No key, no card, no account. The
  14 days start the first time Slate is opened. After them, dictation stops
  until a key is entered. Nothing else is deleted.
- **$27 once**, in US dollars. No subscription, no account.
- **One Mac.** One license is one person on one Mac (activation limit 1 in
  Lemon Squeezy). To move, Settings &rsaquo; License &rsaquo;
  **Deactivate This Mac&hellip;** on the old Mac, then activate on the new
  one. If the old Mac is lost or wiped, the customer emails and the seat is
  freed from the store dashboard.
- **30-day refund**, no questions asked, by email.
- **Updates:** $27 buys Slate 2 and every Slate 2 update. A Slate 3 may be a
  paid upgrade. Never promise lifetime updates.
- **Founder license:** anyone who opened Slate before the paid version was
  released keeps it free, automatically, with no key. Downloading alone does
  not count. It comes from the trial record the app keeps on that Mac, which
  is why the paid release has to set its opening date (see below).
- **Lost key:** it is in the Lemon Squeezy purchase email, and
  <https://app.lemonsqueezy.com/my-orders> sends it again.
- **Support:** contact@bluetomatostudio.com.

### Where the offer is written

If the price, the trial, the refund window, or the seat count ever changes,
change every one of these in the same commit:

- `index.html`: the meta descriptions in the head, the hero offer line and
  Buy button, the Price row of the comparison table, the pricing section, the
  download section's pill, lead, and offer line, and the FAQ answers with the
  ids `cost`, `trial`, `founders`, `refunds`, `new-mac`, `buying`, and
  `updates`. The comment at the top of the page lists the same.
- `og-render.html`: the chips. Then re-render `assets/img/og.png` with the
  command in that file's own comment and look at the result. The PNG is what
  gets served, so editing the HTML alone changes nothing.
- `store-render.html`: the same, for the checkout image, which then has to be
  uploaded to the product in Lemon Squeezy by hand.
- `support.html`, `terms.html`, and `privacy.html`.
- The app itself: Settings &rsaquo; License and the trial messages, in the app
  repo.

### Before the paid site goes live

**The checkout link.** Until the store is live, every Buy button on the site
points at the placeholder `LEMONSQUEEZY_CHECKOUT_URL`, with the comment
`<!-- CHECKOUT-PIN -->` on the same line. This finds all of them, on every
page:

```bash
grep -rn --include='*.html' CHECKOUT-PIN .
```

Replace each placeholder with the checkout link of the one Slate product, then
check none is left. This must print nothing:

```bash
grep -rn --include='*.html' LEMONSQUEEZY_CHECKOUT_URL .
```

Link the checkout directly. Do not add `lemon.js` or any other script for it:
this site loads nothing from another company's servers, and privacy.html
says so.

**The governing law** is settled: Blue Tomato Studio LLC is a California LLC,
and section 18 of `terms.html` names California. Change it only if the
company moves.

**Ship the site and the app on the same day.** The site may not say $27 before
the store can take money, and the founder promise only holds once the app
release that switches the store on is out, with its opening date set in
`Licensing.swift` (`storeConfigured`, `buyURL`, `storeOpenedAt`, `storeID` and
`productID`, all in one commit). That release's DMG becomes the one the
download buttons point at, and the free ones stay beside it.
`docs/DEPLOYMENT.md` in the app repo has the full order.

### The checkout image

`store-render.html` is the source for the product image on the checkout, the
same pattern as `og-render.html`. Slate is sold one way, so there is one image.
The render command is in the file's own comment. Upload the PNG to the product
in Lemon Squeezy. Nothing on the site links to it.

---

## Putting it online (GitHub Pages)

The repo is <https://github.com/MuhammadFHossain/slate-site>. Pages serves the
`main` branch.

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
deploy and add it back after, or keep using a local server.

If you would rather not use Pages, this folder works as is on Cloudflare Pages
(Workers &amp; Pages &rsaquo; Create &rsaquo; Pages &rsaquo; Upload assets) or on
Netlify Drop. All three give free HTTPS. Nothing in the site depends on the
host.

---

## Shipping a new version of the app

This is the whole release, start to finish. `docs/DEPLOYMENT.md` in the app
repo is the longer version.

**1. Build and notarize.**

```bash
cd "$HOME/Desktop/Blue Tomato Studio/Slate/Slate"
scripts/notarize.sh
```

Bump `MARKETING_VERSION` and `CURRENT_PROJECT_VERSION` in `project.yml` first.
`CURRENT_PROJECT_VERSION` is the number Sparkle actually compares, so it has to
go up every single release.

**2. Sign the DMG for the update feed.**

```bash
SPARKLE_BIN="$HOME/Library/Developer/Xcode/DerivedData/$(ls ~/Library/Developer/Xcode/DerivedData | grep '^Slate-' | head -1)/SourcePackages/artifacts/sparkle/Sparkle/bin"
"$SPARKLE_BIN/sign_update" "$HOME/Desktop/Blue Tomato Studio/Slate/Slate/dist/Slate-<version>.dmg"
```

The keychain asks for permission the first time. Choose **Always Allow** so it
does not ask again. It prints something like:

```
sparkle:edSignature="…" length="7879440"
```

**3. Add the build to this folder.** Older DMGs stay where they are.

```bash
cd "$HOME/Desktop/Blue Tomato Studio/Slate/slate-site"
cp "$HOME/Desktop/Blue Tomato Studio/Slate/Slate/dist/Slate-<new>.dmg" .
```

**4. Update five places in `index.html`.** Search for `RELEASE-PIN`; there are
exactly five, all commented:

- the version pill above the headline
- the hero download link
- the download-section link
- the version and size line
- the SHA-256 in the "Is the download safe?" answer (`shasum -a 256 Slate-<version>.dmg`)

**5. Add an entry at the top of `appcast.xml`.** Copy the newest `<item>` and
change: `<title>`, `<pubDate>`, `<sparkle:version>` (the build number),
`<sparkle:shortVersionString>`, the release notes, and in `<enclosure>` the
`url`, both version attributes, the `length` and the `sparkle:edSignature`
from step 2. Leave the old items in place; Sparkle picks the newest one it
can run.

**6. Check the signature before you push.**

```bash
"$SPARKLE_BIN/sign_update" --verify Slate-<version>.dmg "<the edSignature in appcast.xml>"
```

**7. Commit and push.** GitHub Pages redeploys on its own.

```bash
git add -A && git commit -m "Slate <version>" && git push
```

After it is live, the real test is a Mac running the previous version:
Settings &rsaquo; About &rsaquo; **Check now** should offer the new one,
download it, and restart into it.

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
`.gitignore` refuses it by name, but do not lean on that.

---

## Changing things

**The stylesheet is cache-busted** with the date and a letter, such as
`?v=20260924a`, on every page, the 404 included. Bump it on every page when you
change `assets/styles.css`, or people with the page cached keep the old one.
This finds every copy, and all of them should print the same value:

```bash
grep -rn --include='*.html' 'styles.css?v=' .
```

**FAQ links.** A question with an id opens when the address ends in that id,
so a support email can link straight to an answer, such as
`slatedictation.com/#refunds`. The ids are `cost`, `trial`, `founders`,
`refunds`, `new-mac`, `buying`, and `updates`. Renaming one breaks every email
that already links to it.

**The hero island** is drawn at the app's actual sizes: a 185 by 32 notch, a
436 wide body with 18 of padding, a 400 wide transcript column, concave 10 px
flares at the top corners, a 22 px bottom radius, and the emerald hairline
along the bottom. Its states are the app's own strings: Listening, Placing
your words, Placed in Notes. If the island changes in the app, change it here.

**Glass cards** are the `.glass` class. It carries the sheen, the light catch,
the emerald cast, the edge that goes white to emerald, and the specular line,
as one set of tokens at the top of the stylesheet with a dark-mode block under
them. Do not paint a card any other way.

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

Taken from `docs/WEBSITE-HANDOFF.md` in the app repo, which is the real brief.

- Plain words, short sentences, grade 3 to 5.
- No em dashes. Periods and commas do the job.
- Never give the app a personality. "Slate is in the menu bar", not "Slate
  lives in your menu bar".
- No marketing slop: seamless, effortless, unlock, elevate, empower,
  supercharge, game-changer, magical, "just works", "say goodbye to".
- Every claim has to be one that can be shown. The measured ones are: speech
  runs on the Neural Engine using Parakeet, a short sentence comes back in well
  under a second, zero network connections during dictation (checked with
  `lsof`), the model download is about 600 MB, 0% CPU and about 73 MB idle, and
  the clipboard survives insertion.
- Do not say music pauses while you talk. It did, but macOS 15.4 and later let
  only Apple's own apps see what is playing, so Slate leaves music alone
  there, which is any Mac kept up to date. It still works on macOS 14.4 to
  15.3, which is too narrow to promise on the page.
- Languages are counted as "25 European languages", never "25 languages".
- The network list is four things, in the same words on every page and in the
  app's Settings: downloading a speech model (from huggingface.co); checking or
  releasing a license key with the store (api.lemonsqueezy.com), only when
  Activate or Deactivate is pressed; asking slatedictation.com whether there is
  a new version, if that is switched on; and downloading that version if the
  person chooses to install it. None of it happens during dictation. Words are
  never sent anywhere.
- The offer is said one way everywhere (see *Selling Slate* above).
- No testimonials, no invented numbers, no fake urgency: no countdowns, no
  launch discounts, no user counts.

## Notes

- System faces only, like the app: SF Rounded for headlines and controls, SF
  Pro for running text. Nothing is downloaded.
- The site sets no cookies and runs no tracking script. It is hosted on GitHub
  Pages and served through Cloudflare (free plan, since 2026-11-14), whose
  request totals by path are how downloads, update checks (`appcast.xml`) and
  Arabic downloads (`/models/arabic-lite/`) are counted. Both keep server logs,
  which `privacy.html` says plainly. Cloudflare's SSL mode is Full, not Strict,
  on purpose: GitHub's own certificate may not renew behind the proxy.
- Light and dark, following the system, because the app does the same.
