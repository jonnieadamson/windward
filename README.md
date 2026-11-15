# Windward: live test package

Pre-round wind, rain and club-difference planner. This folder is the live-test build: it fetches a real forecast and real course shapes whenever it has an internet connection, and falls back to a labelled demo when it doesn't.

## 1. Fastest way to test on an Android phone (about 5 minutes, no coding)

The app is a Progressive Web App, so it installs from Chrome like a normal app.

1. Put the contents of this folder on any HTTPS web host. Easy free options: Netlify Drop (drag the folder onto app.netlify.com/drop), GitHub Pages, or Cloudflare Pages.
2. Open the address in Chrome on the phone.
3. Chrome menu (three dots) > **Install app** (or **Add to Home screen**).
4. Open it from the home screen. It runs full-screen, remembers your bag and settings, and keeps working offline with the last forecast it loaded.

It must be HTTPS (or localhost) for "Use my location" and installation to work.

## 2. Optional: build an Android app file (APK/AAB)

Needs Node.js and Android Studio. Not tested by the author, so allow time for small fixes.

```
mkdir windward-app && cd windward-app
npm init -y
npm i @capacitor/core @capacitor/cli @capacitor/android @capacitor/geolocation
npx cap init Windward com.yourname.windward --web-dir=www
mkdir www && cp ../windward-live/{index.html,manifest.webmanifest,sw.js,icon-*.png} www/
npx cap add android
npx cap sync
npx cap open android      # Build > Build APK(s) in Android Studio
```

If "Use my location" does not prompt on the phone, add `<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />` to `android/app/src/main/AndroidManifest.xml`.

## 3. What is live and what is not

| Feature | Source | Status |
|---|---|---|
| Wind, gusts, rain, temperature (16 days) | Open-Meteo forecast API, 10 m wind | Live. Falls back to a labelled demo forecast offline. |
| Course hole lines, greens, bunkers, water | OpenStreetMap via Overpass | Live. Falls back to an illustrative layout. Cached on the device for 30 days. |
| Hole elevation change | Open-Meteo elevation API | Live. |
| Weather warnings | Derived from forecast gusts, rain, frost and heat, plus a link to the official source (Met Office, Met Éireann, NWS) | Derived, not the official feed. |
| Sunrise and sunset | Calculated on the device | Approximate (a few minutes). |
| Practice facilities, prices, accessibility | Only the club's or organisation's own published pages, each with a source link | One course so far (St Andrews Links). All others show "Not verified yet". |
| Club review quotes | Hand-picked, six brands, each linked to the full review | Static. Citation list with titles, URLs and retrieval date, plus a Copy citations button. |

## 4. Test checklist

1. Pick Old Course, St Andrews. The badges should say **Live forecast** and **Real course layout**. If either says Demo or Illustrative, check the phone's connection.
2. Compare the wind with the Met Office or BBC forecast for the same place and time. Note any big differences.
3. Check hole directions on the Holes page against a satellite view. Note any hole where the tee/green line is wrong or two courses have been mixed (St Andrews has several courses side by side).
4. Flick through the holes (buttons and swipe). The side panel should change with the wind and your bag.
5. Switch **Competition mode** on: every wind figure, arrow and club suggestion should disappear.
6. Try other courses (Scotland, Ireland, USA), "Use my location", a date 10 days ahead and a date 20 days ahead (should show the demo notice).
7. Enter a few clubs with brands, tap **Rate my bag**.
8. Try airplane mode after loading: it should still open and say demo/cached.

## 5. Things to settle before a public release

- **Open-Meteo** is free for non-commercial use only. A public release needs their commercial plan, or switch to another provider (Met Office DataHub, Tomorrow.io and others). Attribution ("Weather data by Open-Meteo.com") is already shown.
- **OpenStreetMap / Overpass**: data is © OpenStreetMap contributors (ODbL); credit is shown. The free public Overpass servers are for light use, so a release needs your own mirror or a paid host.
- **Review quotes** in Rate my bag are short extracts with links. Get permission or take legal advice before shipping them.
- **Rules of Golf**: Competition mode is a best-effort aid. Committees may add stricter local rules. The player stays responsible.
- **Privacy**: bag, settings and location choices stay on the device. The course's coordinates (not the user's own location) are sent to Open-Meteo and Overpass. A Play Store listing will still need a privacy policy.
- **Warnings**: link to, or integrate, the official warning feeds rather than relying on derived thresholds.

## 6. Feedback button

The app has a **Send feedback** page (home screen, Settings, and a "Layout looks wrong?" link on the Holes page). Testers pick a topic, describe what they saw and tick whether to include diagnostics (course, date, forecast and layout status, settings, device; never their location).

- To receive feedback by email, open `index.html` and set `const FEEDBACK_EMAIL="you@example.com";`. The button then opens the tester's email app with the message ready.
- While it is empty, the button opens the phone's share sheet (email, WhatsApp, Messages and so on), or copies the message if sharing isn't available.

## 7. Adding practice facility data

Practice data is deliberately limited to what the club or organisation publishes itself. In `index.html`, find `const PRACTICE={` and add an entry for the course id with:

- `site` and `siteName`: the club's own website,
- `checked`: the date you checked,
- `items`: each with a name, a description in your own words and a `src` link to the exact page or PDF on the club's or organisation's site.

Items without a `src` are never shown. Courses with no entry show "Not verified yet" with a button that sends the tester's details to you through the feedback page. Prices and hours change, so re-check entries regularly.

## 8. Notes on the review quotes

Quotes are under 15 words each, one per review page, attributed and linked. The titles for some pages were taken from their web addresses, so check them against the live pages before publishing. Seek permission or legal advice before using review text in a released app.

## 9. If a map doesn't appear

The maps are now drawn as vector graphics (SVG) rather than on a canvas, so they work in every browser and WebView. If something still goes wrong the app says so instead of showing a blank box.

- **Badge above the hole map** shows the state of the real course data: "Real course layout" (working), "Loading real course layout…", or "Illustrative layout. Real course data unavailable (reason)". Typical reasons: "Failed to fetch" (no internet or the browser blocked the request), "HTTP 429" (the free map server is busy; wait a minute and reload), "Too few holes are mapped for this course" (OpenStreetMap has little data for it).
- **"This map couldn't be drawn"** with a reason and a **Report this** button means a drawing error. Tap the button; the feedback page includes the error text in its diagnostics.
- **Forecast badge** shows "Live forecast", "Loading live forecast…" or "Demo forecast".
- Real course shapes need internet and a normal web address (https). They will not load in the Claude preview page, which blocks outside requests; the preview shows the illustrative layout.
- The app reloads from a cache. After updating files, close and reopen the app (or clear the site's data in Chrome) to be sure you have the new version.
- To see errors on an Android phone: connect it by USB, turn on USB debugging, and open chrome://inspect on a computer.
