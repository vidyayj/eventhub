# EventHub

A backend API for a simplified event ticketing platform, built with Django and Django REST Framework. Users can browse events, reserve seats and cancel reservations.

## How to run the project

1. Clone the repository and go into the project folder:

```
git clone https://github.com/yourname/eventhub.git
cd eventhub
```

2. Create and activate a virtual environment (Windows):

```
python -m venv venv
venv\Scripts\activate
```

3. Install the dependencies:

```
pip install -r requirements.txt
```

4. Create the database tables:

```
python manage.py migrate
```

5. Start the server:

```
python manage.py runserver
```

The API is now available at `http://127.0.0.1:8000/api/`.

## Endpoints

### Events

| Method | URL | What it does |
|---|---|---|
| GET | `/api/events/` | List all events, sorted by date |
| GET | `/api/events/?status=upcoming` | List events with the given status |
| GET | `/api/events/?venue=pune` | List events whose venue contains the text (case-insensitive) |
| POST | `/api/events/` | Create an event |
| GET | `/api/events/{id}/` | Get one event, including `reservations_count` (confirmed reservations only) |
| PUT / PATCH | `/api/events/{id}/` | Update an event |
| DELETE | `/api/events/{id}/` | Delete an event and its reservations |

### Reservations

| Method | URL | What it does |
|---|---|---|
| GET | `/api/reservations/` | List all reservations |
| GET | `/api/reservations/?event_id=1` | List reservations of one event |
| POST | `/api/reservations/` | Reserve seats. Reduces the event's `available_seats`. Returns 400 if the event is completed or cancelled, if fewer than 1 seat is asked for, or if there are not enough seats |
| GET | `/api/reservations/{id}/` | Get one reservation |
| POST | `/api/reservations/{id}/cancel/` | Cancel a reservation. Sets its status to `cancelled` and gives the seats back to the event. Returns 400 if it is already cancelled |

## Middleware

`RequestLoggingMiddleware` (in `events/middleware.py`) logs the method, path, status code and time taken of every request to the terminal.

## Design decision

**Cancelling a reservation keeps the record and changes its status, instead of deleting it.**

I wanted the booking history to stay in the database, so a cancelled reservation can still be seen and counted later. The `cancel` endpoint also refuses to cancel a reservation twice. Without that check, a second cancel would add the seats back again, and the event would show more free seats than it really has.

Seat deduction (on reserve) and seat restoration (on cancel) both change `available_seats` and the reservation in the same method, so the two numbers stay consistent in normal use.

**Known limitation:** the two writes are not wrapped in `transaction.atomic()`, so two users booking the last seat at exactly the same moment could still cause an overbooking. A production system would add a transaction and row locking.

## Screenshots

Postman screenshots are in the `screenshots` folder:

- `01_reservation_201.png`: successful reservation
- `02_overbooking_400.png`: overbooking failure
- `03_cancellation_200.png`: successful cancellation
