# Subscription Tracker API

Backend API for managing user subscriptions, authentication, renewal reminders, and workflow-triggered email notifications.

I created this project to practise my backend skills while working with authentication, databases, background workflows, and email integrations.

## Overview

This project is an Express + MongoDB API that lets users:

- create an account and sign in with JWT authentication
- create and manage subscription records
- fetch subscriptions for the authenticated user
- schedule renewal reminder workflows with Upstash
- send reminder emails before the renewal date
- protect the API with Arcjet bot detection and rate limiting

## Tech Stack

- Node.js
- Express
- MongoDB + Mongoose
- JWT authentication
- bcryptjs
- Nodemailer
- Upstash Workflow / QStash
- Arcjet

## Features

### Implemented

- user sign-up
- user sign-in
- user listing
- get a single user by id with authorization
- create a subscription for the authenticated user
- get subscriptions for the authenticated user
- automated reminder workflow trigger after subscription creation
- reminder emails at 7, 5, 2, and 1 day before renewal
- global error handling
- Arcjet protection for bot filtering and rate limiting

### Scaffolded / Placeholder Routes

The following routes currently return placeholder responses and are not fully implemented yet:

- `POST /api/v1/auth/sign-out`
- `POST /api/v1/users`
- `PUT /api/v1/users/:id`
- `DELETE /api/v1/users/:id`
- `GET /api/v1/subscriptions`
- `GET /api/v1/subscriptions/:id`
- `PUT /api/v1/subscriptions/:id`
- `DELETE /api/v1/subscriptions/:id`
- `PUT /api/v1/subscriptions/:id/cancel`
- `GET /api/v1/subscriptions/upcoming-renewals`

## Installation

1. Install dependencies:

	 `npm install`

2. Create an environment file:

	 - for development: `.env.development.local`
	 - for production: `.env.production.local`

3. Add the required environment variables.

4. Start the server:

	 - development: `npm run dev`
	 - production: `npm start`

## Environment Variables

The app loads environment variables from `.env.<NODE_ENV>.local`.

Example for `.env.development.local`:

```env
PORT=5500
NODE_ENV=development
SERVER_URL=http://localhost:5500

DB_URI=mongodb://127.0.0.1:27017/subscription-tracker

JWT_SECRET=your_jwt_secret
JWT_EXPIRES_IN=7d

ARCJET_ENV=development
ARCJET_KEY=your_arcjet_key

QSTASH_TOKEN=your_qstash_token
QSTASH_URL=your_qstash_url

EMAIL_PASSWORD=your_gmail_app_password
```

## Available Scripts

- `npm run dev` — starts the API with Nodemon
- `npm start` — starts the API with Node.js

## API Base URL

Local base URL:

`http://localhost:<PORT>/api/v1`

Health route:

- `GET /` → `Welcome to the Subscription Tracker API!`

## Authentication

Protected routes expect a bearer token in the `Authorization` header:

`Authorization: Bearer <jwt_token>`

## API Endpoints

### Auth

- `POST /api/v1/auth/sign-up`
- `POST /api/v1/auth/sign-in`
- `POST /api/v1/auth/sign-out` _(placeholder)_

#### Sign Up Request Body

```json
{
	"name": "John Doe",
	"email": "john@example.com",
	"password": "secret123"
}
```

#### Sign In Request Body

```json
{
	"email": "john@example.com",
	"password": "secret123"
}
```

### Users

- `GET /api/v1/users` — get all users
- `GET /api/v1/users/:id` — get a user by id _(protected)_
- `POST /api/v1/users` _(placeholder)_
- `PUT /api/v1/users/:id` _(placeholder)_
- `DELETE /api/v1/users/:id` _(placeholder)_

### Subscriptions

- `POST /api/v1/subscriptions` — create a subscription _(protected)_
- `GET /api/v1/subscriptions/user/:id` — get subscriptions for the authenticated user _(protected)_
- `GET /api/v1/subscriptions` _(placeholder)_
- `GET /api/v1/subscriptions/:id` _(placeholder)_
- `PUT /api/v1/subscriptions/:id` _(placeholder)_
- `DELETE /api/v1/subscriptions/:id` _(placeholder)_
- `PUT /api/v1/subscriptions/:id/cancel` _(placeholder)_
- `GET /api/v1/subscriptions/upcoming-renewals` _(placeholder)_

#### Create Subscription Request Body

```json
{
	"name": "Netflix",
	"price": 15.99,
	"currency": "USD",
	"frequency": "monthly",
	"category": "entertainment",
	"paymentMethod": "Visa ending in 1234",
	"startDate": "2026-03-01T00:00:00.000Z"
}
```

Notes:

- `renewalDate` is auto-generated when omitted.
- allowed `currency`: `USD`, `EUR`, `GBP`
- allowed `frequency`: `daily`, `weekly`, `monthly`, `yearly`
- allowed `status`: `active`, `cancelled`, `expired`

### Workflows

- `POST /api/v1/workflows/subscription/reminder`

This route is used by Upstash Workflow to process reminder jobs.

## How Reminder Scheduling Works

When a subscription is created:

1. the subscription is saved to MongoDB
2. a workflow run is triggered through Upstash
3. the workflow waits until reminder dates
4. reminder emails are sent 7, 5, 2, and 1 day before renewal

## Data Models

### User

- `name`
- `email`
- `password`

### Subscription

- `name`
- `price`
- `currency`
- `frequency`
- `category`
- `paymentMethod`
- `status`
- `startDate`
- `renewalDate`
- `user`

## Security Notes

- passwords are hashed with `bcryptjs`
- JWT is used for protected routes
- Arcjet provides shield, bot detection, and token bucket rate limiting
- duplicate and validation database errors are normalized by the error middleware

## Email Notes

- reminder emails are sent with Gmail through Nodemailer
- the sender address is currently configured in [config/nodemailer.js](config/nodemailer.js)
- for Gmail, use an App Password instead of your account password

## Development Notes

- MongoDB connection is initialized when the server starts
- the app uses ES modules (`"type": "module"`)
- environment loading depends on `NODE_ENV`

## Example Response

Successful sign-in response:

```json
{
	"success": true,
	"message": "User signed in successfully",
	"data": {
		"token": "<jwt_token>",
		"user": {
			"_id": "...",
			"name": "John Doe",
			"email": "john@example.com"
		}
	}
}
```