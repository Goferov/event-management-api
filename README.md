# Event Management API

## Introduction

This project is an API built using Laravel 10, designed to manage event creation, registration of attendees, and authentication. The API supports user login and logout using Laravel Sanctum for authentication, CRUD operations for events and attendees, and sending reminder notifications for upcoming events. It also leverages Laravel's Queue system for background processing and the Scheduler for automating tasks like sending reminders.

## Table of Contents
- [Features](#features)
- [Technologies](#technologies)
- [Database Design and Structure](#database-design-and-structure)
- [Queues and Scheduled Tasks](#queues-and-scheduled-tasks)
- [Installation](#installation)
- [License](#license)

## Features

- **User Authentication**: Supports user login and logout using Laravel Sanctum.
- **Event Management**: Create, update, delete, and view events with information on attendees.
- **Attendee Management**: Register users as attendees to specific events, view and remove attendees.
- **Event Reminders**: Automatic reminders sent to event attendees before an event starts.

### Endpoints

1. **User Authentication**
    - **Login**: `POST /api/login`
        - Description: Authenticate user and return an API token.
        - Parameters:
            - `email` (required)
            - `password` (required)
        - Response: JSON with an API token.
        - **Authentication Required**: No
    - **Logout**: `POST /api/logout`
        - Description: Logs out the authenticated user.
        - Middleware: `auth:sanctum`
        - Response: JSON with logout confirmation.
        - **Authentication Required**: Yes

2. **Events**
    - **List Events**: `GET /api/events`
        - Description: Retrieve all events.
        - Response: Paginated list of events with details.
        - **Authentication Required**: No
    - **Create Event**: `POST /api/events`
        - Description: Create a new event.
        - Middleware: `auth:sanctum`
        - Parameters:
            - `name` (required)
            - `desc` (optional)
            - `start_time` (required, date)
            - `end_time` (required, date)
        - Response: JSON with the created event.
        - **Authentication Required**: Yes
    - **Show Event**: `GET /api/events/{id}`
        - Description: Retrieve a specific event by ID.
        - Response: Event details including attendees.
        - **Authentication Required**: No
    - **Update Event**: `PUT /api/events/{id}`
        - Description: Update an event's details.
        - Middleware: `auth:sanctum`
        - Parameters:
            - `name`, `desc`, `start_time`, `end_time` (all optional)
        - Response: JSON with the updated event.
        - **Authentication Required**: Yes (only the owner of the event can update)
    - **Delete Event**: `DELETE /api/events/{id}`
        - Description: Delete an event by ID.
        - Middleware: `auth:sanctum`
        - Response: HTTP 204 No Content.
        - **Authentication Required**: Yes (only the owner of the event can delete)

3. **Attendees**
    - **List Attendees for an Event**: `GET /api/events/{id}/attendees`
        - Description: Retrieve all attendees for a given event.
        - Response: Paginated list of attendees.
        - **Authentication Required**: No
    - **Register Attendee for an Event**: `POST /api/events/{id}/attendees`
        - Description: Register the authenticated user as an attendee for a specific event.
        - Middleware: `auth:sanctum`
        - Response: JSON with the attendee's details.
        - **Authentication Required**: Yes
    - **Show Attendee Details**: `GET /api/events/{eventId}/attendees/{attendeeId}`
        - Description: Retrieve specific attendee details for an event.
        - Response: JSON with the attendee's details.
        - **Authentication Required**: No
    - **Remove Attendee**: `DELETE /api/events/{eventId}/attendees/{attendeeId}`
        - Description: Remove an attendee from an event.
        - Middleware: `auth:sanctum`
        - Response: HTTP 204 No Content.
        - **Authentication Required**: Yes (only the attendee or the event owner can remove the attendee)

4. **Send Event Reminders**
    - **Send Event Reminders**: Automatically scheduled via a command `app:send-event-reminders`, which sends reminders to attendees of upcoming events.

## Technologies

- **Laravel 10**: PHP framework used for building the API.
- **MariaDB**: Database for managing data.
- **Laravel Sanctum**: Used for API authentication and token management.
- **Notifications**: Laravel’s notification system used for sending event reminders.

## Database Design and Structure

- **Users**: Stores user credentials and details.
- **Events**: Stores event information such as name, description, start and end times.
- **Attendees**: Links users to events as attendees.

### Main Models:
1. **User**
    - Attributes: `id`, `name`, `email`, `password`, `created_at`, `updated_at`

2. **Event**
    - Attributes: `id`, `name`, `description`, `start_time`, `end_time`, `user_id`, `created_at`, `updated_at`
    - Relations:
        - `user`: The creator of the event.
        - `attendees`: Users attending the event.

3. **Attendee**
    - Attributes: `id`, `user_id`, `event_id`, `created_at`, `updated_at`
    - Relations:
        - `user`: The user attending the event.
        - `event`: The event the attendee is registered for.

## Queues and Scheduled Tasks

This application leverages Laravel's **Queue** and **Scheduler** to handle background processing and periodic tasks.

### Queue Usage
- **Event Reminders**: Notifications to event attendees are dispatched to the queue for asynchronous processing, ensuring that the main application flow is not delayed by sending emails.
- **Queue Driver**: The queue system can be configured with drivers like `database`, `redis`, etc. Ensure your `.env` is properly set up to enable queue functionality.

### Scheduled Tasks
- **Daily Event Reminders**: The API uses Laravel's task scheduling feature to automate sending reminders.
    - The command `app:send-event-reminders` is scheduled to run daily:
      ```php
      protected function schedule(Schedule $schedule): void
      {
          $schedule->command('app:send-event-reminders')->daily();
      }
      ```
    - This command retrieves all events starting in the next 24 hours and sends reminder notifications to registered attendees.

To enable scheduling, you need to set up a cron job on your server to call the Laravel scheduler every minute:
```bash
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```
## Installation

1. Clone the repository:
```
git clone https://github.com/your-repository-url.git
```
2. Navigate to the project directory:
```bash
cd your-project-directory
```
3. Install dependencies:
```bash
composer install
```
4. Set up environment variables by copying `.env.example` to `.env`:
```bash
cp .env.example .env
```
5. Configure your `.env` file for MariaDB and other settings.

6. Run database migrations:
```bash
php artisan migrate
```
7. Clone the repository:
```bash
git clone https://github.com/your-repository-url.git
```
8. (Optional) Seed the database
```bash
php artisan db:seed
```
9. Start the queue worker:
```bash
php artisan queue:work
```
10. Set up a cron job to run the Laravel scheduler every minute (on your server):
```bash
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```
11. Serve the application:
```bash
php artisan serve
```

## Installation
This project is licensed under the [MIT License](LICENSE.md) - see the file for details.

