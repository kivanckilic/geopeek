# GEOpeek — phone app (PWA, Android first)

Installable web app: GPS position → azimuth, elevation, LNB skew, polar-mount angles to any GEO satellite,
plus a phone-compass / tilt pointing aid. Works offline after the first load.

## How it gets its data
| What | Where from | When |
|---|---|---|
| GEO element sets (NORAD TLE) | CelesTrak `GROUP=geo`, fetched by the GitHub Action and published as `data/geo.tle` next to the app | once a day (02:17 UTC) + on every push |
| Site latitude / longitude | phone GPS (`watchPosition`, high accuracy) | live |
| Site height | GPS altitude if its accuracy is ≤ 30 m, otherwise terrain height (Copernicus DEM GLO-90 via Open-Meteo) | on each fix (DEM re-queried only after ~60 m movement) |
| Another location (optional) | place search (Open-Meteo geocoding), typed coordinates (decimal or DMS) or saved sites | on demand |
| Magnetic declination | WMM2025, computed on the phone | live |

Because the TLE file is served from the same site as the app, there is no CORS problem and CelesTrak is
hit once per day by GitHub, not by every phone.

## Deploy (GitHub Pages, ~5 minutes, free)
1. Create a new GitHub repository (public, or private on a plan that supports Pages), e.g. `look-angles`.
2. Rename `_github` to `.github` if it is named that way here, then upload **all** files from this folder, including `.github/workflows/deploy.yml`
   (web UI: *Add file → Upload files*, drag the folder contents; or `git push`).
3. Repository **Settings → Pages → Build and deployment → Source: GitHub Actions**.
4. **Actions** tab → *Deploy app + daily NORAD TLE* → **Run workflow** (first run).
5. The app is live at `https://<user>.github.io/<repo>/`.

## Install on the phone
* **Android (Chrome):** open the URL → menu ⋮ → *Install app* / *Add to Home screen*.
* **iPhone (Safari):** open the URL → Share → *Add to Home Screen*.
* Allow **Location**. For the pointing aid, tap *Start pointing aid* and allow **Motion & Orientation** (iOS asks once).

## Location: My location / Another location
* **Settings → My location:** live GPS position of the phone.
* **Settings → Another location:** search a place name, type `lat, lon`, or pick a saved site. Height comes from the place search
  or *Get terrain height*. *Save this location* stores named sites on the phone (also works for the current GPS position,
  e.g. to save a station site). Typing coordinates switches to Another location automatically.

## Pointing aid
* Hold the phone **upright** and aim the **back camera** along the dish boresight direction: the app shows
  *Turn left/right* and *Tilt up/down* to the target; it turns mint and vibrates (Android) within ±2° az / ±1° el.
* Phone **flat**: shows the heading of the phone's top edge (azimuth only).
* Phone compasses read magnetic north; the app adds the WMM2025 declination, so all targets are compared in true azimuth.
* Accuracy is typically ±5° and much worse next to the dish, mast or a car. Use it for coarse alignment,
  then peak on signal (spectrum analyser / satfinder / modem Es/N0).

## Notes
* GitHub disables scheduled workflows after 60 days without repository activity; re-enable them from the
  Actions tab (or push any commit).
* If CelesTrak refuses a download (HTTP 403, e.g. a blocked runner IP), the workflow fails **before**
  deploying, so the previous day's data stays online. Rerun later from the Actions tab.
* Without the Action (e.g. app copied to another web server), the app tries CelesTrak directly at most once
  per 24 h; you can always load a TLE file from *Settings → Load TLE file*.
* Skew sign: + = rotate LNB clockwise as seen from behind the dish looking at the satellite.
* Want a Play Store / sideload APK? Point https://www.pwabuilder.com at the deployed URL to package this PWA.
