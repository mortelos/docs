---
title: "MortelOS Booking"
nav_title: "Booking"
slug: "package-booking"
version: "0"
description: "Personal appointment links with open times, booking questions, an embeddable booking form and bookings in the owner's calendar."
section: "packages"
order: 63
status: "mvp"
audience: "developers"
package: "mortelos/booking"
canonical_path: "/docs/0/package-booking"
last_verified: "2026-10-16"
public: true
---

# MortelOS Booking

`mortelos/booking` gives every signed-in user their own appointment links. Each link has its own open times and questions, and a public booking form that other sites can embed. A booking goes into the link owner's calendar through a calendar contract that the host binds. This page describes version 0.1.1.

The owner screens, the public form and the messages are in Dutch.

## Use It For

| Area | Package responsibility |
| --- | --- |
| Links | Per-user booking links with a name, duration, start-time interval, notice, horizon, buffer and an on/off switch. |
| Open times | Weekly or one-off blocks per link, each with a location: online, an address or by phone. Single days can be closed and weekly blocks can be ended. |
| Questions | Extra questions on the booking form: short text, long text, single choice, multiple choice, phone number or yes/no. |
| Public form | A public booking page, a confirmation page and a signed cancel link, per tenant and embeddable in an iframe. |
| Availability | Free slots from the open times, minus the owner's busy calendar time and existing bookings. |
| Chat tool | The `booking` agent tool, so a user can manage their links from the chat on the Afspraken pages. |
| Navigation | An Afspraken item in the sidebar. |

## Install

```bash
composer require mortelos/booking
```

Packages resolve from the MortelOS registry. Configure registry access once before installing; see [Installation](/docs/0/installation#configuring-package-access).

The package needs PHP 8.4, Livewire 4.4 or later, `mortelos/framework` 0.6.42 or later, `mortelos/ui` 0.4.8 or later and `mortelos/chat` 0.4. The service provider is auto-discovered.

The package adds its migrations to the tenant migration paths (`tenancy.migration_parameters`). The tables are created per tenant when you run the tenant migrations: `booking_links`, `booking_blocks`, `booking_block_skips`, `booking_questions` and `bookings`.

## Connect A Calendar

The package does not talk to a calendar itself. Bind `Mortelos\Booking\Contracts\BookingCalendar` to your own implementation:

| Method | What it does |
| --- | --- |
| `isConnected(string $ownerUserId): bool` | Whether the owner has a working calendar. |
| `busy(string $ownerUserId, CarbonImmutable $from, CarbonImmutable $to): array` | The owner's busy periods as a list of `BusyTime`. |
| `create(string $ownerUserId, BookingEvent $event): string` | Creates the calendar event and returns its ID. |
| `cancel(string $ownerUserId, string $eventId): void` | Cancels the calendar event. |

Throw `Mortelos\Booking\Calendar\CalendarUnavailable` when the calendar cannot be reached. The visitor then gets a message to try again, and nothing is saved.

```php
use Mortelos\Booking\Contracts\BookingCalendar;

// In a service provider of the host app. GoogleBookingCalendar is your own class.
$this->app->bind(BookingCalendar::class, GoogleBookingCalendar::class);
```

Without a binding, the package uses `NullBookingCalendar`. It reports no calendar connected, so the public form shows a not-connected page with status 503.

A `BookingEvent` has its start and end in UTC, plus the visitor-facing time zone. Its summary is the link name and the visitor name. Its description holds the visitor's details, the answers and the cancel link. The visitor is set as the attendee. If the booking cannot be saved after the event was created, the package cancels the event again.

For tests, bind `Mortelos\Booking\Testing\FakeBookingCalendar`. Use `connect()` to connect an owner, `setBusy()` to set busy periods and `failNext()` to make the next call fail. Check the public `created` and `cancelled` lists afterwards.

## Routes

| Area | Prefix | Middleware | Route names |
| --- | --- | --- | --- |
| Owner pages | `afspraken` (`os_prefix`) | `web`, `auth` (`os_middleware`) | `booking.index` and the pages under it. |
| Public pages | `afspraak` (`public_prefix`) | `AllowEmbedding`, `InitializeTenancyFromBookingUrl`, `throttle:booking-public` (`public_middleware`) | `booking.public.show`, `booking.public.confirmed`, `booking.public.cancel` |

Public URLs carry the tenant slug and the link slug. `InitializeTenancyFromBookingUrl` looks up the tenant by `slug` on `tenancy.tenant_model` and answers 404 when there is none. The current tenant must have a slug for the package to build booking URLs. Confirmation and cancel URLs are signed.

`AllowEmbedding` removes `X-Frame-Options` and sets `Content-Security-Policy: frame-ancestors` to the `frame_ancestors` config value. It also sets `Cache-Control: no-store`. The default `*` lets any site embed the form; set your own origins to narrow it.

The `booking-public` rate limiter allows 10 POST requests per minute and 20 per day per IP address, and 60 other requests per minute per IP address. The package defines it only when the host has not defined a limiter with that name, so define your own to change the limits.

## Configuration

The package merges `config/booking.php` under the `booking` key and registers no publish tag. To change a value, add a `config/booking.php` to the host. The merge is one level deep: a top-level key you set, such as `defaults`, replaces the whole package value for that key.

| Key | Default | Meaning |
| --- | --- | --- |
| `timezone` | `Europe/Amsterdam` | Time zone for open times, slots and the dates shown to visitors. |
| `os_prefix` | `afspraken` | URL prefix of the owner pages. |
| `os_middleware` | `['web', 'auth']` | Middleware of the owner pages. |
| `public_prefix` | `afspraak` | URL prefix of the public pages. |
| `public_middleware` | see Routes | Middleware of the public pages. |
| `frame_ancestors` | `*` | Value of the `frame-ancestors` directive on public pages. |
| `public_css_entry` | `resources/css/app.css` | CSS entry for the public pages. |
| `navigation.label`, `navigation.sort` | `Afspraken`, `16` | Sidebar label and position. |
| `defaults.slot_interval_minutes`, `defaults.notice_days`, `defaults.horizon_days`, `defaults.buffer_minutes` | `30`, `1`, `28`, `15` | Settings for a new link. |

## Link Settings And Limits

| Setting | Allowed values |
| --- | --- |
| Duration | 15, 30, 45, 60, 90 or 120 minutes. |
| Start times | Every 15, 30 or 60 minutes. |
| Notice | 0, 1 or 2 days. With 0, the first slot is the next start time from now. |
| Horizon | 7 to 84 days, in whole weeks. |
| Buffer | 0, 5, 10, 15 or 30 minutes before and after each appointment. |
| Link address | Lower-case letters, digits and hyphens, unique in the tenant. A new link gets the owner's first name and the link name, with a number added when that is taken. |
| Open times | Start and end on a whole quarter hour, with the end after the start. An address is required for an address location, up to 255 characters. |
| Choice questions | At least two different options. |

A weekly block repeats from its first date, optionally until an end date. Closing one day of a weekly block skips that date only. Ending a weekday stops weekly blocks on that weekday after the last date. Existing bookings always stay.

## Visitor Rules

- A name is required: up to 100 characters, without links. A valid e-mail address is required.
- A phone number has 6 to 20 characters: digits, spaces, `+`, `(`, `)` and `-`. It is required when the slot is by phone.
- Short text answers allow up to 255 characters and long text answers up to 2,000, both without links.
- One e-mail address can have at most two upcoming bookings with the same owner.
- A hidden `website` field catches bots. A filled-in field rejects the form.
- The chosen slot must still be free. A lock per tenant and owner handles one booking at a time. A slot that was just taken returns status 409 and keeps the visitor's input.
- An inactive link or unknown link address shows a closed page with status 404.
- Visitors can cancel through the signed link until the appointment starts. Cancelling also cancels the calendar event.

## Chat Tool

When the framework's agent tool registry is present, the package registers the `booking` tool. Its policy action is `agent_tool:booking` and it is deny-by-default, so grant it to the roles that may use it. The tool is offered only when the chat run's source context has `surface` set to `booking`, which means the chat on the Afspraken pages. It has write risk and asks for no approval.

| Action | What it does |
| --- | --- |
| `links.list` | Lists the user's links with their open times. |
| `link.create` | Creates a link from `name` and `duration_minutes`. |
| `blocks.list` | Shows the open times of one link. |
| `blocks.add` | Adds weekly or one-off blocks. |
| `dates.close` | Closes single dates. |
| `blocks.end` | Stops a weekday after `last_date`. |
| `question.add` | Adds a question to the booking form. |

The tool works only on the run user's own links. It never cancels or moves bookings. It lists bookings that stay, with times only: visitor names stay out of the tool output.

## Navigation

When the framework's navigation registry is bound, the package adds a sidebar item to the `Vast` group. The item has the configured label, the `calendar` icon, the route `booking.index` and the permission `nav.sidebar.booking`.

## Outlets

An outlet is a place where you extend a package from your app without changing it; read [Outlets](/docs/0/outlets) for how each kind works. List the outlets the installed version declares with:

```bash
php artisan mortelos:outlets:list booking
```

In version 0.1.1, the `BookingCalendar` contract, the `booking` config and the `booking-public` rate limiter are the places a host plugs in.

## Boundaries

The package owns links, open times, questions, availability, the public form, bookings and the chat tool. The host app owns the calendar implementation, user accounts, tenancy with tenant slugs, and the policies for `agent_tool:booking` and `nav.sidebar.booking`. The host also decides where the form is embedded. The package sends no e-mail itself. The visitor is added as an attendee to the calendar event, so any invitation depends on the calendar implementation.
