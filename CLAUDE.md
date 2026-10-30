# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A beginner's first web pages: static files (`index.html` and `todo.html`), each with inline CSS and JavaScript. There is no build step, package manager, linter, or test suite.

The user is a beginner and communicates in Hebrew. Reply in Hebrew and explain changes in simple terms.

## Running

Open the page in the default browser (PowerShell, Windows):

```
Start-Process "index.html"
```

After editing, the user needs to refresh the browser (`F5`) to see changes.

## Structure of index.html

- The page is Hebrew and right-to-left (`lang="he" dir="rtl"`). Keep new UI text in Hebrew and RTL-friendly.
- The Rubik font is loaded from Google Fonts. Everything else is inline, with no other external dependencies.
- Layout: a centered `.card` holding the `h1` title, the `#greet-button` button and the `#greeting` paragraph, on top of an animated gradient background.
- The button has a gentle `float` animation (it animates the `translate` property so it doesn't clash with the hover `transform`).
- Clicking the button picks a random greeting from the `greetings` array (never the same one twice in a row, tracked by `lastGreeting`), replays the `pop-in` CSS animation (by removing the `show` class, forcing a reflow, then re-adding it), and calls `launchConfetti()`.
- A `.nav-link` under the greeting links to `todo.html`.
- The confetti is hand-written on a full-screen `<canvas id="confetti">` with `pointer-events: none`. `launchConfetti()` pushes particles that start at the button. A single `requestAnimationFrame` loop (`drawConfetti`), guarded by the `animating` flag, applies gravity and drag and drops particles once they leave the screen. Repeated clicks add particles to the running loop instead of starting a second one.

## Structure of todo.html

- A to-do list in the same visual style as `index.html` (gradient background, white card, Rubik). It links back with a `.nav-link`.
- Each task is `{ id, text, start, end, done }`. `start` and `end` are `datetime-local` strings ("2026-10-30T14:00", local time). Both are required, and `end` must be after `start` (`validate()`).
- Tasks are saved to `localStorage` under `todo-tasks`. Reads and writes are wrapped in try/catch.
- `render()` rebuilds the whole `#task-list` from `tasks`, `filter` ("all" / "active" / "done") and `editingId` (the task shown as an inline edit form, opened by double-clicking a task). Completed tasks are sorted to the bottom; within each group, tasks keep the order they were added in. Items are built with DOM methods and `textContent`, never `innerHTML` with user text.
- Checking a task calls `launchConfetti(checkbox)`, the same canvas confetti as `index.html`, starting from the checkbox.
- Reminders: `getReminders(task)` builds the full schedule from the day the task was added (`id` is its creation timestamp) to its deadline. The user's workday is 08:00–16:00 (`WORK_START` / `WORK_END`), every day of the week. The schedule:
  - a reminder at the task's start time;
  - on days with 2 or more days left: 08:00 and 15:30;
  - on the day before the deadline: 08:00, 12:00 and 15:30, plus 20:00 ("tomorrow is the last day");
  - on the deadline day: 08:00, then 5h, 3h, 2h, 1h, 30m and 15m before the deadline (`lastDayOffsets`), then at the deadline itself.
  The user asked for reminders that get denser as the deadline nears, to help with starting and finishing tasks. Keep that intent when changing the schedule.
- `checkReminders()` runs every 30 seconds. Each task stores `lastNotified`, a timestamp. Reminders between it and now fire, and if several were missed, only the latest one is shown. When the page opens (`checkReminders(true)`), missed reminders are skipped unless the latest one is the deadline itself. The user chose this so that reopening the page doesn't bring up reminders for every task.
- The add form fills start = now and end = one hour later. With start = now there's no start reminder unless the user picks a later start. The old default (next round hour) made the start reminders of every task added in that hour fire together. Adding, editing or un-checking a task resets `lastNotified` to now.
- A reminder opens `#alarm`, a full-screen window that stays until the user clicks "הבנתי". While it is open, a loud three-note Web Audio sound repeats every 2.5 seconds and the tab title flashes. More reminders that arrive while it is open are added to its list. The user asked for reminders they must hear, see and close themselves, so don't make them quieter or auto-dismiss them.
- Sound only works after the page has been clicked once. `#sound-bar` asks for that click and hides once audio is running.
- A system `Notification` (with `requireInteraction`) is also shown when permission is granted (`#notify-bar` asks for it). Notifications have no `tag`: with a shared tag, a new one silently replaced one still open.
- Urgency colors: `.soon` (less than a day left) and `.overdue`. `updateUrgency()` updates them in place so an open edit form isn't rebuilt.
- Google Calendar allows at most 5 reminders per event, so the calendar stage will need several small reminder events per task, all deleted when the task is completed.
- Planned next stages: publishing to GitHub Pages, Google Calendar sync (events created and deleted with tasks), and syncing the list through the user's Google account.
