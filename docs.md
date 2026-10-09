# External API Documentation

The External API allows third-party integrations to access VA data using an API key.

---

## Base URL

```
/api/external
```

---

## Authentication

All requests must include an API key. It can be provided in one of two ways:

**Authorization Header (Bearer Token):**
```
Authorization: Bearer <API_KEY>
```

**Query Parameter:**
```
?apikey=<API_KEY>
```

If no valid API key is provided, the API returns `401 Unauthorized`.

---

## Standard Response Format

Most endpoints return the following JSON structure:

```json
{
  "errorCode": 200,
  "errorMessage": null,
  "customMessage": null,
  "data": { }
}
```

| Field           | Type             | Description                              |
|-----------------|------------------|------------------------------------------|
| `errorCode`     | `number`         | HTTP status code                         |
| `errorMessage`  | `string \| null` | Error message, if applicable             |
| `customMessage` | `string \| null` | Optional custom message                  |
| `data`          | `any \| null`    | The requested data, or `null` on failure |

The `values/*` endpoints use a simplified response:
```json
{
  "value": 42
}
```

---

## Endpoints

### Details

#### `GET /api/external/details/subTier`

Returns the VA's current subscription tier and whether it is active.

**Response `data`:**
```json
{
  "selectedTier": "string",
  "active": true
}
```

---

### Members

#### `GET /api/external/members/all`

Returns a list of all memberships in the VA.

**Response `data`:** Array of membership objects.

---

#### `GET /api/external/members/specific`

Returns a single member, including their rank, leave of absence records, and PIREPs with aircraft/livery info.

**Query Parameters:**

| Parameter | Type     | Required | Description      |
|-----------|----------|----------|------------------|
| `id`      | `number` | Yes      | The member's ID  |

**Response `data`:** Membership object with nested `Rank`, `leaveOfAbsence`, and `pirep` (including `Aircraft` and `livery`).

**Error responses:**
- `400` – `No ID found` (if `id` query param is missing)
- `400` – `No Member found` (if no member matches the given ID)

---

### Stats

#### `GET /api/external/stats/all`

Returns aggregated statistics for the VA over the last 60 days and all time.

**Response `data`:**
```json
{
  "newPIREPs": 12,
  "totalPIREPs": 345,
  "topCraft": { },
  "newPassengers": 1500,
  "totalPilots": 80,
  "newPilots": 5,
  "newFT": 2400
}
```

| Field          | Type             | Description                                           |
|----------------|------------------|-------------------------------------------------------|
| `newPIREPs`    | `number`         | Approved PIREPs filed in the last 60 days             |
| `totalPIREPs`  | `number`         | Total approved PIREPs of all time                     |
| `topCraft`     | `object \| null` | Most frequently used aircraft (with livery), or null  |
| `newPassengers`| `number`         | Passengers carried in the last 60 days (PASSENGER type PIREPs) |
| `totalPilots`  | `number`         | Total number of pilots in the VA                      |
| `newPilots`    | `number`         | Pilots who joined in the last 60 days                 |
| `newFT`        | `number`         | Total flight time (in minutes) logged in the last 60 days|

---

### PIREPs

#### `GET /api/external/pireps/all`

Returns all PIREPs for the VA, including aircraft (with livery), multipliers, route, and VA info.

**Response `data`:** Array of PIREP objects.

---

#### `GET /api/external/pireps/specific`

Returns a single PIREP, including aircraft (with livery), multipliers, and route.

**Query Parameters:**

| Parameter | Type     | Required | Description     |
|-----------|----------|----------|-----------------|
| `id`      | `number` | Yes      | The PIREP's ID  |

**Error responses:**
- `400` – `Data missing` (if a body field is missing; the specific field will be specified in the response)

---

#### `POST /api/external/pirep/`

Returns a single PIREP, including aircraft (with livery), multipliers, and route.

**BODY fields:**

| Parameter      | Type     | Required | Description                                                              |
|----------------|----------|----------|--------------------------------------------------------------------------|
| `membershipID` | `number` | Yes      | The Pilot's ID (found through API or on the VA Dashboard)                |
| `flightNum`    | `string` | Yes      | The flight number                                                        |
| `depICAO`      | `string` | Yes      | The departure airport ICAO                                               |
| `arrICAO`      | `string` | Yes      | The arrival airport ICAO                                                 |
| `aircraft`     | `number` | Yes      | The aircraft's ID (found through API)                                    |
| `estFThrs`     | `number` | Yes      | Flight time hours                                                        |
| `estFTmins`    | `number` | Yes      | Flight time minutes                                                      |
| `fuel`         | `number` | Yes      | The fuel burned)                                                         |
| `date`         | `Date`   | Yes      | The date the flight was flown (Format: **YEAR-MM-DD**T**HR:MN:SS.MS**Z ) |
| `type`         | `string` | Yes      | PASSENGER/CARGO                                                          |
| `load`         | `number` | Yes      | The load of PAX or CARGO in Kg                                           |
| `bags`         | `number` | No       | The number of bags carried for PAX                                       |
| `comments`     | `string` | No       | Any comments provided by pilot for themselves or for staff               |
| `operator`     | `string` | No       | The operator of the flight                                               |
| `multi`        | `number` | No       | The ID of the multiplier (found through API)                             |


**Error responses:**
- `400` – `No ID found` (if `id` query param is missing)
- `400` – `No PIREP found` (if no PIREP matches the given ID)

---

### Operations

#### `GET /api/external/operations/all`

Returns all operations for the VA, including multipliers.

**Response `data`:** Array of operation objects.

---

#### `GET /api/external/operations/specific`

Returns a single operation, including multipliers.

**Query Parameters:**

| Parameter | Type     | Required | Description          |
|-----------|----------|----------|----------------------|
| `id`      | `number` | Yes      | The operation's ID   |

**Error responses:**
- `400` – `No ID found` (if `id` query param is missing)
- `400` – `No Operation found` (if no operation matches the given ID)

---

### Aircraft

#### `GET /api/external/aircrafts/all`

Returns all active aircraft in the VA's fleet, including livery info. Results are ordered by `minimumHours` descending.

**Response `data`:** Array of aircraft objects (with `livery`).

---

### Events

#### `GET /api/external/events/all`

Returns all events belonging to the VA.

**Response `data`:** Array of event objects.

---

### Routes

#### `GET /api/external/routes/all`

Returns all routes for the VA, including associated aircraft (with livery).

**Response `data`:** Array of route objects.

---

#### `GET /api/external/routes/specific`

Returns a single route, including associated aircraft (with livery).

**Query Parameters:**

| Parameter | Type     | Required | Description    |
|-----------|----------|----------|----------------|
| `id`      | `number` | Yes      | The route's ID |

**Error responses:**
- `400` – `No ID found` (if `id` query param is missing)
- `400` – `No Route found` (if no route matches the given ID)

---

### Invites

#### `GET /api/external/invite/`

Returns all invite records for the VA.

**Response `data`:** Array of invite objects.

---

#### `POST /api/external/invite/`

Creates a new invite; and returns its object as well as the URL that must be used.

**BODY fields:**

| Parameter     | Type      | Required | Description                                                              |
|---------------|-----------|----------|--------------------------------------------------------------------------|
| `oneTimeUse`  | `boolean` | No       | Optional value for making the invite usable multiple, or just one time   |

**Response `data`:** Newly created invite object.

**Error responses:**
- `400` – `Data missing` (if required body fields are missing)
- `400` – `Invite already exists` (if an active invite already exists for the email)

---

### Ranks

#### `GET /api/external/ranks/pilots`

Returns all active pilot ranks for the VA.

**Response `data`:** Array of rank objects with `rankType: "PILOT"`.

---

#### `GET /api/external/ranks/atc`

Returns all active ATC ranks for the VA.

**Response `data`:** Array of rank objects with `rankType: "ATC"`.

---

### Multipliers

#### `GET /api/external/multiplier/`

Returns all the multipliers records for the VA.

**Query parameters:**

| Parameter     | Type      | Required | Description                                                                  |
|---------------|-----------|----------|------------------------------------------------------------------------------|
| `pirep`       | `boolean` | No       | Optional inclusion of all the PIREPs that have ever used this multiplier     |
| `operations`  | `boolean` | No       | Optional inclusion of all the Operations that have ever used this multiplier |


**Response `data`:** Array of invite objects.

---

### Values

The `values` endpoints return a simplified `{ "value": <number> }` response.

#### `GET /api/external/values/pilot`

Returns the total number of pilots (memberships) in the VA.

---

#### `GET /api/external/values/aircraft`

Returns the total number of aircraft in the VA.

---

#### `GET /api/external/values/routes`

Returns the total number of routes in the VA.

---

#### `GET /api/external/values/pireps`

Returns the total number of PIREPs in the VA.

---

#### `GET /api/external/values/hours`

Returns the total flight hours across all pilots in the VA (sum of all pilot `flightTime` values in minutes, divided by 60 to convert to hours).

---

## Error Reference

| Status | Message                       | Description                                   |
|--------|-------------------------------|-----------------------------------------------|
| `401`  | `Unauthorized`                | Missing or invalid API key                    |
| `404`  | `Virtual Airline not found`   | The API key's associated VA does not exist    |
| `400`  | `No ID found`                 | A required `id` query parameter is missing    |
| `400`  | `No Member found`             | No member found with the given ID             |
| `400`  | `No PIREP found`              | No PIREP found with the given ID              |
| `400`  | `No Operation found`          | No operation found with the given ID          |
| `400`  | `No Route found`              | No route found with the given ID              |
| `400`  | `No Invite found`             | No invite found with the given ID             |
| `400`  | `Invite already exists`       | Active invite already exists for this email   |
