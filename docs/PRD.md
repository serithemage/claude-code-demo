# RealWorld (Conduit) - Product Requirements Document

## 1. Project Overview

### 1.1 Project Name
**Conduit** - Social Blogging Platform

### 1.2 Project Purpose
A full-stack web application that provides article posting/viewing and social features between users as a Medium.com clone. Acquire practical application development skills through implementation compliant with the RealWorld specification.

### 1.3 Reference Specifications
- RealWorld Official Documentation: https://realworld-docs.netlify.app/
- RealWorld API Specification: https://github.com/gothinkster/realworld/tree/main/api

---

## 2. Functional Requirements

### 2.1 Authentication & User Management

#### 2.1.1 User Registration
- New registration with email address, username, and password
- Issue JWT token after registration completion
- Store token in localStorage

#### 2.1.2 Login
- Authentication with email address and password
- Issue JWT token on successful authentication
- Store token in localStorage

#### 2.1.3 Logout
- Logout available from settings page
- Delete token from localStorage

#### 2.1.4 User Information Management
- **Create**: Automatically created during registration
- **Read**: Get current user information
- **Update**: Change profile image, username, bio, email address, password
- **Delete**: Not required (out of scope)

### 2.2 Article Management

#### 2.2.1 Article Creation
- Input title, description, body (Markdown), tag list
- Automatically generate slug on creation
- Automatically assign author information

#### 2.2.2 Article Viewing
- Global Feed: Display all articles in chronological order
- User Feed: Display only articles from followed users
- Tag Filter: Display only articles with specific tags
- Pagination support (default: 10 items/page)

#### 2.2.3 Article Update
- Edit title, description, body
- Only author can edit

#### 2.2.4 Article Deletion
- Only author can delete
- Also delete related comments and favorites

### 2.3 Comment Feature

#### 2.3.1 Comment Creation
- Post comments on articles
- Only logged-in users can post

#### 2.3.2 Comment Viewing
- Display comment list per article
- Display in chronological order

#### 2.3.3 Comment Deletion
- Only comment author can delete
- **Update feature not required** (out of scope)

### 2.4 Social Features

#### 2.4.1 Follow/Unfollow
- Can follow other users
- Display followed users' articles in feed
- Can unfollow

#### 2.4.2 Favorite (Like)
- Add favorite to articles
- Can unfavorite
- Display favorite count per article

### 2.5 Tag Feature
- Assign tags during article creation
- Display popular tags list
- Filter articles by tag

---

## 3. Screen Structure

### 3.1 Common Layout

#### Header (When Not Logged In)
- Conduit logo (link to home)
- Home
- Sign in
- Sign up

#### Header (When Logged In)
- Conduit logo (link to home)
- Home
- New Article
- Settings
- Profile (username + icon)

#### Footer
- Conduit brand
- Copyright notice

### 3.2 Page List

| Route | Page Name | Description |
|-------|-----------|-------------|
| `/#/` | Home | Article feed, popular tags |
| `/#/login` | Login | Login form |
| `/#/register` | Register | Registration form |
| `/#/settings` | Settings | User settings, logout |
| `/#/editor` | New Article | Article creation form |
| `/#/editor/:slug` | Edit Article | Article edit form |
| `/#/article/:slug` | Article Detail | Article body, comments |
| `/#/profile/:username` | Profile | User info, posted articles |
| `/#/profile/:username/favorites` | Favorites | User's favorite articles |

### 3.3 Page Details

#### 3.3.1 Home Page (`/#/`)
- Banner (Conduit logo and description)
- Feed Tabs
  - Your Feed (logged in only, articles from followed users)
  - Global Feed (all articles)
  - #TagName (when tag selected)
- Article Preview List
  - Author icon, name, post date
  - Favorite button and count
  - Title, description
  - Tag list
- Pagination
- Sidebar: Popular tags list

#### 3.3.2 Authentication Pages (`/#/login`, `/#/register`)
- Page title
- Link to other page (Sign in ↔ Sign up)
- Error message list
- Form
  - Registration: username, email, password
  - Login: email, password
- Submit button

#### 3.3.3 Settings Page (`/#/settings`)
- Profile image URL
- Username
- Bio (textarea)
- Email address
- New password
- Update Settings button
- Logout button

#### 3.3.4 Editor Page (`/#/editor`, `/#/editor/:slug`)
- Article title
- Article description (What's this article about?)
- Body (Markdown)
- Tag input (add with Enter)
- Publish Article button

#### 3.3.5 Article Detail Page (`/#/article/:slug`)
- Article Banner
  - Title
  - Author info and follow/favorite buttons
  - Edit/delete buttons (author only)
- Article body (Markdown rendering)
- Tag list
- Author info section (bottom)
- Comment Section
  - Comment input form
  - Comment list (with delete button)

#### 3.3.6 Profile Page (`/#/profile/:username`)
- User Info
  - Profile image
  - Username
  - Bio
  - Follow button / Edit settings button
- Article Tabs
  - My Articles (posted articles)
  - Favorited Articles (favorite articles)
- Article list

---

## 4. API Endpoint Overview

### 4.1 Authentication
| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/api/users/login` | Login | Not required |
| POST | `/api/users` | User registration | Not required |
| GET | `/api/user` | Get current user | Required |
| PUT | `/api/user` | Update user info | Required |

### 4.2 Profile
| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/api/profiles/:username` | Get profile | Optional |
| POST | `/api/profiles/:username/follow` | Follow | Required |
| DELETE | `/api/profiles/:username/follow` | Unfollow | Required |

### 4.3 Articles
| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/api/articles` | Get article list | Optional |
| GET | `/api/articles/feed` | Get feed | Required |
| GET | `/api/articles/:slug` | Get article detail | Optional |
| POST | `/api/articles` | Create article | Required |
| PUT | `/api/articles/:slug` | Update article | Required |
| DELETE | `/api/articles/:slug` | Delete article | Required |

### 4.4 Comments
| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/api/articles/:slug/comments` | Get comment list | Optional |
| POST | `/api/articles/:slug/comments` | Create comment | Required |
| DELETE | `/api/articles/:slug/comments/:id` | Delete comment | Required |

### 4.5 Favorites
| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/api/articles/:slug/favorite` | Add favorite | Required |
| DELETE | `/api/articles/:slug/favorite` | Remove favorite | Required |

### 4.6 Tags
| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/api/tags` | Get tag list | Not required |

---

## 5. Non-Functional Requirements

### 5.1 Performance
- Page load time: Within 3 seconds
- API response time: Within 500ms

### 5.2 Security
- Authentication with JWT tokens
- Password hashing (bcrypt)
- CORS configuration
- XSS protection
- SQL injection protection (Prisma ORM)

### 5.3 Usability
- Responsive design support
- Clear error message display
- Loading state display

### 5.4 Compatibility
- Modern browser support (Chrome, Firefox, Safari, Edge)
- Full compatibility with RealWorld API specification

---

## 6. Glossary

| Term | Description |
|------|-------------|
| Conduit | Application name for the RealWorld project |
| Slug | Article identifier used in URLs (e.g., how-to-train-your-dragon) |
| Feed | List of articles from authors the user follows |
| Global Feed | List of articles from all users |
| Favorite | Article like/favorite feature |
