# Requirements Specification: Airbnb Clone Backend

## Table of Contents
1. [User Authentication System](#user-authentication-system)
2. [Property Management System](#property-management-system)
3. [Booking System](#booking-system)

## User Authentication System

### Overview
Secure user authentication and authorization system supporting multiple login methods and role-based access control.

### Functional Requirements
1. User registration with email/password  
2. Social authentication (Google, Facebook)  
3. Email verification  
4. Password reset functionality  
5. JWT-based authentication  
6. Role-based access control (Guest, Host, Admin)  
7. Session management  
8. Profile management

### API Endpoints

#### POST /api/auth/register
Creates a new user account.

**Input:**
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123!",
  "firstName": "John",
  "lastName": "Doe",
  "userType": "guest"
}
```

**Output (Success):**
```json
{
  "message": "User registered successfully",
  "userId": "12345",
  "verificationRequired": true
}
```

**Validation Rules:**
- Email must be valid format  
- Password minimum 8 characters with uppercase, lowercase, number, and special character  
- Required fields: email, password, firstName, lastName  
- UserType must be either "guest" or "host"

#### POST /api/auth/login
Authenticates a user and returns JWT tokens.

**Input:**
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123!"
}
```

**Output (Success):**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "userId": "12345",
  "userType": "guest"
}
```

#### POST /api/auth/verify-email
Verifies user's email address.

**Input:**
```json
{
  "token": "email_verification_token"
}
```

#### POST /api/auth/forgot-password
Initiates password reset process.

**Input:**
```json
{
  "email": "user@example.com"
}
```

#### POST /api/auth/reset-password
Resets user password with token.

**Input:**
```json
{
  "token": "password_reset_token",
  "newPassword": "NewSecurePassword123!"
}
```

### Performance Criteria
- Registration API response time: < 500ms  
- Login API response time: < 300ms  
- Support up to 1000 concurrent authentication requests  
- JWT token generation time: < 100ms  
- Password hashing using bcrypt with salt rounds: 12

### Security Requirements
- Passwords must be hashed using bcrypt  
- JWT tokens expire after 15 minutes (access), 7 days (refresh)  
- Rate limiting: 5 attempts per 15 minutes for login  
- HTTPS enforcement  
- Password reset tokens expire after 1 hour  


## Property Management System

### Overview
Comprehensive system for hosts to create, manage, and list properties with availability calendar, pricing, and media management.

### Functional Requirements
- Property creation and editing  
- Media upload and management  
- Availability calendar management  
- Pricing configuration (base price, seasonal pricing, discounts)  
- Amenity management  
- Location services integration  
- Property search and filtering  
- Review and rating system

### API Endpoints

#### POST /api/properties
Creates a new property listing.

**Input:**
```json
{
  "title": "Beautiful Beach House",
  "description": "Luxury beachfront property with amazing views",
  "type": "house",
  "pricePerNight": 150,
  "bedrooms": 3,
  "bathrooms": 2,
  "maxGuests": 6,
  "amenities": ["wifi", "pool", "parking"],
  "location": {
    "address": "123 Beach Road",
    "city": "Miami",
    "state": "FL",
    "country": "USA",
    "zipCode": "33139",
    "latitude": 25.7617,
    "longitude": -80.1918
  }
}
```

**Output (Success):**
```json
{
  "propertyId": "prop_12345",
  "message": "Property created successfully"
}
```

#### GET /api/properties
Retrieves properties with filtering and pagination.

**Query Parameters:**
- location (string)  
- checkIn (date)  
- checkOut (date)  
- guests (number)  
- minPrice (number)  
- maxPrice (number)  
- amenities (array)  
- type (string)  
- page (number)  
- limit (number)

**Output:**
```json
{
  "properties": [
    {
      "id": "prop_12345",
      "title": "Beautiful Beach House",
      "type": "house",
      "pricePerNight": 150,
      "rating": 4.8,
      "reviewCount": 45,
      "thumbnail": "https://example.com/image.jpg",
      "location": {
        "city": "Miami",
        "state": "FL"
      }
    }
  ],
  "totalCount": 125,
  "currentPage": 1,
  "totalPages": 13
}
```

#### PUT /api/properties/{propertyId}
Updates a property listing.

**Input:**
```json
{
  "pricePerNight": 160,
  "description": "Updated description with more details"
}
```

#### POST /api/properties/{propertyId}/availability
Sets property availability.

**Input:**
```json
{
  "date": "2023-12-15",
  "available": true,
  "price": 175
}
```

#### POST /api/properties/{propertyId}/images
Uploads property images.

**Input:** Multipart form data with image files

**Output:**
```json
{
  "imageUrls": [
    "https://bucket.s3.amazonaws.com/prop_12345/image1.jpg",
    "https://bucket.s3.amazonaws.com/prop_12345/image2.jpg"
  ]
}
```

### Validation Rules
- Property title: 10-100 characters  
- Description: 50-2000 characters  
- Price per night: minimum $10, maximum $10,000  
- Location coordinates must be valid  
- Maximum 10 images per property  
- Image files: max 5MB each, only jpg, jpeg, png formats

### Performance Criteria
- Property search response time: < 200ms  
- Property creation: < 500ms  
- Image upload: < 2s per image  
- Support up to 10,000 properties in database  
- Search should handle 100+ concurrent requests

### Data Storage
- Properties stored in PostgreSQL with PostGIS extension for location queries  
- Images stored in AWS S3 with CloudFront CDN  
- Availability calendar stored in optimized format for quick date range queries


## Booking System

### Overview
Comprehensive booking management system handling reservations, payments, cancellations, and calendar synchronization.

### Functional Requirements
- Availability checking  
- Reservation creation  
- Payment processing integration  
- Booking confirmation  
- Cancellation management  
- Host approval workflow  
- Calendar synchronization  
- Booking modifications

### API Endpoints

#### GET /api/bookings/availability/{propertyId}
Checks availability for specific dates.

**Query Parameters:**
- checkIn (required, date)  
- checkOut (required, date)  
- guests (required, number)

**Output:**
```json
{
  "available": true,
  "totalPrice": 1050,
  "breakdown": {
    "nights": 7,
    "pricePerNight": 150,
    "cleaningFee": 75,
    "serviceFee": 105,
    "tax": 120
  }
}
```

#### POST /api/bookings
Creates a new booking.

**Input:**
```json
{
  "propertyId": "prop_12345",
  "checkIn": "2023-12-15",
  "checkOut": "2023-12-22",
  "guests": 4,
  "paymentMethodId": "pm_12345",
  "guestInfo": {
    "firstName": "John",
    "lastName": "Doe",
    "email": "john@example.com",
    "phone": "+1234567890"
  }
}
```

**Output (Success):**
```json
{
  "bookingId": "book_67890",
  "status": "confirmed",
  "totalAmount": 1050,
  "confirmationCode": "ABC123XYZ"
}
```

#### GET /api/bookings/{bookingId}
Retrieves booking details.

**Output:**
```json
{
  "id": "book_67890",
  "propertyId": "prop_12345",
  "propertyTitle": "Beautiful Beach House",
  "checkIn": "2023-12-15",
  "checkOut": "2023-12-22",
  "guests": 4,
  "status": "confirmed",
  "totalAmount": 1050,
  "guest": {
    "firstName": "John",
    "lastName": "Doe",
    "email": "john@example.com"
  },
  "host": {
    "firstName": "Jane",
    "lastName": "Smith"
  },
  "createdAt": "2023-11-10T14:30:00Z"
}
```

#### POST /api/bookings/{bookingId}/cancel
Cancels a booking.

**Input:**
```json
{
  "reason": "Change of plans"
}
```

**Output:**
```json
{
  "bookingId": "book_67890",
  "status": "cancelled",
  "refundAmount": 945,
  "cancellationFee": 105
}
```

#### GET /api/users/{userId}/bookings
Retrieves user's booking history.

**Query Parameters:**
- status (optional: upcoming, completed, cancelled)  
- page (number)  
- limit (number)

### Validation Rules
- Check-in date must be at least 24 hours in the future  
- Check-out date must be after check-in date  
- Guests count cannot exceed property maximum  
- Booking cannot overlap with existing reservations  
- Cancellation must follow property's cancellation policy

### Payment Processing
- Integration with Stripe/PayPal/M-Pesa 
- Support for multiple payment methods  
- Secure handling of payment information  
- Refund processing according to cancellation policy

### Performance Criteria
- Availability check response time: < 100ms  
- Booking creation: < 500ms  
- Payment processing: < 2s  
- Support 500 concurrent booking requests  
- Handle peak booking periods (holidays, events)

### Business Rules
- 24-hour confirmation window for hosts to approve instant bookings  
- Different cancellation policies (flexible, moderate, strict)  
- Automatic booking confirmation for properties with instant booking enabled  
- Cleaning and service fees calculated as percentage of booking total  
- Tax calculation based on property location

### Notification System
- Email confirmation to guest upon booking  
- Notification to host about new booking  
- Reminder emails 48 hours before check-in  
- Follow-up email after checkout requesting review