# TableReserve Bot — Bot specification

**Archetype:** booking

**Voice:** friendly and professional — write every user-facing message, button label, error, and empty state in this voice.

A restaurant reservation bot that manages table bookings with real-time availability, sends confirmation/reminders, and provides owner dashboards for managing reservations. Guests can reschedule/cancel via buttons, and owners see daily capacity summaries and can mark no-shows. All data is stored with configurable privacy retention.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Restaurant staff/owners
- Walk-in customers
- Phone callers using Telegram

## Success criteria

- Guests see accurate available slots based on restaurant config
- Reservations are confirmed with unique booking codes
- Owners receive real-time notifications of new bookings
- Guests can reschedule/cancel via buttons without text input
- Daily capacity summary shown in owner dashboard

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Show welcome message and booking explanation
- **Book Table** (button, actor: user, callback: booking:start) — Initiate guest booking flow
  - inputs: date, party size
  - outputs: reservation confirmation
- **/today** (command, actor: owner, command: /today) — Show today's reservation dashboard
  - outputs: compact reservation list, capacity summary
- **/upcoming** (command, actor: owner, command: /upcoming) — Show upcoming reservations overview
  - inputs: number of days
  - outputs: grouped reservations by day

## Flows

### Guest Booking
_Trigger:_ /start or 'Book Table' button

1. Welcome message
2. Date selection (calendar)
3. Party size selection
4. Show available slots
5. Collect optional guest info
6. Confirm reservation with code
7. Send owner notification

_Data touched:_ reservation, restaurant_config

### Reschedule
_Trigger:_ reservation:reschedule button

1. Confirm reschedule intent
2. Select new date
3. Select new time slot
4. Update reservation
5. Send confirmation to guest
6. Send update to owner

_Data touched:_ reservation

### Cancel
_Trigger:_ reservation:cancel button

1. Confirm cancellation
2. Mark reservation as canceled
3. Send confirmation to guest
4. Send update to owner

_Data touched:_ reservation

### Daily Summary
_Trigger:_ owner:today command

1. Fetch today's reservations
2. Calculate seat availability
3. Display compact dashboard

_Data touched:_ restaurant_config, reservation

### Reminder
_Trigger:_ scheduled event (reminder_lead_time before reservation)

1. Send reminder message
2. Include reschedule/cancel buttons

_Data touched:_ reservation

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **restaurant_config** _(retention: persistent)_ — Restaurant operating parameters
  - fields: opening_hours_weekdays, opening_hours_weekends, slot_duration, booking_window_days, reminder_lead_hours, time_granularity_minutes
- **table** _(retention: persistent)_ — Physical table inventory
  - fields: table_id, name, capacity, is_active
- **reservation** _(retention: persistent)_ — Confirmed table bookings
  - fields: booking_code, guest_name, guest_phone, party_size, start_time, end_time, assigned_tables, status, created_at, updated_at
- **owner_user** _(retention: persistent)_ — Authorized owner accounts
  - fields: telegram_id, notification_preferences

## Integrations

- **Telegram** (required) — Bot API messaging and notifications
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- View/edit restaurant configuration
- See today's reservations and capacity
- View upcoming reservations
- Mark reservations as completed/no-show
- Configure notification settings

## Notifications

- New booking confirmation to guest
- Reservation reminder to guest
- Daily summary to owner
- Upcoming bookings list to owner
- No-show marking by owner

## Permissions & privacy

- Guest contact info only visible to owners
- Guest data retention policy configurable (default 1 year)
- Booking codes act as reservation identifiers
- No-show status visible to all staff

## Edge cases

- No available slots for requested date/party size
- Guest provides invalid phone format
- Owner attempts to access without authorization
- Multiple simultaneous reschedule requests for same table

## Required tests

- End-to-end booking flow with slot availability calculation
- Owner dashboard shows correct remaining seats
- Reminder messages trigger at configured time
- Reschedule flow preserves party size constraints
- No-show marking updates reservation status

## Assumptions

- Default opening hours are 12:00-23:00
- Slot duration defaults to 90 minutes
- No waitlist functionality by default
- No automatic penalties for no-shows
