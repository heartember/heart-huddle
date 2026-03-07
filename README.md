# Heart Huddle

A women's Authentic Relating circle in Santa Cruz. This repo hosts the public-facing website at [hearthuddle.live](https://hearthuddle.live), powered by GitHub Pages.

---

## Pages

### `index.html` — Main hub
The primary landing page at `hearthuddle.live`. Contains three sections:

- **Get the invites** — Buttondown subscribe form for the weekly email list
- **This week's gathering** — RSVP button, event details, and live attendee list (shown only when an event is active)
- **Next gathering coming soon** — shown when no event is active

### `rsvp.html` — Who's coming
A focused page at `hearthuddle.live/rsvp` showing only the attendee list for the current week. Linked from the confirmation email so attendees can see who else is joining.

### `join.html` — Quick RSVP link
A simple page at `hearthuddle.live/join` with a direct RSVP button. Useful for sharing in texts or group chats when someone asks how to sign up.

### `style.css` — Shared styles
Brand colors, typography, animations, and shared components used across all pages. Index-specific styles live in `index.html` directly.

---

## Brand

- **Colors:** Cream `#f5f0e8`, Flame `#e05218`, Ink `#2d2826`
- **Fonts:** Playfair Display (headings), DM Sans (body)
- **Icons:** `hearthuddle-icon.png` (flame), `logo__1_.png` (full logo)

---

## Google Forms

### RSVP Form
Attendees use this to reserve their spot each week. Limited to 9 participants (plus Ember = 10 total).

- Collects: Name, Email
- On submission: triggers the Apps Script `checkGuestLimit()` function
- Form closes automatically when capacity is reached
- URL: linked from the Buttondown invite email and from `hearthuddle.live/join`

### Cancellation Form
Attendees use this to cancel their reservation if they can't make it.

- Linked from the confirmation email as a button
- On submission: triggers `checkGuestLimit()`, which reopens the RSVP form if a cancellation brings the count below the limit

---

## Apps Script

The Google Apps Script lives in the Google Sheet connected to the RSVP form. It handles all backend logic.

### Functions

#### `checkGuestLimit(e)` — triggered on form submission
Runs automatically whenever someone submits the RSVP form or cancellation form.

1. Notifies Ember of the new submission with current reservation count
2. Sends a confirmation email to the participant with a link to the attendee list and a cancellation button
3. Closes the RSVP form when the limit is reached, with a custom closed message
4. Reopens the RSVP form if a cancellation brings the count back below the limit

#### `resetReservations()` — runs automatically via time-driven trigger (Mondays)
Clears all reservation rows from the sheet, reopens the RSVP form, and notifies Ember. Does **not** deactivate the hub page — that's separate.

#### `activateEvent()` — run manually on Wednesdays
Sets `IS_ACTIVE = true` in Script Properties, causing the hub page to show the RSVP section and attendee list.

**Wednesday rhythm:** Run `activateEvent()`, then send the Buttondown invite.

#### `deactivateEvent()` — run manually after the event
Sets `IS_ACTIVE = false`, hiding the RSVP section on the hub page and showing "Next gathering coming soon" instead.

#### `doGet()` — Web App endpoint
Serves a JSON response to the website pages. Returns:

```json
{
  "isActive": true,
  "names": ["Ember", "Jane", "Sasha"],
  "total": 3,
  "limit": 9
}
```

Deployed as a Web App: **Execute as: Me**, **Who has access: Anyone**. After any code changes, redeploy via Manage Deployments → edit → New Version. The URL stays the same.

---

## Buttondown

Weekly invites are sent via [Buttondown](https://buttondown.com/hearthuddle). Subscribers sign up via the form on the hub page or at `buttondown.com/hearthuddle`.

The invite email includes:
- Event details (date, time, location)
- RSVP link
- Link to the attendee list (`hearthuddle.live/rsvp`)

A typical invite looks like:

> Hi friends,
> Thursday, March 6th is our next Heart Huddle — an evening of connection games, presence, and good company.
> ...
> Space is limited to 10. RSVP here.
> Curious who's RSVP'd so far? See the guest list anytime.

---

## Weekly Workflow

| When | What |
|------|------|
| Wednesday | Run `activateEvent()` in Apps Script, send Buttondown invite |
| Thursday | Heart Huddle happens |
| After event | Run `deactivateEvent()` in Apps Script |
| Monday (auto) | `resetReservations()` clears the sheet and reopens the form |

---

## Deployment

Hosted on GitHub Pages from the `main` branch. Changes pushed to `main` go live within ~1 minute at `hearthuddle.live`.
