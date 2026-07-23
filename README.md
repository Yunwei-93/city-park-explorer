# City Park Explorer

## 1. Overview
**City Park Explorer** is a web application that allows users to explore, review, and favorite parks in New York City.  
Users can read other users’ experiences, share their own reviews, and discover the best parks for recreation and relaxation.

## 2. Tech Stack
- Backend: Node.js, Express, MongoDB
- Frontend: Handlebars templates + client-side JavaScript, HTML, CSS
- API Testing: Postman

## 3. Features

### 3.1 Core Features

- **Home Page**
  - Shows a brief introduction to City Park Explorer and what the website does.
  - Provides navigation to the core site features (park directory, user profile, etc.).
  - Highlights **popular parks** on the homepage (e.g., parks with more reviews).（）

- **Park Directory**
  - Displays a list of all parks, including **name, short introduction, address, and average rating**.
  - Supports **search** by different criteria such as area, facility type, or tags.
  - Supports **rating-based filtering** so users can quickly find higher-rated parks.

- **Individual Park Page**
  - Shows detailed park information: basic info, facility details, and the full list of user reviews.
  - Allows users to **post reviews** for a park, including a numeric rating and detailed text content.

- **User Functionality**
  
  - **Registered users** can:
    - Write reviews for parks.
    - Mark parks as favorites (Mark parks as favorites. If a user marks the same park again, it will be removed from their favorites (toggling favorite on second click)).
    - View the list of parks they have reviewed.
    - Comment on other users’ reviews.
    - Delete the reviews or comments they posted
  
  - **Guests (unregistered users)** can:
    - Browse all public pages (home page, park directory, park details).
    - Read other users’ reviews, but cannot post or favorite parks.

- **Admin Page**
  - Administrators can **add, edit, or delete parks** from the park directory.
  - Administrators can **delete user comments or reviews** that violate rules or are inappropriate.

### 3.2 Optional Features

- **Browsing History**
  - Track and display parks that a user has recently viewed, making it easier to revisit them.

- **User Following**
  - Allow users to follow other users and see parks or reviews from people they care about.

- **Social Media Sharing**
  - Provide quick sharing buttons so users can share a park or a review to external social platforms.

- **Personalized Recommendations**
  - Recommend parks to users on their personal page based on:
    - Their ZIP code / location.
    - Themes or types of parks they have liked or reviewed before.



## 4. How to Run

- `npm install`
- `npm start`
-  The app will run at `http://localhost:3000`.


## 5. User Module Documentation
This section details the **User** implementation, including data models, authentication flow, and API endpoints. All user routes are prefixed with `/users`.

### 5.1 User Data Model

User data is stored in the `users` collection.

**Schema Example:**
```javascript
{
  _id: ObjectId("..."),
  first_name: "Alice",
  last_name: "Smith",
  email: "alice@example.com",
  passwordHash: "<bcrypt hash>",    // Never returned to frontend
  role: "user",                      // "user" or "admin"
  address_zip: "12345",              // Optional
  address_city: "Hoboken",           // Optional
  favorite_Parks: [                  // Array of Park ObjectIds
    ObjectId("..."),
    ObjectId("...")
  ],
  createdAt: ISODate("2025-01-01T00:00:00Z")
}
```

**Validation Rules:**
- **Name:** 2–50 characters; letters, spaces, apostrophes, and hyphens only.
- **Email:** Normalized to lowercase; must be valid and unique.
- **Password:** Min 8 chars; must contain 1 uppercase, 1 lowercase, 1 digit, 1 special char.
- **Role:** Defaults to "user".
- **Zip/City:** Optional. Zip must be valid US format (5 or 9 digits). City max 100 chars.

### 5.2 Server-Rendered Pages (Handlebars)

These routes serve the HTML shell. Dynamic data is loaded via client-side `fetch`.

| Method | Path                    | Description             | Access Control    |
|--------|-------------------------|-------------------------|-------------------|
| GET    | `/users/register`       | Registration form       | Guests only       |
| GET    | `/users/login`          | Login form              | Guests only       |
| GET    | `/users/userProfile`    | User profile page       | Logged-in Users   |
| GET    | `/users/adminProfile`   | Admin dashboard         | Admins only       |
| GET    | `/users/logout`         | Logout confirmation     | Logged-in Users   |


### 5.3 User JSON API Reference

These endpoints are used by the frontend to perform actions.

| Method | Path                            | Auth        | Description                                    |
|--------|---------------------------------|-------------|------------------------------------------------|
| POST   | `/users/register`               | No          | Register a new user                            |
| POST   | `/users/login`                  | No          | Log in, create session                         |
| POST   | `/users/logout`                 | Yes*        | Log out, destroy session                       |
| GET    | `/users/me`                     | Yes         | Get current logged-in user                     |
| GET    | `/users/:id`                    | Optional    | Get a user by ID                               |
| GET    | `/users/me/favorites`           | Yes         | Get current user's favorite parks              |
| POST   | `/users/me/favorites/:parkId`   | Yes         | Add park to favorites, Toggle park in favorites|
| DELETE | `/users/me/favorites/:parkId`   | Yes         | Remove park from favorites                     |
| POST   | `/users/admin/role`             | Admin only  | Promote a user to admin          

#### 1. Register User

**`POST /users/register`**

**Request Body:**
```json
{
  "firstName": "Alice",
  "lastName": "Smith",
  "email": "alice@example.com",
  "password": "StrongP@ssw0rd!",
  "confirmPassword": "StrongP@ssw0rd!",
  "addressZip": "12345",
  "addressCity": "Hoboken"
}
```

**Response (200):** Returns created user object( without password). Note: Does not log user in automatically.
```json
{
  "_id": "507f1f77bcf86cd799439011",
  "first_name": "Alice",
  "last_name": "Smith",
  "email": "alice@example.com",
  "role": "user",
  "address_zip": "12345",
  "address_city": "Hoboken",
  "favorite_Parks": [],
  "createdAt": "2025-01-01T00:00:00.000Z"
}
```

**Errors:** 
- `400` - Validation error
- `409` - Email already exists
- `500` - Server error

---


#### 2. Login

**`POST /users/login`**

**Request Body:**
```json
{
  "email": "alice@example.com",
  "password": "StrongP@ssw0rd!"
}
```

**Response (200):** Returns user object (without the password) and sets session cookie.
```json
{
  "_id": "507f1f77bcf86cd799439011",
  "first_name": "Alice",
  "last_name": "Smith",
  "email": "alice@example.com",
  "role": "user",
  "address_zip": "12345",
  "address_city": "Hoboken",
  "favorite_Parks": [],
  "createdAt": "2025-01-01T00:00:00.000Z"
}
```

**Errors:**
- `400` - Input error
- `401` - Invalid credentials
- `500` - Server error
---


#### 3. Logout

**`POST /users/logout`**

**Auth:** Required (any logged-in user).

**Response:**
```json
{
  "loggedOut": true
}
```
**Description:** 
Destroys session and clears AuthCookie.
**Errors:**

- `500` - Server error

---



#### 4. Get Current User

**`GET /users/me`**

**Auth:** Required.

**Response(200):**
```json
{
  "_id": "...",              
  "first_name": "Alice",   
  "last_name": "Smith",     
  "email": "alice@...",    
  "role": "user",          
  "address_zip": "12345",    
  "address_city": "Hoboken", 
  "favorite_Parks": [],   
  "createdAt": "..." 
}        
```

**Errors:**

- `401` - Not authenticated error
- `404` - User not found
- `400` - Validation failed
- `500` - Server error

---

#### 5. Get User by ID

**`GET /users/:id`**

**Description:** Fetch public profile info of any user.
**Auth:** Optional (anyone can view public user profiles).
**Path Parameters:** id - User's MongoDB ObjectId (string)

**Response（200）:**
```json
{
  "_id": "...",              
  "first_name": "Alice",   
  "last_name": "Smith",     
  "email": "alice@...",    
  "role": "user",          
  "address_zip": "12345",    
  "address_city": "Hoboken", 
  "favorite_Parks": "[...]",   
  "createdAt": "..." 
}        
```
**Errors:**
- `401` - Not authenticated error
- `404` - User not found
- `400` - Validation failed
- `500` - Server error
---


#### 6. Favorite Parks Management

  **6.1 Get Favorites**
  **`GET /users/me/favorites`**
  **Auth:** Required.
  **Response（200）:**
  ```json
  {
    "_id": "...",              
    "first_name": "Alice",   
    "last_name": "Smith",     
    "email": "alice@...",    
    "role": "user",          
    "address_zip": "12345",    
    "address_city": "Hoboken", 
    "favorite_Parks": "[...]",   
    "createdAt": "..." 
  }   
  ```     

  **Response(200):**
  ```json
  [
    {
      "_id": "PARK_ID_1",
      "park_name": "Central Park",
      "park_location": "Manhattan",
      "park_zip": "10024",
      "description": "Large public park in NYC",
      "park_type": "Urban Park",
      "rating": 4.8,
      "reviewCount": 1250
    },
    {
      "_id": "PARK_ID_2",
      "park_name": "Prospect Park",
      "park_location": "Brooklyn",
      "park_zip": "11215",
      "description": "Historic Brooklyn park",
      "park_type": "Urban Park",
      "rating": 4.7,
      "reviewCount": 890
    }
  ]     
  ```

  **Errors:**
  - `401` - Not authenticated error
  - `404` - User not found
  - `400` - Validation failed
  - `500` - Server error
    
  ---

    
  **6.2 Add to Favorites (Toggle Park in Favorites)**
  **`POST /users/me/favorites/:parkId`**
  **Auth:** Required.
  **Path Parameters:** parkId - Park's MongoDB ObjectId (string)
  **Response(200):**
  ```json
  {
  "_id": "507f1f77bcf86cd799439011",
  "favorite_Parks": ["PARK_ID_1", "PARK_ID_2", "PARK_ID_3"]
  }
  ```
  **Errors:**
  - `401` - Not authenticated error
  - `404` - User not found
  - `400` - parkId parameter is required or validation failed
  - `500` - Server error
  **Description:**
  Toggles park in user's favorites:
  -If park is not in favorites → adds it (The returned list having this parkID)
  -If park is already in favorites → removes it (The returned list without this parkID)


  **6.3 Remove from Favorites**
  **`DELETE /users/me/favorites/:parkId`**
  **Auth:** Required.
  **Path Parameters:** parkId - Park's MongoDB ObjectId (string)

  **Response(200):**
  ```json
  {
  "_id": "507f1f77bcf86cd799439011",
  "favorite_Parks": ["PARK_ID_1", "PARK_ID_2", "PARK_ID_3"]
  }
  ```

  **Errors:**
  - `401` - Not authenticated error
  - `404` - User not found
  - `400` - parkId parameter is required or validation failed
  - `500` - Server error
    
  ---


#### 7. Promote to Admin

**`POST /users/admin/role`**

**Auth:** Admin only.

**Request Body:**
```json
{
"userId": "USER_ID_STRING"
}
```

**Response(200):** ()
```json

{
"_id": "507f1f77bcf86cd799439011",
"first_name": "Alice",
"last_name": "Smith",
"email": "alice@example.com",
"role": "admin",
"address_zip": "12345",
"address_city": "Hoboken",
"favorite_Parks": ["PARK_ID_1", "PARK_ID_2"],
"createdAt": "2025-01-01T00:00:00.000Z"
}
```
return updated user object with `role: "admin"`.

  **Errors:**
  - `401` - Not authenticated error
  - `404` - User not found
  - `400` - Input validation failed
  - `409` - Duplicated roles
  - `500` - Server error
    
---


## 6. Park Module Documentation

This section details the **Park** implementation, including data models, validation rules, and API endpoints.  
All park routes are prefixed with:
```
/parks
```

### 6.1 Park Data Model

Park data is stored in the `parks` collection.

**Schema Example:**
```json
{
  "_id": "ObjectId(...)",
  "park_name": "Central Park",
  "park_location": "M",
  "park_zip": ["10024", "10025"],
  "description": "Large public park in NYC with trails, lakes, and playgrounds.",
  "park_type": "Urban Park",
  "rating": 4.6,
  "reviewCount": 1250
}
```

**Note:**
- `rating` and `reviewCount` are computed fields based on user reviews.
- They are not meant to be edited directly by the frontend or admin; they are recalculated from the review module.

#### Validation Rules (Helpers)

The park-related validation helpers live in `helpers.js` and are used by the data layer (`data/parks.js`):

**Park Name (`park_name`)**
- Required, non-empty string.
- Length: 1–80 characters.
- Must not be only whitespace.

**ZIP Codes (`park_zip`)**
- Required.
- Accepts either:
  - A single string `"10024"` → stored internally as `["10024"]`.
  - An array of strings `["10024", "10025"]`.
- Each ZIP must:
  - Be a string of exactly 5 digits.
  - No empty array allowed.

**Location (`park_location`)**
- Required, non-empty string.
- Stored as a borough code (uppercase):
  - `"M"` – Manhattan
  - `"B"` – Brooklyn
  - `"Q"` – Queens
  - `"X"` – Bronx
  - `"R"` – Staten Island
- Frontend can map these codes to human-readable names.

**Description (`description`)**
- Required, non-empty string.
- Max length: 999 characters.

**Park Type (`park_type`)**
- Required, non-empty string.
- Max length: 20 characters (e.g., "Urban Park", "Playground", "Dog Park").

**Rating (`rating`)**
- If omitted, defaults to 0.
- Must be a number between 0 and 5 (inclusive).
- Normally set/updated by review logic (not by admin directly).

**Sort (sort query param)**
- Allowed values:
  - `rating_asc`, `rating_desc`
  - `name_asc`, `name_desc`
  - `reviews_asc`, `reviews_desc`
- Default: `rating_desc`.

**Limit (limit query param for popular parks)**
- Must be a positive integer.
- Default: 10 if not provided.

**Min Rating (minRating query param)**
- Optional.
- Must be a number between 0 and 5 if provided.

### 6.2 Park JSON API Reference

These endpoints power the Home Page, Park Directory, Park Detail Page, and any "Popular / Recommended" sections on the frontend.

| Method   | Path                 | Auth       | Description                                      |
|----------|----------------------|------------|--------------------------------------------------|
| GET      | `/parks`             | No         | Get all parks (with filters & sorting)           |
| GET      | `/parks/:id`         | No         | Get single park by ID                            |
| GET      | `/parks/popular`     | No         | Get popular parks (top N by rating/reviews)      |
| GET      | `/parks/recommend`   | No         | Get recommended parks based on ZIP/location      |
| POST     | `/parks`             | Admin only | Create a new park                                |
| PUT      | `/parks/:id`         | Admin only | Update an existing park                          |
| DELETE   | `/parks/:id`         | Admin only | Delete a park                                    |


### 6.3 Endpoint Details & Frontend Integration

#### 1. Get All Parks

**`GET /parks`**

- **Auth:** Not required.
- **Used by:**
  - Park Directory page (initial load + filters).
  - Any page that needs a full list with search/filter UI.

**Query Parameters:**

All params are optional:

- `search` – fuzzy search on park name/description.
- `location` – borough code (M, B, Q, X, R).
- `type` – park type string (e.g., "Urban Park").
- `zipcode` – 5-digit ZIP code string.
- `minRating` – number between 0 and 5 (filter by minimum rating).
- `sort` – one of:
  - `rating_asc`, `rating_desc`
  - `name_asc`, `name_desc`
  - `reviews_asc`, `reviews_desc`



**Success (200):**

Returns an array of park objects:

```json
[
  {
    "_id": "PARK_ID_1",
    "park_name": "Central Park",
    "park_location": "M",
    "park_zip": ["10024", "10025"],
    "description": "Large public park in NYC",
    "park_type": "Urban Park",
    "rating": 4.8,
    "reviewCount": 1250
  },
  {
    "_id": "PARK_ID_2",
    "park_name": "Prospect Park",
    "park_location": "B",
    "park_zip": ["11215"],
    "description": "Historic Brooklyn park",
    "park_type": "Urban Park",
    "rating": 4.7,
    "reviewCount": 890
  }
]
```

**Errors:**
- `400` – validation failed (e.g., invalid ZIP, invalid sort value, bad minRating).
- `500` – Internal error.


#### 2. Get Park by ID

**`GET /parks/:id`**

- **Auth:** Not required.
- **Used by:**
  - Park Detail page – load one park's info, rating, and metadata.

**Path Parameters:**
- `id` – Park's MongoDB ObjectId string.

**Success (200):**
```json
{
  "_id": "PARK_ID_1",
  "park_name": "Central Park",
  "park_location": "M",
  "park_zip": ["10024", "10025"],
  "description": "Large public park in NYC",
  "park_type": "Urban Park",
  "rating": 4.8,
  "reviewCount": 1250
}
```

**Errors:**
- `400` – ID missing or not a valid ObjectId.
- `404` – No park found with that ID.
- `500` – Internal error.


#### 3. Get Popular Parks

**`GET /parks/popular`**

- **Auth:** Not required.
- **Used by:**
  - Home Page – "Popular Parks" section (e.g., top 3 or top 5 parks).
  - Any sidebar widget showing "Top rated parks".

**Query Parameters:**
- `limit` – optional, positive integer.
  - Example: `?limit=3` → top 3 parks.
  - Default: 10 if not provided.

**Success (200):**
```json
[
  {
    "_id": "PARK_ID_1",
    "park_name": "Central Park",
    "park_location": "M",
    "park_zip": ["10024"],
    "description": "Large public park in NYC",
    "park_type": "Urban Park",
    "rating": 4.8,
    "reviewCount": 1250
  },
  {
    "_id": "PARK_ID_2",
    "park_name": "Prospect Park",
    "park_location": "B",
    "park_zip": ["11215"],
    "description": "Historic Brooklyn park",
    "park_type": "Urban Park",
    "rating": 4.7,
    "reviewCount": 890
  }
]
```

(Sorted by popularity: high rating and/or high reviewCount.)

**Errors:**
- `400` – invalid limit (not a number, non-positive).
- `500` – Internal error.

#### 4. Get Recommended Parks

**`GET /parks/recommend`**

- **Auth:** Not required.
- **Used by:**
  - User Profile page – "Parks Near You" or "Recommended For You".
  - Any page where recommendations are needed based on user address.

**Query Parameters:**
- `zipcode` – optional 5-digit string.
- `location` – optional borough code (M, B, Q, X, R).

Either or both may be provided. The backend will:
- Validate inputs using the existing helpers.
- Return parks that are "close" or in matching borough/ZIP ranges.


**Success (200):**

Returns an array of park objects (same shape as other park lists).

**Errors:**
- `400` – validation failed (invalid zipcode or location).
- `500` – Internal error.

#### 5. Create Park (Admin)

**`POST /parks`**

- **Auth:** Admin only 
- **Used by:**
  - Admin Dashboard – "Add New Park" form.

**Request Body:**
```json
{
  "park_name": "New Riverside Park",
  "park_location": "M",
  "park_zip": ["10027"],
  "description": "Small riverside park with biking trails.",
  "park_type": "Urban Park"
}
```

`rating` and `reviewCount` are not sent by the client; they are managed by the backend.

**Success (200):**
```json
{
  "_id": "NEW_PARK_ID",
  "park_name": "New Riverside Park",
  "park_location": "M",
  "park_zip": ["10027"],
  "description": "Small riverside park with biking trails.",
  "park_type": "Urban Park",
  "rating": 0,
  "reviewCount": 0
}
```
**Errors:**
- `400` – Missing required fields or validation failed.
- `409` – Park already exists (name/ZIP combination already stored).
- `500` – Internal error.

#### 6. Update Park (Admin)

**`PUT /parks/:id`**

- **Auth:** Admin only.
- **Used by:**
  - Admin Dashboard – edit park info (name, zip, description, type, etc).

**Path Parameters:**
- `id` – Park ObjectId string.

**Request Body:**

Any subset of editable fields:
```json
{
  "park_name": "Central Park (North)",
  "park_location": "M",
  "park_zip": ["10026", "10027"],
  "description": "Updated description...",
  "park_type": "Urban Park"
}
```

**Important:**
- Fields like `rating` and `reviewCount` are read-only and cannot be updated directly.
- If the backend detects an attempt to change them, it throws an error like "Rating cannot be modified directly" and returns `400`.

**Success (200):**
```json
{
  "_id": "PARK_ID_1",
  "park_name": "Central Park (North)",
  "park_location": "M",
  "park_zip": ["10026", "10027"],
  "description": "Updated description...",
  "park_type": "Urban Park",
  "rating": 4.8,
  "reviewCount": 1250
}
```

**Errors:**
- `400` – invalid ID, validation failed, or attempt to modify rating/reviewCount directly.
- `404` – park not found.
- `409` – another park already exists with same name/ZIP (conflict).
- `500` – Internal error.


#### 7. Delete Park (Admin)

**`DELETE /parks/:id`**

- **Auth:** Admin only.
- **Used by:**
  - Admin Dashboard – remove a park that should no longer appear.

**Path Parameters:**
- `id` – Park ObjectId string.

**Success (200):**

Typical shape (example):
```json
{
  "deleted": true,
  "parkId": "PARK_ID_1"
}
```

**Errors:**
- `400` – invalid ID.
- `404` – park not found.
- `500` – Internal error.

---

## 7. Review Module Documentation

This section documents the Review system: how reviews are stored, how park ratings are updated,

**Prefix:** All routes below assume the review router is mounted at `/reviews`

### 7.1 Review Data Model

Review data is stored in the `reviews` collection.

**Schema Example:**
```javascript
{
  _id: ObjectId("..."),
  user_id: ObjectId("USER_ID"),   // reference to users._id
  park_id: ObjectId("PARK_ID"),   // reference to parks._id
  rating: 4,                      // numeric rating
  review_content: "Nice park!",
  createdAt: ISODate("2025-01-01T12:00:00Z"),
  updatedAt: ISODate("2025-01-01T12:00:00Z") || null    // or ISODate(...) after editing
}
```

### 7.2 Validation Rules

**High-level rules:**

**IDs**
- `reviewId`, `userId`, `parkId` must be valid Mongo ObjectId strings.

**Rating**
- Must be a valid numeric rating.
- Must be between 1 and 5 inclusive.
- Anything else (string, float, <1, >5) will throw a validation error.

**Review Content**
- Must be a non-empty string.
- Length must be between 10 and 1000 characters.
- Cannot contain the same character repeated 5 times or more in a row 


### 7.3 Park Rating & Review Count Sync

The review data layer maintains park rating and review count via:
```javascript
async function recalculateParkRating(parkId)
```

**Logic:**

1. Fetch all reviews for the given `parkId` (`getReviewsByParkId`).
2. Compute:
   - `rating` = average of `review.rating`, rounded to 2 decimals.
   - `reviewCount` = number of reviews for that park.
3. Update the corresponding park document in `parks` collection:
```javascript
   { rating: r, reviewCount: reviewCount }
```


### 7.4 Review JSON API – Overview

Assuming mounted under `/reviews`:

| Method   | Path                      | Auth       | Description                                    |
|----------|---------------------------|------------|------------------------------------------------|
| GET      | `/reviews/:reviewId`      | No         | Get a single review by ID                      |
| PUT      | `/reviews/:reviewId`      | User/Admin | Update a review                                |
| DELETE   | `/reviews/:reviewId`      | User/Admin | Delete a review                                |
| GET      | `/reviews/park/:parkId`   | No         | Get all reviews for a given park               |
| POST     | `/reviews/park/:parkId`   | User       | Create a new review for a park                 |
| GET      | `/reviews/user/:userId`   | No         | Get all reviews written by a specific user     |

**Permission rules:**

- **Read (GET):** public.
- **Create (POST /park/:parkId):** must be logged in.
- **Update/Delete (PUT/DELETE /:reviewId):**
  - Must be logged in, and
  - Either the author of the review, or `role === "admin"`.

### 7.5 Endpoint Details & Frontend Integration

#### 7.5.1 Get Review by ID

**`GET /reviews/:reviewId`**

- **Auth:** Not required.

**Response (200):**
```json
{
  "_id": "REVIEW_ID",
  "user_id": "USER_ID",
  "park_id": "PARK_ID",
  "rating": 4,
  "review_content": "Nice park.",
  "createdAt": "2025-01-01T12:00:00.000Z",
  "updatedAt": null
}
```

**Errors:**
- `400` – invalid review ID.
- `404` – review not found.
- `500` – internal server error.

#### 7.5.2 Get Reviews for a Park

**`GET /reviews/park/:parkId`**

- **Auth:** Not required.

**Path Params:**
- `parkId` – park's MongoDB ObjectId (string).

**Response (200):**
```json
[
  {
    "_id": "REVIEW_ID_1",
    "user_id": "USER_ID_1",
    "park_id": "PARK_ID",
    "rating": 5,
    "review_content": "Amazing view!",
    "createdAt": "2025-01-01T12:00:00.000Z",
    "updatedAt": null
  },
  {
    "_id": "REVIEW_ID_2",
    "user_id": "USER_ID_2",
    "park_id": "PARK_ID",
    "rating": 4,
    "review_content": "Good for jogging.",
    "createdAt": "2025-01-02T09:30:00.000Z",
    "updatedAt": "2025-01-03T10:00:00.000Z"
  }
]
```

If there are no reviews, returns an empty array `[]`.

**Errors:**
- `400` – invalid park ID.
- `404` – park not found.
- `500` – internal server error.

**Typical frontend use:**


#### 7.5.3 Create Review for a Park

**`POST /reviews/park/:parkId`**

- **Auth:** Requires logged-in user (`req.session.user`).

**Path Params:**
- `parkId` – park's MongoDB ObjectId (string).

**Request Body (JSON):**
```json
{
  "rating": 4,
  "review_content": "Nice and clean park."
}
```

**Behavior:**

1. Validates user session.
2. Validates `parkId`, `rating`, `review_content`.
3. Ensures:
   - The user exists.
   - The park exists.
   - The same user cannot review the same park twice.
4. Inserts review into `reviews` collection.
5. Recalculates the park's rating and reviewCount.

**Success (200):**

Returns the created review object, e.g.:
```json
{
  "_id": "NEW_REVIEW_ID",
  "user_id": "USER_ID",
  "park_id": "PARK_ID",
  "rating": 4,
  "review_content": "Nice and clean park.",
  "createdAt": "2025-01-05T14:00:00.000Z",
  "updatedAt": null
}
```

**Errors:**
- `401` – not logged in.
- `400` – invalid IDs or invalid rating/review content.
- `404` – park not found.
- `409` – user already reviewed this park (duplicate review).
- `500` – internal server error.



#### 7.5.4 Update Review

**`PUT /reviews/:reviewId`**

- **Auth:** Required (author or admin).

**Path Params:**
- `reviewId` – review's MongoDB ObjectId.

**Request Body:**
```json
{
  "rating": 5,
  "review_content": "Updated review text."
}
```


**Success (200):**

Returns updated review object:
```json
{
  "_id": "REVIEW_ID",
  "user_id": "USER_ID",
  "park_id": "PARK_ID",
  "rating": 5,
  "review_content": "Updated review text.",
  "createdAt": "2025-01-01T12:00:00.000Z",
  "updatedAt": "2025-01-06T10:00:00.000Z"
}
```

**Errors:**
- `401` – not logged in.
- `400` – invalid review ID or invalid fields.
- `403` – user is neither author nor admin.
- `404` – review not found.
- `500` – internal server error.


#### 7.5.5 Delete Review

**`DELETE /reviews/:reviewId`**

- **Auth:** Required (author or admin).

**Path Params:**
- `reviewId` – review's MongoDB ObjectId.

**Success (200):**

Intended response shape:
```json
{
  "review_id": "REVIEW_ID",
  "deleted_comments_count": 3,
  "deleted": true
}
```

**Errors:**
- `401` – not logged in.
- `400` – invalid review ID.
- `403` – user is neither author nor admin.
- `404` – review not found.
- `500` – internal server error.

#### 7.5.6 Get Reviews by User

**`GET /reviews/user/:userId`**

- **Auth:** Not required.

**Path Params:**
- `userId` – user's MongoDB ObjectId.

**Response (200):**

Returns an array of reviews written by this user:
```json
[
  {
    "_id": "REVIEW_ID_1",
    "user_id": "USER_ID",
    "park_id": "PARK_ID_1",
    "rating": 4,
    "review_content": "Nice park.",
    "createdAt": "2025-01-01T12:00:00.000Z",
    "updatedAt": null
  },
  {
    "_id": "REVIEW_ID_2",
    "user_id": "USER_ID",
    "park_id": "PARK_ID_2",
    "rating": 5,
    "review_content": "Love this place!",
    "createdAt": "2025-01-03T09:00:00.000Z",
    "updatedAt": "2025-01-04T10:30:00.000Z"
  }
]
```

If the user has no reviews, returns `[]`.

**Errors:**
- `400` – invalid user ID.
- `500` – internal server error.
- `404` – User not found.

---



## 8. Comment Module Documentation

This section describes the **Comment** functionality: how comments are stored, created / read / edited / deleted.

All comment routes are prefixed with:
```
/comments
```

### 8.1 Comment Data Model

Comment data is stored in the `comments` collection.

**Schema Example:**
```javascript
{
  _id: ObjectId("..."),
  user_id: ObjectId("..."),             // author of the comment
  review_id: ObjectId("..."),           // the review this comment belongs to
  parent_comment_id: ObjectId("..."),   // optional, null for top-level comments
  comment_content: "I totally agree!",
  createdAt: ISODate("2025-01-01T00:00:00Z"),
  updatedAt: ISODate("2025-01-02T10:30:00Z") // null or timestamp
}
```


### 8.2 Comment JSON API Overview

All endpoints below assume the base path `/comments`.

| Method   | Path                         | Auth       | Description                                        |
|----------|------------------------------|------------|----------------------------------------------------|
| GET      | `/comments/:commentId`       | Public     | Get a single comment by ID                         |
| DELETE   | `/comments/:commentId`       | Logged-in  | Delete a comment (author or admin only)            |
| PUT      | `/comments/:commentId`       | Logged-in  | Update a comment's content (author or admin)       |
| GET      | `/comments/review/:reviewId` | Public     | List all comments for a given review               |
| POST     | `/comments/review/:reviewId` | Logged-in  | Add a comment or reply under a review              |
| GET      | `/comments/user/:userId`     | Public     | List all comments made by a given user             |

**Common error codes:**

- `400` – Invalid ID or bad request body.
- `401` – Not authenticated (no session).
- `403` – Not allowed (not your comment and not admin).
- `404` – Comment or review not found.
- `500` – Internal server error.

### 8.3 Endpoints in Detail

#### 1. Get Comment by ID

**`GET /comments/:commentId`**

Fetch a single comment document.

**Path params:**
- `commentId` – string, MongoDB ObjectId.

**Response 200:**
```json
{
  "_id": "COMMENT_ID",
  "user_id": "USER_ID",
  "review_id": "REVIEW_ID",
  "parent_comment_id": null,
  "comment_content": "Nice review!",
  "createdAt": "2025-01-01T00:00:00.000Z",
  "updatedAt": null
}
```

**Errors:**
- `400` – invalid commentId.
- `404` – no comment with this ID.
- `500` – internal server error.

#### 2. Delete Comment

**`DELETE /comments/:commentId`**

Delete a comment. Only:
- The author of the comment, or
- An admin user (`req.session.user.role === 'admin'`)
can delete.

- **Auth:** Required. Uses `req.session.user`.

**Path params:**
- `commentId` – comment to delete.

**Typical Response 200:**
```json
{
  "comment_id": "COMMENT_ID",
  "deleted": true
}
```

**Errors:**
- `401` – not logged in.
- `400` – invalid commentId.
- `403` – trying to delete someone else's comment and not admin.
- `404` – comment not found.
- `500` – internal server error.

#### 3. Update Comment

**`PUT /comments/:commentId`**

Update the text content of a comment.

Only:
- The author, or
- An admin
may edit.

- **Auth:** Required.

**Path params:**
- `commentId` – comment to update.

**Request body:**
```json
{
  "comment_content": "Updated content of this comment"
}
```

**Response 200:**
```json
{
  "_id": "COMMENT_ID",
  "user_id": "USER_ID",
  "review_id": "REVIEW_ID",
  "parent_comment_id": null,
  "comment_content": "Updated content of this comment",
  "createdAt": "2025-01-01T00:00:00.000Z",
  "updatedAt": "2025-01-02T11:22:33.000Z"
}
```

**Errors:**
- `401` – not logged in.
- `400` – invalid commentId or invalid comment_content.
- `403` – not author and not admin.
- `404` – comment not found.
- `500` – internal server error.


#### 4. Get Comments for a Review

**`GET /comments/review/:reviewId`**

Return all comments under a given review.

- **Auth:** Public (no login required).

**Path params:**
- `reviewId` – review whose comments you want.

**Response 200:**
```json
[
  {
    "_id": "COMMENT_ID_1",
    "user_id": "USER_ID_1",
    "review_id": "REVIEW_ID",
    "parent_comment_id": null,
    "comment_content": "Top-level comment",
    "createdAt": "2025-01-01T00:00:00.000Z",
    "updatedAt": null
  },
  {
    "_id": "COMMENT_ID_2",
    "user_id": "USER_ID_2",
    "review_id": "REVIEW_ID",
    "parent_comment_id": "COMMENT_ID_1",
    "comment_content": "Reply to COMMENT_ID_1",
    "createdAt": "2025-01-01T01:00:00.000Z",
    "updatedAt": null
  }
]
```

**Errors:**
- `400` – invalid reviewId.
- `404` – review not found (thrown inside data layer and mapped to 404).
- `500` – internal server error.

#### 5. Add Comment (Top-Level or Reply)

**`POST /comments/review/:reviewId`**

Create a new comment under a review.

- If `parentCommentId` is not provided → top-level comment on the review.
- If `parentCommentId` is provided → reply to an existing comment.

- **Auth:** Required (must be logged in).

The user ID comes from session:
```javascript
req.session.user._id
```

**Path params:**
- `reviewId` – the review being commented on.

**Request body (top-level comment):**
```json
{
  "comment_content": "This is my comment"
}
```

**Request body (reply to another comment):**
```json
{
  "comment_content": "I agree with you!",
  "parentCommentId": "PARENT_COMMENT_ID"
}
```

**Response 200 (example):**
```json
{
  "_id": "NEW_COMMENT_ID",
  "user_id": "USER_ID",
  "review_id": "REVIEW_ID",
  "parent_comment_id": null,
  "comment_content": "This is my comment",
  "createdAt": "2025-01-01T10:00:00.000Z",
  "updatedAt": null
}
```

**Errors:**
- `401` – not logged in.
- `400` – invalid review ID, invalid parent comment ID, or bad comment content.
- `404` – review not found, or parent comment not found.
- `409` – (if enforced) duplicate–comment rule from data layer (e.g., already replied in a certain way).
- `500` – internal server error.


#### 6. Get Comments by User

**`GET /comments/user/:userId`**

Return all comments written by a specific user.

- **Auth:** Public.

**Path params:**
- `userId` – user whose comments to fetch.

**Response 200 (example):**
```json
[
  {
    "_id": "COMMENT_ID_1",
    "user_id": "USER_ID",
    "review_id": "REVIEW_ID_1",
    "parent_comment_id": null,
    "comment_content": "First comment",
    "createdAt": "2025-01-01T00:00:00.000Z",
    "updatedAt": null
  },
  {
    "_id": "COMMENT_ID_2",
    "user_id": "USER_ID",
    "review_id": "REVIEW_ID_2",
    "parent_comment_id": "ANOTHER_COMMENT_ID",
    "comment_content": "Reply on another review",
    "createdAt": "2025-01-03T12:00:00.000Z",
    "updatedAt": null
  }
]
```

**Errors:**
- `400` – invalid userId.
- `404` – User not found
- `500` – internal server error.

### 8.4 How to Use Comment APIs

**To show comments under a review page:**
- Call `GET /comments/review/:reviewId` on page load.
- Render list or nested thread based on `parent_comment_id`.

**To add a new top-level comment:**
- Use `POST /comments/review/:reviewId` with `{ comment_content }`.

**To reply to a comment:**
- Use `POST /comments/review/:reviewId` with `{ comment_content, parentCommentId }`.

**To edit a comment:**
- Use `PUT /comments/:commentId` with `{ comment_content }`.

**To delete a comment:**
- Use `DELETE /comments/:commentId`.



## 9. License

This project is for CS546 final group project purposes only.
