---
name: pocket-genie
description: Turn any itinerary, booking list or rough travel plan into a calm, glanceable, in-the-moment trip companion - a single-file mobile web app (a "Pocket 神灯", a pocket genie lamp) with a live Now/Next hero, a timeline that only puts a clock on the things that are truly booked, a failure plan for every stop, show-the-driver address cards, and honest verified-versus-unverified facts. Built to reduce travel anxiety, not to list attractions. Use this skill whenever the user wants a trip app, travel companion, day-of itinerary page, "what do I do right now" guide, a city guide for one specific trip, or wants to turn a plan, emails or bookings into a phone-friendly page - even if they never say "app" or "genie" and only say things like "make my trip plan usable on my phone", "I'm nervous about landing in a new city", "make a one-day itinerary I can follow", or "turn these bookings into a timeline".
---

# Pocket 神灯 (pocket-genie)

A map gives you routes. It does not tell you what the street will feel like, what to do when the train is cancelled, or which door is the real hotel entrance. A Pocket 神灯 (a pocket genie lamp) is the missing layer: one phone-sized page you can ask at any moment - where am I, what is next, what if it goes wrong - and always get a calm answer, so the traveler can stop worrying and look up at the city.

The product is **editorial judgment plus calm structure**. It is not 3D, not a booking engine, not a list of "top 10 things to do". If the page would read the same for any visitor, it has failed; it should read like a well-informed friend wrote it for this person on this day.

## When to use it

Use it when the user has (or is forming) a concrete trip - dates, a city, some bookings - and wants something to *follow* while moving. Use it for one-day layovers, weekend city breaks, multi-day single-city trips and multi-city tours.

Do not use it for open-ended "what should I see in X" research, flight or hotel booking, or expense splitting. If the user only wants recommendations, give recommendations; offer the page only if a real trip exists.

## The six principles (and why each exists)

1. **Only bookings own the clock.** Reservations, tickets, flights and hard deadlines get a fixed time. Everything else flows, with a soft `~` hint only where there is a real reason (queue, heat, closing time). Why: over-scheduled itineraries create anxiety the moment one thing slips, and something always slips.
2. **Answer three questions before anything else:** Where am I in the day? What is next? What if it goes wrong? The hero card answers the first two at a glance; every stop answers the third.
3. **Every stop carries its own failure plan.** A fallback, what it costs, and when to trigger it ("if the bus is not here by 18:10, take a taxi, about 35 EUR"). Why: anxiety lives in the unknown, and a pre-decided plan B removes it.
4. **Judgment beats coverage.** Say what to skip and why. Use numbers instead of adjectives ("saves 2.9 km", "about a fifth cheaper than the other shop"). Offer two or three nearby options, not twelve.
5. **Be honest about confidence.** Mark anything not verified for that exact weekday and date. A page that admits "hours unverified, call ahead" is trusted; a confident wrong opening time ruins the trip. End with a dated "checked on" footer.
6. **Carry-ability.** One HTML file, system fonts, no login, no tracking, readable one-handed in sunlight, `noindex`. It makes no network requests beyond itself, so a dropped signal mid-trip changes nothing; a tiny service worker (see *Deploying*) lets it reload offline too.

## Pick a shape

The approach has settled into four shapes. Choose by situation, then adapt.

| Shape | Best for | What it contributes |
|---|---|---|
| **Editorial Scroll** | Multi-day, multi-city trips read the night before | Serif/cream editorial voice, one color per day, sticky day rail, a reference panel (flights, hotels, tickets, reservations), walking routes with named landmarks, vibe notes, sensory previews, local phrase cribs, transit-point pickpocket alerts, emergency footer. Linear; no sense of "now". |
| **Flexible Day** | One-day city itinerary with a few bookings | Pastel paper UI, one `DATA` object, `fixed` vs `flow` split, "also nearby" extras per anchor, weather turned into a plan, dated to-do list, tap-to-mark-done, hero counting down to the next booked item. |
| **24-Hour Carry** | Layovers and short stopovers | Live `past`/`now` row states, Now/Next hero, to-scale route map, show-the-driver address cards with copy button, tax-refund math, phrase list, decision notes ("route changed: shopping leg drops from 15 km to 8 km"), sources footer. |
| **Quiet Multi-Day** | A few days in one city, shopping/food heavy | Restrained system-gray styling, quick-nav chips to utility sections (shop, cafe, dine, bags, tax, notes), to-scale city map, same engine. |

A suggested brand mark: the **Day Arc** - a sunrise-to-sunset arc with one dot showing where you are in the day. It is the same idea as the hero card (countdown, now, next). Reuse the idea for icons: soft rounded-square background, one arc, one dot.

Default to **Flexible Day** for one day and **Quiet Multi-Day** for several days. Use **Editorial Scroll** only when the user wants a beautiful pre-trip read and a separate carry version exists. Do not merge the pre-trip read and the in-the-moment page into one file; they serve different moments.

## Workflow

### 1. Intake - batch, then assume
Collect in one message (use a multi-question prompt if the tool exists): destination and dates; who is traveling and their taste (fashion, food, design, nature, kids, accessibility); every booking with its time; hard constraints (bag size, mobility, dietary); the language the traveler reads; how they move around (walking, metro, taxi, car). Read any emails, PDFs or notes the user supplied before asking anything that is already in them. For gaps, state an assumption in one line and proceed; do not interrogate.

### 2. Anchor math
List every fixed item with its source (ticket, confirmation, deadline). Work **backwards from the hardest anchor** to find true slack: landing time + immigration worst case + transfer + check-in vs. the first reservation. Use the pessimistic number for immigration and baggage, not the optimistic one. State the slack on the page ("you land at 15:20; dinner is at 20:00; immigration plus transfer eats about two hours").

### 3. Research and verify
For each stop confirm, for the specific weekday: opening hours, closure days, public holidays, reservation rules, dress code, price, typical queue. Prefer primary sources (venue site, transit operator, official tourist board) and recent reports. When two sources disagree, show the disagreement. Anything unconfirmed gets a `warn` chip such as "hours unverified - call ahead", never a confident guess. If search is unavailable, say so at the top of the page.

### 4. Author the DATA object
All content lives in one `DATA` object; the renderer is generic. Fields are in the next section. Write prose in the language the traveler reads; keep proper nouns, menu terms and local-currency prices in their original form so they stay searchable and usable at the door.

### 5. Render with the reference engine
Start from the engine at the end of this file, apply a theme, add the panels the trip needs. Keep it a single `index.html`.

### 6. Verify (see the checklist) and ship
Run the checklist, fix what it finds, then deploy to any static host. Hand back the URL, the top assumptions, the unverified list and the dated deadlines.

## DATA reference

```js
const DATA = {
  city: "Sample City",
  tz: "Europe/Lisbon",              // IANA zone of the destination; the clock always shows local time
  tzLabel: "local time",
  idle: "Sample City - one day",    // hero text when the trip is over
  quick: [{ id: "todo", label: "Before you go" }],   // chips linking to sections
  days: [{
    id: "d1", date: "2026-11-14",   // ISO date; weekday is computed, never typed
    title: "Sat 14 Nov", note: "One sentence that frames the day and names the anchor.",
    timeline: [{
      time: "13:00",                // wall-clock start; end is derived from dur, never hand-typed
      dur: 90,                      // minutes
      soft: false,                  // true = shown as "~13:00" (flexible)
      anchor: true,                 // booked or deadline; feeds the "next fixed" line
      emoji: "🍽", color: "peach",  // tile: blush | mint | butter | lilac | sky | peach
      title: "Lunch", alt: "",      // optional local-language label shown beside the title
      sub: "<b>Two sentences max.</b> What it is and the one fact that matters.",
      chips: [["booked", "Booked"], ["warn", "last order 14:30"]],
      map: "Venue name City",       // search query for the maps link
      local: "Address in local script",   // shows a copy button for taxi drivers
      tel: "+000000000000",         // only if the user wants it on the page
      more: "Expandable depth: why it is good, what to order, the fallback and its cost."
    }]
  }],
  panels: [{ id: "todo", title: "Before you go", note: "", rows: [["Fri 13 Nov", "Online check-in opens. <b>Do it before 12:00.</b>"]] }],
  emergency: [["Police", "112", "112"]],   // [label, shown, tel:]
  footer: "Checked on 2026-11-01: opening hours, transit fares, holiday calendar."
};
```

Optional sections (add as `panels` or custom blocks) are listed under *Utility modules*.

## Writing rules

- **`sub` is two sentences at most.** Bold the one fact the eye should catch. Put everything else in `more`.
- **Lead with the decision, then the reason.** "Take the metro, not a taxi: 20 minutes, covered, 1.60 EUR." Not a paragraph of options.
- **Name what a stranger would miss.** The entrance that is not on the street, the floor the reception is on, the turnstile that needs a ticket to exit.
- **Sensory and landmark cues** make unfamiliar streets feel known: "the stone glows amber under the streetlights", "past the old clock tower on your left".
- **Say what to skip.** One line on the overrated queue or tourist-priced option saves more anxiety than three recommendations.
- **Soft times only with a reason.** `~16:30` plus why ("shade after 17:00").
- **Taste is data.** Record the traveler's taste profile once and let it steer the "nearby" derivatives. A shared motif (a favorite emoji, a recurring treat) can be a small signature if the user wants one.
- **Tone is calm and specific.** No hype words, no emoji walls. One emoji per tile is the limit.

## Chip vocabulary

Keep a small, consistent set so the eye learns it.

| Chip | Meaning | Color |
|---|---|---|
| `booked` | Reserved or ticketed | green |
| `star` | The point of this stop | lilac |
| `cool` | Reassurance: easy, covered, air-conditioned, close | blue |
| `warn` | Deadline, hours risk, unverified fact | amber |
| `heat` | The place first-timers get burned (scam, wrong queue, wrong exit) | rose |
| `opt` | Optional, skip freely | dashed outline |
| (none) | A plain fact: price, duration, distance | neutral |

## Utility modules

Add only the ones the trip needs.

- **Arrival card** - airport to bed: exit, taxi rank or train, cost, the scam to avoid, which floor the hotel desk is on, what to do if the room is not ready (bag storage, a free lounge).
- **Border and baggage buffer** - an honest worst-case time for immigration, and how carry-on-only changes it.
- **Transit** - line, exit number, walking minutes, fare, last train, and the point at which a taxi is the better choice.
- **Show-the-driver cards** - address in local script, a phonetic line, a copy button. One card per place the traveler will need to be driven to.
- **Phrase crib** - situation, local phrase, pronunciation. Eight to ten lines, situational ("ask for the bill", "where is the toilet").
- **Tax-refund math** - the threshold, the real net percentage after fees, the order of the airport procedure, and the "do not open the sealed bag" rule.
- **Money** - what is cash-only, card minimums, how much cash to carry, bank-holiday closures.
- **Weather turned into a plan** - not a forecast; a decision ("a cold snap, not rain: layer up and book the indoor stop for the coldest hour").
- **Dress codes and closed days** - including holidays that close banks but not malls.
- **Nearby derivatives** - two to six on-taste extras per anchor, each within a stated walking time, each with weekday-verified hours.
- **Car, parking and tolls** - one park-once plan with the day rate, any toll or low-emission-zone charge, and the one fact to confirm by phone.
- **Before-you-go list with dated deadlines** - cancellation cut-offs, online check-in opening, dietary-notice windows, arrival-form windows, visa or entry-system reminders. Date every row.
- **Scale-true route map** - inline SVG; stops placed at real relative distances, a scale bar, a dashed circle around the walkable cluster, a one-line caption with the key number.
- **Decision notes** - when the plan changed, say why in numbers.
- **Emergency footer** - local emergency numbers, tourist hotline, consular line, and a placeholder for the hotel's phone.
- **Sources footer** - what was checked and when.

## Themes

Pick one and keep the rest of the engine unchanged.

| Theme | Tokens | Feel |
|---|---|---|
| **Paper Pastel** (default) | bg `#FCF7F1`, ink `#4B443E`, soft `#948578`, six pastel tile colors with deeper text tones, radius 22px, serif display + system UI | Warm, friendly, iOS-like |
| **Editorial Cream** | bg `#FAF7EE`, surface `#F3EDD9`, ink `#1F1C17`, accent `#8A6F2B`, per-country accents (`#B54A2C`, `#2D4F78`), serif display | Magazine-like pre-trip read |
| **Quiet Gray** | bg `#F5F5F7`, card `#FFFFFF`, ink `#1D1D1F`, secondary `#6E6E73`, link blue `#0066CC`, map land `#F2EFE9`, park `#DCEBD5`, water `#B9D8F0` | Restrained, Apple-like |

Design rules that held across every shape: system fonts only; minimum 44 px tap targets; 15-16 px body text; sticky header with a live local clock; past rows fade, the current row is highlighted; no horizontal page scroll at 375 px; respect the safe-area inset; add a web manifest and an apple-touch icon so "Add to Home Screen" gives a full-screen page.

## Verification checklist

Treat the page as unfinished until each item has a named check, not a feeling.

1. **Weekday and date** - compute the weekday of every day tab in a script and compare with the label.
2. **Hours** - each venue is open at the planned time on that weekday; holidays checked. Unknown becomes a `warn` chip.
3. **Time math** - add up legs and buffers from the hardest anchor; confirm the slack stated on the page.
4. **Time travel** - open the page with `?at=YYYY-MM-DDTHH:MM` for before the trip, mid-timeline and after the trip; confirm the hero text and the `past`/`now` row states each time.
5. **Links** - every maps query opens the right place; every copy button copies the right text.
6. **Fixed vs flow** - no fixed clock on a flexible item.
7. **Mobile** - 375 px wide, no horizontal scroll, taps are comfortable, text is readable.
8. **Offline** - the network panel shows no requests except the page itself (no external fonts, scripts or images); with the service worker installed, a reload with the network off still renders.
9. **Privacy scan** - see below.
10. **Console** - no errors.

## Privacy and safety

An itinerary reveals when someone is away from home, where they sleep and what they have booked.

- Do not put booking references, ticket or collection numbers, passport numbers, card digits or personal phone numbers on a page that is public. By default write "booked - reference in your email". Include codes only if the user asks for them and the page is private (unlisted URL behind a password or a private host).
- `noindex` is not security. If it must be private, say so and use access control.
- Mark the page `noindex, nofollow`, add no analytics, and keep progress marks in `localStorage` on the device only.
- Keep each trip in its own repo or deployment and share the brand assets separately, so one trip can be deleted without touching the others.
- Before publishing run a quick scan for long alphanumeric codes, phone numbers and email addresses.

## Anti-patterns

- A fixed clock on every row, so one delay breaks the whole page.
- Paragraph-long `sub` text; the page becomes an article nobody reads on a street corner.
- Recommendations without a fallback, a price or a reason.
- Confident hours that were never checked for that weekday.
- Twelve options where two would do.
- A hero that shows the trip's start date after the trip has begun.
- Hand-typed start and end minutes that drift from the displayed time; derive them.
- Decorative emoji walls, gradients on every card, or a motion that delays reading.
- Mixing the pre-trip read and the in-the-moment page in one file.

## Hand-back format

When the page is built, report in this order and keep it short: the file or URL; the assumptions taken; what is verified and how; what is unverified and what would verify it; the dated deadlines the traveler must act on; and the single riskiest moment of the trip with its fallback.

## Reference engine

A complete working page: Paper Pastel theme, live Now/Next hero, time-travel testing, tap-to-mark-done, copy buttons. Replace the sample data, keep the engine. Try it by appending `?at=2026-11-14T13:20`.

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover">
<meta name="robots" content="noindex,nofollow">
<meta name="theme-color" content="#FCF7F1">
<link rel="icon" href="data:,">
<title>Pocket 神灯 - Sample City</title>
<style>
:root{--paper:#FCF7F1;--card:#fff;--ink:#4B443E;--soft:#948578;--faint:#C6BAAD;--line:#F0E7DA;
--blush:#FBE4E6;--blush-d:#C1717F;--mint:#E1F0E7;--mint-d:#5E9B7C;--butter:#FBEFD2;--butter-d:#AE873A;
--lilac:#ECE5F4;--lilac-d:#8477AB;--sky:#E1EBF4;--sky-d:#5F87A6;--peach:#FCE7DC;--peach-d:#C1815C;--r:22px;
--ui:-apple-system,BlinkMacSystemFont,"SF Pro Text","Helvetica Neue",sans-serif;--serif:ui-serif,"New York",Georgia,serif}
*{box-sizing:border-box;-webkit-tap-highlight-color:transparent}
body{margin:0;background:var(--paper);color:var(--ink);font:15px/1.5 var(--ui);padding-bottom:env(safe-area-inset-bottom)}
header{position:sticky;top:0;z-index:5;display:flex;justify-content:space-between;align-items:center;padding:12px 18px;background:rgba(252,247,241,.9);backdrop-filter:blur(14px);border-bottom:1px solid var(--line)}
.wm{font:600 18px var(--serif)}.wm small{display:block;font:600 9px var(--ui);letter-spacing:2px;text-transform:uppercase;color:var(--soft)}
#clock{font:600 20px var(--ui);font-variant-numeric:tabular-nums;text-align:right}#clock small{display:block;font-size:10px;letter-spacing:1px;color:var(--soft)}
.wrap{max-width:560px;margin:0 auto;padding:16px 16px 80px}
.hero{background:linear-gradient(135deg,var(--blush),var(--butter));border-radius:var(--r);padding:20px}
.lab{font:600 11px var(--ui);letter-spacing:2px;text-transform:uppercase;color:var(--blush-d)}
.big{font:600 30px/1.12 var(--serif);margin:6px 0}.hsub{color:var(--ink)}.hfix{margin-top:8px;font-size:13px;color:var(--soft)}
.quick{display:flex;gap:8px;overflow-x:auto;padding:14px 0 2px}.quick a{flex:none;padding:10px 16px;border-radius:99px;background:var(--card);color:var(--ink);text-decoration:none;font-size:13px;border:1px solid var(--line)}
h2{font:600 21px var(--serif);margin:28px 0 2px}.note{color:var(--soft);font-size:14px;margin:2px 0 10px}
.card{background:var(--card);border-radius:var(--r);box-shadow:0 1px 2px rgba(75,68,62,.06),0 8px 24px rgba(75,68,62,.05);overflow:hidden}
.row{display:grid;grid-template-columns:48px 44px 1fr;gap:10px;padding:14px;border-bottom:1px solid var(--line);transition:opacity .2s}
.row:last-child{border:0}.row.past{opacity:.45}.row.now{background:#FFF9F1;box-shadow:inset 4px 0 0 var(--blush-d)}.row.done .title{text-decoration:line-through}
.t{font:600 13px var(--ui);font-variant-numeric:tabular-nums;color:var(--soft);padding-top:12px}
.tile{width:44px;height:44px;border-radius:14px;display:grid;place-items:center;font-size:21px;border:0;cursor:pointer}
.title{font-weight:650}.title i{font-style:normal;color:var(--soft);font-weight:500;font-size:13px}.sub{margin-top:4px;font-size:14px}
.chips{display:flex;flex-wrap:wrap;gap:6px;margin-top:8px}.chip{font-size:11.5px;padding:3px 9px;border-radius:99px;background:var(--line);color:var(--soft)}
.chip.booked{background:var(--mint);color:var(--mint-d)}.chip.star{background:var(--lilac);color:var(--lilac-d)}.chip.cool{background:var(--sky);color:var(--sky-d)}
.chip.warn{background:var(--butter);color:var(--butter-d)}.chip.heat{background:var(--blush);color:var(--blush-d)}.chip.opt{background:none;border:1px dashed var(--faint)}
.acts{display:flex;gap:8px;margin-top:8px}.acts a,.acts button{min-height:36px;padding:0 12px;border-radius:12px;border:1px solid var(--line);background:#fff;color:var(--ink);font:13px var(--ui);text-decoration:none;display:inline-grid;place-items:center;cursor:pointer}
details{margin-top:8px}summary{cursor:pointer;color:var(--soft);font-size:13px;min-height:28px}.mb{font-size:14px;padding-top:4px}
.prow{padding:14px;border-bottom:1px solid var(--line)}.prow:last-child{border:0}.prow b.l{display:block;font-size:12px;color:var(--soft);margin-bottom:2px}
.k{color:var(--blush-d)}
</style>
</head>
<body>
<header><div class="wm">Pocket 神灯<small id="city"></small></div><div id="clock">--:--</div></header>
<div class="wrap">
  <div class="hero"><div class="lab" id="hl"></div><div class="big" id="hb"></div><div class="hsub" id="hs"></div><div class="hfix" id="hf"></div></div>
  <nav class="quick" id="quick"></nav>
  <main id="app"></main>
</div>
<script>
/* SAMPLE DATA - fictional place. Replace everything below, keep the engine. */
const DATA = {
  city: "Sample City", tz: "Europe/Lisbon", tzLabel: "local time", idle: "Sample City - one day",
  quick: [{ id: "d1", label: "Timeline" }, { id: "todo", label: "Before you go" }, { id: "sos", label: "Emergency" }],
  days: [{
    id: "d1", date: "2026-11-14", title: "Sat 14 Nov",
    note: "One fixed lunch and one fixed dinner. Everything between them flows.",
    timeline: [
      { time: "10:30", dur: 60, soft: true, emoji: "☕", color: "butter", title: "Slow coffee near the hotel",
        sub: "<b>Walk-in, no booking.</b> Order at the counter; the back room is quieter.",
        chips: [["cool", "5 min walk"], ["opt", "skip if tired"]], map: "Sample Cafe, Sample City" },
      { time: "13:00", dur: 90, anchor: true, emoji: "🍽", color: "peach", title: "Lunch at Harbour Kitchen", alt: "Almoço",
        sub: "<b>Reserved for two.</b> Booking required; walk-ins are turned away at lunch.",
        chips: [["booked", "Booked"], ["warn", "last order 14:30"]], map: "Harbour Kitchen, Sample City", local: "Rua Exemplo 12, 1000-000 Sample City",
        more: "<b>Fallback:</b> if you are running late, phone the venue before 12:30 and ask to be held for 15 minutes." },
      { time: "15:00", dur: 60, soft: true, emoji: "🖼", color: "lilac", title: "Gallery on the same street",
        sub: "<b>Free, open Tuesday to Saturday until 19:00.</b> Best use of the gap after lunch.",
        chips: [["star", "the reason to come"], ["warn", "closes 19:00"]], map: "Sample Gallery, Sample City" },
      { time: "17:30", dur: 45, emoji: "🚇", color: "mint", title: "Metro back to the hotel",
        sub: "<b>Two stops, covered, 1.60 EUR.</b> Rush hour starts at 17:00.",
        chips: [["cool", "indoors"], ["heat", "use the north exit - the south one is a 10 min detour"]], map: "Sample Metro Station, Sample City",
        more: "<b>Fallback:</b> a taxi from the rank outside is about 12 EUR if it rains." },
      { time: "20:00", dur: 120, anchor: true, emoji: "🥂", color: "blush", title: "Dinner at Corner Table",
        sub: "<b>Reserved for two at 20:00.</b> Smart casual; no sandals.",
        chips: [["booked", "Booked"], ["warn", "dress code"]], map: "Corner Table, Sample City", local: "Travessa Modelo 3, 1000-001 Sample City" }
    ]
  }],
  panels: [{ id: "todo", title: "Before you go", note: "Two rows have real deadlines.",
    rows: [["Fri 13 Nov", "<b>Before 12:00:</b> free-cancellation cut-off for lunch."], ["Same day", "Mention any allergies when you confirm; late notice may not be accommodated."]] }],
  emergency: [["Emergency", "112", "112"], ["Tourist helpline", "Add local number", ""]],
  footer: "Sample data. Checked on 2026-11-01: add your sources here."
};

/* ---------------- engine ---------------- */
const $ = id => document.getElementById(id);
const esc = s => String(s ?? "").replace(/[&<>"]/g, c => ({ "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;" }[c]));
const mapsLink = q => "https://maps.apple.com/?q=" + encodeURIComponent(q);   // swap per country if a local app navigates better
const hm = m => String(Math.floor(m / 60)).padStart(2, "0") + ":" + String(m % 60).padStart(2, "0");
const toMin = t => { const [h, m] = t.split(":").map(Number); return h * 60 + m; };
const dayMin = d => { const [y, m, dd] = d.split("-").map(Number); return Date.UTC(y, m - 1, dd) / 60000; };
const dur = m => { m = Math.max(0, Math.round(m)); const d = Math.floor(m / 1440), h = Math.floor(m % 1440 / 60);
  const r = m % 60;
  return d >= 2 ? d + " days" : d ? d + "d " + h + "h" : h ? h + "h" + (r ? " " + r + "m" : "") : m + "m"; };

/* wall-clock minutes at the destination; ?at=YYYY-MM-DDTHH:MM overrides for testing */
function nowMin() {
  const at = new URLSearchParams(location.search).get("at");
  if (at) { const [d, t] = at.split("T"); return dayMin(d) + toMin(t); }
  const p = Object.fromEntries(new Intl.DateTimeFormat("en-GB", { timeZone: DATA.tz, year: "numeric", month: "2-digit", day: "2-digit",
    hour: "2-digit", minute: "2-digit", hourCycle: "h23" }).formatToParts(new Date()).map(x => [x.type, x.value]));
  return Date.UTC(+p.year, +p.month - 1, +p.day, +p.hour, +p.minute) / 60000;
}

const EVENTS = DATA.days.flatMap(d => d.timeline.map(e => { const s = dayMin(d.date) + toMin(e.time); return { ...e, abs: s, end: s + (e.dur || 30) }; }));
const KEY = "pg-done-v1";
const getDone = () => { try { return new Set(JSON.parse(localStorage.getItem(KEY) || "[]")); } catch { return new Set(); } };
const setDone = s => { try { localStorage.setItem(KEY, JSON.stringify([...s])); } catch {} };

function rowHtml(e, done) {
  const id = e.abs + "|" + e.title;
  const acts = (e.map ? `<a href="${mapsLink(e.map)}" target="_blank" rel="noopener">📍 Map</a>` : "")
    + (e.local ? `<button type="button" data-copy="${esc(e.local)}">📋 Copy address</button>` : "")
    + (e.tel ? `<a href="tel:${esc(e.tel)}">📞 Call</a>` : "");
  return `<div class="row${done.has(id) ? " done" : ""}" data-abs="${e.abs}" data-end="${e.end}">
    <div class="t">${e.soft ? "~" : ""}${esc(e.time)}</div>
    <button class="tile" style="background:var(--${e.color})" data-done="${esc(id)}" aria-label="Mark done">${e.emoji}</button>
    <div><div class="title">${esc(e.title)}${e.alt ? ` <i>${esc(e.alt)}</i>` : ""}</div><div class="sub">${e.sub}</div>
      ${e.chips ? `<div class="chips">${e.chips.map(([c, t]) => `<span class="chip ${c}">${esc(t)}</span>`).join("")}</div>` : ""}
      ${acts ? `<div class="acts">${acts}</div>` : ""}
      ${e.more ? `<details><summary>Details</summary><div class="mb">${e.more}</div></details>` : ""}</div></div>`;
}

function render() {
  const done = getDone();
  $("city").textContent = DATA.city;
  $("quick").innerHTML = DATA.quick.map(q => `<a href="#${q.id}">${esc(q.label)}</a>`).join("");
  const byDay = d => EVENTS.filter(e => e.abs >= dayMin(d.date) && e.abs < dayMin(d.date) + 1440);
  $("app").innerHTML = DATA.days.map(d => `<section id="${d.id}"><h2>${esc(d.title)}</h2><div class="note">${d.note || ""}</div>
      <div class="card">${byDay(d).map(e => rowHtml(e, done)).join("")}</div></section>`).join("")
    + DATA.panels.map(p => `<section id="${p.id}"><h2>${esc(p.title)}</h2><div class="note">${p.note || ""}</div>
      <div class="card">${p.rows.map(([l, h]) => `<div class="prow"><b class="l">${esc(l)}</b>${h}</div>`).join("")}</div></section>`).join("")
    + `<section id="sos"><h2>Emergency</h2><div class="card">${DATA.emergency.map(([l, s, t]) =>
        `<div class="prow"><b class="l">${esc(l)}</b>${t ? `<a href="tel:${esc(t)}">${esc(s)}</a>` : esc(s)}</div>`).join("")}</div></section>`
    + `<p class="note" style="margin-top:24px">${esc(DATA.footer)}</p>`;
}

function tick() {
  const n = nowMin();
  $("clock").innerHTML = hm(((n % 1440) + 1440) % 1440) + `<small>${esc(DATA.tzLabel)}</small>`;
  document.querySelectorAll(".row").forEach(r => { const a = +r.dataset.abs, e = +r.dataset.end;
    r.classList.toggle("past", n >= e); r.classList.toggle("now", n >= a && n < e); });
  const cur = EVENTS.find(e => n >= e.abs && n < e.end), nxt = EVENTS.find(e => e.abs > n), fix = EVENTS.find(e => e.abs > n && e.anchor);
  let lab, big, sub;
  if (!cur && !nxt) { lab = "Trip complete"; big = DATA.city; sub = esc(DATA.idle); }
  else if (cur) { lab = "Now"; big = esc(cur.title); sub = nxt ? `Then: <b>${esc(nxt.title)}</b> at ${nxt.time} (in ${dur(nxt.abs - n)})` : "That was the last stop. Wander freely."; }
  else if (nxt === EVENTS[0]) { lab = "Trip starts in"; big = dur(nxt.abs - n); sub = `First stop: <b>${esc(nxt.title)}</b>`; }
  else { lab = "Next up"; big = dur(nxt.abs - n); sub = `<b>${esc(nxt.title)}</b> · ${nxt.time}`; }
  $("hl").textContent = lab; $("hb").innerHTML = big; $("hs").innerHTML = sub;
  $("hf").textContent = fix && fix !== nxt ? `Next fixed: ${fix.title} at ${fix.time} (in ${dur(fix.abs - n)})` : "";
}

document.addEventListener("click", ev => {
  const c = ev.target.closest("[data-copy]"), d = ev.target.closest("[data-done]");
  if (c) { const txt = c.dataset.copy; (navigator.clipboard ? navigator.clipboard.writeText(txt) : Promise.reject()).catch(() => {
      const ta = document.createElement("textarea"); ta.value = txt; document.body.appendChild(ta); ta.select(); document.execCommand("copy"); ta.remove(); });
    c.textContent = "✓ Copied"; setTimeout(() => c.textContent = "📋 Copy address", 1400); }
  if (d) { const s = getDone(), id = d.dataset.done; s.has(id) ? s.delete(id) : s.add(id); setDone(s); d.closest(".row").classList.toggle("done", s.has(id)); }
});
document.addEventListener("visibilitychange", () => { if (!document.hidden) tick(); });
render(); tick(); setInterval(tick, 15000);
</script>
</body>
</html>
```

## Deploying

Any static host works (drag-and-drop hosting, GitHub Pages, a CDN bucket). Add a `site.webmanifest` and icons for full-screen "Add to Home Screen". Add `X-Robots-Tag: noindex, nofollow` as a response header where the host allows it. Re-run the verification checklist after every content edit, because late edits are where stale times and wrong weekdays creep in.

**Offline reloads.** A plain HTML file will not reload without a signal unless something caches it. Add this `sw.js` next to `index.html` and register it with one line at the end of the page script: `if ("serviceWorker" in navigator) navigator.serviceWorker.register("sw.js");`. It is network-first, so edits always show up when online, and it falls back to the cached copy when offline. Service workers need HTTPS (or localhost), which every common static host provides. Tested: with the network switched off, the page reloaded and rendered.

```js
const C = "pg-v1";
self.addEventListener("install", e => { self.skipWaiting(); e.waitUntil(caches.open(C).then(c => c.addAll(["./", "./index.html"]))); });
self.addEventListener("activate", e => e.waitUntil(self.clients.claim()));
self.addEventListener("fetch", e => e.respondWith(
  fetch(e.request).then(r => { const copy = r.clone(); caches.open(C).then(c => c.put(e.request, copy)); return r; })
    .catch(() => caches.match(e.request, { ignoreSearch: true }))));
```
