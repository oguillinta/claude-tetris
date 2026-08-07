---
name: weather
description: Look up current local weather conditions for a city via web search. Trigger this whenever the user asks about the weather, temperature, forecast, or conditions ("how hot is it", "check the weather", "what's the temperature in X", "/weather"), even if they don't name a city — default to Lima, Peru, the user's home city. Also trigger for recurring/scheduled weather checks set up via /loop or /schedule.
---

# Weather

Report current weather conditions for a city using web search — there is no weather API configured for this project, so live data always comes from a search.

## Steps

1. **Determine the city.** If the user names one, use it. If they don't, default to **Lima, Peru** (the user's home city) rather than asking.
2. **Search** with `WebSearch` using a query like `current weather <city> now` (include "now" or "today" — it steers results toward live conditions instead of stale cached pages).
3. **Extract and report**, in this order, using whatever the sources actually provide (omit fields not reported rather than guessing):
   - Condition (e.g. cloudy, clear, rain)
   - Temperature (°F and °C if both are available)
   - Feels-like temperature, if different from actual
   - Humidity
   - Wind speed
4. **Cite sources** as markdown links beneath the answer, per `WebSearch`'s own citation requirement.

## Notes

- Weather sources often disagree slightly or show mixed timestamps (e.g. AccuWeather vs. Weather Underground). Prefer the most consistent/recent-looking figures across 2-3 results rather than quoting just the first hit; don't dwell on discrepancies, just report the best estimate.
- If this is invoked repeatedly on a short interval (e.g. every minute via `/loop`), you don't need to re-explain the methodology each time — just give the fresh reading. Temperature rarely changes meaningfully minute-to-minute, so if the user sets up a very frequent recurring check, it's worth a brief one-line note suggesting a longer interval (e.g. hourly), without refusing the request.
