#AUC-82
# Story CRUD API Documentation

## Overview
This document provides comprehensive documentation for the Story CRUD API endpoints implemented in the social-be application. The API follows RESTful principles and uses TypeScript with the TSED framework.

## Files Changed/Added

### New Files Added:
1. **`src/adapter/in/rest/StoryController.ts`** - REST controller for story endpoints
2. **`src/application/services/StoryService.ts`** - Business logic service for story operations
3. **`src/domain/ports/in/StoryUseCase.ts`** - Use case interface for story operations
4. **`src/domain/ports/out/data/StoryRepoPort.ts`** - Repository port interface
5. **`src/adapter/out/data/StoryMongoRepo.ts`** - MongoDB repository implementation
6. **`src/models/db/StoryEntity.ts`** - Database entity for stories
7. **`src/models/http-models/StoriesRequest.ts`** - Request DTOs
8. **`src/models/http-models/StoriesResponse.ts`** - Response DTOs

### Files Modified:
1. **`src/constants/Token.ts`** - Added story-related tokens
2. **`src/adapter/providers.ts`** - Added story service and repository providers

### Dependencies
No new packages were added. The implementation uses existing dependencies:
- `@tsed/*` packages for framework functionality
- `typeorm` for database operations
- `mongodb` for MongoDB integration
- `@tsed/schema` for validation and documentation

## API Endpoints

### Base URL
All story endpoints are prefixed with `/story`

---

## 1. List Stories

**Endpoint:** `GET /story/`

**Description:** Retrieves all non-deleted stories ordered by creation date (newest first)

**Request:**
- **Method:** GET
- **Headers:** 
  - `Content-Type: application/json`
- **Body:** None

**Response:**
- **Status Code:** 200 OK
- **Content-Type:** `application/json`
- **Body:** Array of `StoryEntity` objects

**Example Response:**
```json
{
    "stories": [
        {
            "userId": "0198e165-87c7-715f-9056-a65d1d0bdb6d",
            "storyType": "video",
            "mediaId": "66e9f0b3a4d2c1f5e7a90123",
            "overlays": [
                {
                    "type": "text",
                    "value": "Hello World!",
                    "stickerId": null,
                    "position": {
                        "x": 120,
                        "y": 200
                    },
                    "scale": 1,
                    "rotation": 0,
                    "font": "Inter",
                    "color": "#FFFFFF",
                    "backgroundColor": "#000000",
                    "size": 18
                },
                {
                    "type": "sticker",
                    "value": null,
                    "stickerId": "stkr_fire",
                    "position": {
                        "x": 40,
                        "y": 90
                    },
                    "scale": 1.2,
                    "rotation": 25,
                    "font": null,
                    "color": null,
                    "backgroundColor": null,
                    "size": null
                }
            ],
            "viewersCount": 0
        }
    ],
    "users": {
        "0198e165-87c7-715f-9056-a65d1d0bdb6d": {
            "id": "0198e165-87c7-715f-9056-a65d1d0bdb6d",
            "name": null,
            "avatar": null
        }
    }
}
```

---

## 2. Create Story

**Endpoint:** `POST /story/`

**Description:** Creates a new story

**Request:**
- **Method:** POST
- **Headers:** 
  - `Content-Type: application/json`
- **Body:** `CreateStoryRequest` object

**Request Body Schema:**
```typescript
{
  userId: string;                    // Required
  storyType: "image" | "video";     // Required
  mediaId: string;                   // Required
  metadata?: {                       // Optional
    location?: {
      name?: string;
      lat?: number;
      lng?: number;
    };
    hashtags?: string[];
    mentions?: string[];
    privacy?: "public" | "private" | "friends-only";
    expireAt?: Date;
    music?: {
      trackId?: string;
      title?: string;
      artist?: string;
      startTime?: number;
      duration?: number;
    };
  };
  overlays?: {                       // Optional
    type: "text" | "sticker";
    value?: string;
    stickerId?: string;
    position: { x: number; y: number };
    scale: number;
    rotation: number;
    font?: string;
    color?: string;
    backgroundColor?: string;
    size?: number;
  }[];
}
```

**Example Request:**
```json
{
  "userId": "user123",
  "storyType": "image",
  "mediaId": "507f1f77bcf86cd799439012",
  "metadata": {
    "location": {
      "name": "New York",
      "lat": 40.7128,
      "lng": -74.0060
    },
    "hashtags": ["#travel", "#nyc"],
    "mentions": ["@friend1"],
    "privacy": "friends-only",
    "expireAt": "2024-01-15T10:30:00.000Z"
  },
  "overlays": [
    {
      "type": "text",
      "value": "Hello World!",
      "position": { "x": 100, "y": 200 },
      "scale": 1.0,
      "rotation": 0,
      "font": "Arial",
      "color": "#FFFFFF",
      "size": 16
    }
  ]
}
```

**Response:**
- **Status Code:** 201 Created
- **Content-Type:** `application/json`
- **Body:** `CreateStoryResponse` object

**Example Response:**
```json
{
  "userId": "user123",
  "storyType": "image",
  "mediaId": "507f1f77bcf86cd799439012",
  "metadata": {
    "location": {
      "name": "New York",
      "lat": 40.7128,
      "lng": -74.0060
    },
    "hashtags": ["#travel", "#nyc"],
    "mentions": ["@friend1"],
    "privacy": "friends-only",
    "expireAt": "2024-01-15T10:30:00.000Z"
  },
  "overlays": [
    {
      "type": "text",
      "value": "Hello World!",
      "position": { "x": 100, "y": 200 },
      "scale": 1.0,
      "rotation": 0,
      "font": "Arial",
      "color": "#FFFFFF",
      "size": 16
    }
  ]
}
```

---

## 3. Update Story

**Endpoint:** `PUT /story/:storyId`

**Description:** Updates an existing story

**Request:**
- **Method:** PUT
- **Headers:** 
  - `Content-Type: application/json`
- **Path Parameters:**
  - `storyId` (string): The ID of the story to update
- **Body:** `UpdateStoryRequest` object

**Request Body Schema:**
```typescript
{
  storyType?: "image" | "video";    // Optional
  mediaId?: string;                  // Optional
  metadata?: {                       // Optional
    location?: {
      name?: string;
      lat?: number;
      lng?: number;
    };
    hashtags?: string[];
    mentions?: string[];
    privacy?: "public" | "private" | "friends-only";
    expireAt?: Date;
    music?: {
      trackId?: string;
      title?: string;
      artist?: string;
      startTime?: number;
      duration?: number;
    };
  };
  overlays?: {                       // Optional
    type: "text" | "sticker";
    value?: string;
    stickerId?: string;
    position?: { x: number; y: number };
    scale?: number;
    rotation?: number;
    font?: string;
    color?: string;
    backgroundColor?: string;
    size?: number;
  }[];
}
```

**Example Request:**
```json
{
  "metadata": {
    "privacy": "public",
    "hashtags": ["#updated", "#story"]
  },
  "overlays": [
    {
      "type": "sticker",
      "stickerId": "sticker123",
      "position": { "x": 50, "y": 100 },
      "scale": 1.5,
      "rotation": 45
    }
  ]
}
```

**Response:**
- **Status Code:** 200 OK
- **Content-Type:** `application/json`
- **Body:** `UpdateStoryResponse` object

**Example Response (Success):**
```json
{
  "success": true,
  "message": "Story updated successfully",
  "data": {
    "userId": "user123",
    "storyType": "image",
    "mediaId": "507f1f77bcf86cd799439012",
    "metadata": {
      "privacy": "public",
      "hashtags": ["#updated", "#story"]
    },
    "overlays": [
      {
        "type": "sticker",
        "stickerId": "sticker123",
        "position": { "x": 50, "y": 100 },
        "scale": 1.5,
        "rotation": 45
      }
    ]
  }
}
```

**Example Response (Story Not Found):**
```json
{
  "success": false,
  "message": "Story not found"
}
```

---

## 4. Delete Story

**Endpoint:** `DELETE /story/:storyId`

**Description:** Soft deletes a story (sets isDeleted flag to true)

**Request:**
- **Method:** DELETE
- **Headers:** 
  - `Content-Type: application/json`
- **Path Parameters:**
  - `storyId` (string): The ID of the story to delete
- **Body:** None

**Response:**
- **Status Code:** 200 OK
- **Content-Type:** `application/json`
- **Body:** `DeleteStoryResponse` object

**Example Response (Success):**
```json
{
  "success": true,
  "message": "Story deleted successfully"
}
```

**Example Response (Story Not Found):**
```json
{
  "success": false,
  "message": "Story not found"
}
```

---

## Data Models

### StoryEntity (Database Model)
```typescript
{
  id: ObjectId;                      // MongoDB ObjectId
  userId: string;                    // User who created the story
  storyType: "image" | "video";     // Type of media content
  mediaId: ObjectId;                 // Reference to MediaManagementEntity
  metadata: {
    location?: {
      name: string;
      lat: number;
      lng: number;
    };
    hashtags: string[];              // Array of hashtags
    mentions: string[];              // Array of user mentions
    privacy: "public" | "private" | "friends-only";
    expireAt: Date;                  // Story expiration date
    music?: {
      trackId: string;
      title: string;
      artist: string;
      startTime: number;             // Start time in seconds
      duration: number;              // Duration in seconds
    };
  };
  overlays: {
    type: "text" | "sticker";
    value?: string;                  // Text content for text overlays
    stickerId?: string;              // Sticker ID for sticker overlays
    position: { x: number; y: number };
    scale: number;                   // Scale factor
    rotation: number;                // Rotation in degrees
    font?: string;                   // Font family for text
    color?: string;                  // Text color
    backgroundColor?: string;        // Background color
    size?: number;                   // Font size
  }[];
  viewersCount: number;              // Number of viewers
  createdAt: Date;                   // Creation timestamp
  isDeleted: boolean;                // Soft delete flag
}
```

---

## Error Handling

The API uses standard HTTP status codes and includes error messages in the response body:

- **400 Bad Request:** Invalid request data or missing required fields
- **404 Not Found:** Story not found (for update/delete operations)
- **500 Internal Server Error:** Server-side errors

Error responses follow this format:
```json
{
  "success": false,
  "message": "Error description"
}
```

---

## Architecture Notes

### Design Patterns Used:
1. **Clean Architecture:** Separation of concerns with distinct layers (Controller → Service → Repository)
2. **Dependency Injection:** Using TSED's DI container for loose coupling
3. **Repository Pattern:** Abstracting data access through repository interfaces
4. **Use Case Pattern:** Business logic encapsulated in use case interfaces

### Database:
- Uses MongoDB with TypeORM
- Soft delete implementation (isDeleted flag)
- Automatic timestamps (createdAt, updatedAt)

### Validation:
- Uses `@tsed/schema` decorators for request/response validation
- Required and optional field validation
- Type safety with TypeScript

---

## Testing

Currently, no test files are present for the story functionality. Consider adding:
- Unit tests for `StoryService`
- Integration tests for `StoryController`
- Repository tests for `StoryMongoRepo`
- End-to-end API tests

---

## Future Enhancements

Potential improvements for the story API:
1. Add pagination for list stories endpoint
2. Add filtering by user, story type, or date range
3. Add story viewing tracking
4. Add story analytics endpoints
5. Add bulk operations (delete multiple stories)
6. Add story archiving functionality
7. Add story sharing capabilities
