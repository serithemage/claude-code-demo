---
name: route-tester
description: Test API routes in the RealWorld (Conduit) project using JWT authentication. Use this skill when testing API endpoints, validating route functionality, or debugging authentication issues. Includes patterns for testing with curl and mock data.
---

# RealWorld Route Tester Skill

## Purpose
This skill provides patterns for testing API routes in the RealWorld (Conduit) project using JWT-based authentication.

## When to Use This Skill
- Testing new API endpoints
- Validating route functionality after changes
- Debugging authentication issues
- Testing POST/PUT/DELETE operations
- Verifying request/response data

## RealWorld Authentication Overview

The RealWorld project uses:
- **JWT tokens** stored in localStorage (frontend)
- **Authorization header**: `Token jwt.token.here`
- **bcrypt** for password hashing
- Token stored in user response after login/register

## Testing Methods

### Method 1: curl with JWT Token (RECOMMENDED)

#### Step 1: Get a Token via Login

```bash
# Login to get JWT token
curl -X POST http://localhost:3000/api/users/login \
  -H "Content-Type: application/json" \
  -d '{"user":{"email":"test@test.com","password":"testpassword"}}'
```

Response contains the token:
```json
{
  "user": {
    "email": "test@test.com",
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "username": "testuser",
    "bio": null,
    "image": null
  }
}
```

#### Step 2: Use Token for Authenticated Requests

```bash
# Use the token from login response
TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

# Get current user
curl -X GET http://localhost:3000/api/user \
  -H "Authorization: Token $TOKEN"

# Create article
curl -X POST http://localhost:3000/api/articles \
  -H "Content-Type: application/json" \
  -H "Authorization: Token $TOKEN" \
  -d '{"article":{"title":"Test Article","description":"About testing","body":"Testing content","tagList":["test"]}}'
```

### Method 2: Register New User

```bash
# Register new user
curl -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -d '{"user":{"username":"newuser","email":"new@test.com","password":"password123"}}'
```

## Common Testing Patterns

### Authentication API

```bash
# Register
curl -X POST http://localhost:3000/api/users \
  -H "Content-Type: application/json" \
  -d '{"user":{"username":"testuser","email":"test@test.com","password":"testpassword"}}'

# Login
curl -X POST http://localhost:3000/api/users/login \
  -H "Content-Type: application/json" \
  -d '{"user":{"email":"test@test.com","password":"testpassword"}}'

# Get current user (authenticated)
curl -X GET http://localhost:3000/api/user \
  -H "Authorization: Token $TOKEN"

# Update user (authenticated)
curl -X PUT http://localhost:3000/api/user \
  -H "Content-Type: application/json" \
  -H "Authorization: Token $TOKEN" \
  -d '{"user":{"bio":"Updated bio","image":"https://example.com/image.jpg"}}'
```

### Articles API

```bash
# List articles (no auth required)
curl -X GET "http://localhost:3000/api/articles?limit=10&offset=0"

# List articles by author
curl -X GET "http://localhost:3000/api/articles?author=testuser"

# List articles by tag
curl -X GET "http://localhost:3000/api/articles?tag=reactjs"

# Get feed (authenticated)
curl -X GET http://localhost:3000/api/articles/feed \
  -H "Authorization: Token $TOKEN"

# Get single article
curl -X GET http://localhost:3000/api/articles/how-to-train-your-dragon

# Create article (authenticated)
curl -X POST http://localhost:3000/api/articles \
  -H "Content-Type: application/json" \
  -H "Authorization: Token $TOKEN" \
  -d '{"article":{"title":"New Article","description":"Description","body":"Content","tagList":["tag1","tag2"]}}'

# Update article (authenticated, author only)
curl -X PUT http://localhost:3000/api/articles/new-article \
  -H "Content-Type: application/json" \
  -H "Authorization: Token $TOKEN" \
  -d '{"article":{"title":"Updated Title"}}'

# Delete article (authenticated, author only)
curl -X DELETE http://localhost:3000/api/articles/new-article \
  -H "Authorization: Token $TOKEN"
```

### Comments API

```bash
# Get comments (no auth required)
curl -X GET http://localhost:3000/api/articles/how-to-train-your-dragon/comments

# Create comment (authenticated)
curl -X POST http://localhost:3000/api/articles/how-to-train-your-dragon/comments \
  -H "Content-Type: application/json" \
  -H "Authorization: Token $TOKEN" \
  -d '{"comment":{"body":"Great article!"}}'

# Delete comment (authenticated, author only)
curl -X DELETE http://localhost:3000/api/articles/how-to-train-your-dragon/comments/1 \
  -H "Authorization: Token $TOKEN"
```

### Favorites API

```bash
# Favorite article (authenticated)
curl -X POST http://localhost:3000/api/articles/how-to-train-your-dragon/favorite \
  -H "Authorization: Token $TOKEN"

# Unfavorite article (authenticated)
curl -X DELETE http://localhost:3000/api/articles/how-to-train-your-dragon/favorite \
  -H "Authorization: Token $TOKEN"
```

### Profile API

```bash
# Get profile (optional auth)
curl -X GET http://localhost:3000/api/profiles/testuser

# Follow user (authenticated)
curl -X POST http://localhost:3000/api/profiles/testuser/follow \
  -H "Authorization: Token $TOKEN"

# Unfollow user (authenticated)
curl -X DELETE http://localhost:3000/api/profiles/testuser/follow \
  -H "Authorization: Token $TOKEN"
```

### Tags API

```bash
# Get tags (no auth required)
curl -X GET http://localhost:3000/api/tags
```

## Service Ports

| Service | Port | Base URL |
|---------|------|----------|
| Backend | 3000 | http://localhost:3000/api |
| Frontend | 5173 | http://localhost:5173 |

## Testing Checklist

Before testing a route:

- [ ] Ensure backend is running (`pnpm --filter backend dev`)
- [ ] Identify if authentication is required
- [ ] Get a valid JWT token if needed
- [ ] Prepare request body (if POST/PUT)
- [ ] Run the test
- [ ] Verify response status and data
- [ ] Check database changes if applicable

## Verifying Database Changes

After testing routes that modify data:

```bash
# Using Prisma Studio
cd backend && npx prisma studio

# Or using SQLite CLI
sqlite3 backend/prisma/dev.db

# Check specific table
sqlite> SELECT * FROM User ORDER BY createdAt DESC LIMIT 5;
sqlite> SELECT * FROM Article ORDER BY createdAt DESC LIMIT 5;
sqlite> SELECT * FROM Comment WHERE articleId = 'uuid-here';
```

## Debugging Failed Tests

### 401 Unauthorized

**Possible causes**:
1. Token expired
2. Token not provided
3. Invalid token format (must be `Token <jwt>`, not `Bearer <jwt>`)

**Solutions**:
```bash
# Re-login to get fresh token
curl -X POST http://localhost:3000/api/users/login \
  -H "Content-Type: application/json" \
  -d '{"user":{"email":"test@test.com","password":"testpassword"}}'
```

### 403 Forbidden

**Possible causes**:
1. Trying to edit/delete another user's resource
2. Not the author of article/comment

**Solutions**:
- Use the correct user's token
- Check resource ownership

### 404 Not Found

**Possible causes**:
1. Incorrect URL or slug
2. Resource doesn't exist

**Solutions**:
1. Verify the endpoint URL
2. Check the slug is correct
3. Ensure the resource was created

### 422 Unprocessable Entity

**Possible causes**:
1. Validation error
2. Missing required fields
3. Invalid field format

**Solutions**:
1. Check request body format
2. Verify required fields are present
3. Check field validation rules in API-Spec.md

## Response Format

### Success Responses

```json
{
  "user": {...},      // For user endpoints
  "article": {...},   // For single article
  "articles": [...],  // For article list
  "articlesCount": 10,
  "profile": {...},
  "comment": {...},
  "comments": [...],
  "tags": [...]
}
```

### Error Responses

```json
{
  "errors": {
    "body": ["can't be empty"],
    "email": ["has already been taken"]
  }
}
```

## Key Files

- `backend/src/routes/` - Route definitions
- `backend/src/controllers/` - Request handlers
- `backend/src/middleware/auth.ts` - JWT verification
- `docs/API-Spec.md` - Complete API specification

## Related Skills

- **backend-dev-guidelines** - Backend development patterns
- **error-tracking** - Sentry error tracking
