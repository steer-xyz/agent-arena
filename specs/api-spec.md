# Agentic Arena - API Specification

## API Overview
The Agentic Arena API is a RESTful service built with FastAPI, providing endpoints for user management, workflow submissions, evaluations, and leaderboards.

**Base URL**: `https://api.agenticarena.com/v1`
**Authentication**: JWT Bearer tokens
**Content-Type**: `application/json` (unless specified otherwise)

## Authentication

All authenticated endpoints require a JWT token in the Authorization header:
```
Authorization: Bearer <jwt_token>
```

### Authentication Endpoints

#### POST /auth/register
Register a new user account.

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "securePassword123",
  "username": "johndoe",
  "full_name": "John Doe"
}
```

**Response (201)**:
```json
{
  "id": "uuid",
  "email": "user@example.com",
  "username": "johndoe",
  "full_name": "John Doe",
  "created_at": "2024-01-01T00:00:00Z",
  "access_token": "jwt_token_here",
  "refresh_token": "refresh_token_here"
}
```

#### POST /auth/login
Authenticate user and receive access token.

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Response (200)**:
```json
{
  "access_token": "jwt_token_here",
  "refresh_token": "refresh_token_here",
  "token_type": "bearer",
  "expires_in": 3600,
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "username": "johndoe"
  }
}
```

#### POST /auth/refresh
Refresh access token using refresh token.

**Request Body**:
```json
{
  "refresh_token": "refresh_token_here"
}
```

#### POST /auth/forgot-password
Request password reset email.

**Request Body**:
```json
{
  "email": "user@example.com"
}
```

## User Management

#### GET /users/me
Get current user profile.

**Headers**: `Authorization: Bearer <token>`

**Response (200)**:
```json
{
  "id": "uuid",
  "email": "user@example.com",
  "username": "johndoe",
  "full_name": "John Doe",
  "bio": "AI developer and prompt engineer",
  "avatar_url": "https://cdn.example.com/avatars/johndoe.jpg",
  "created_at": "2024-01-01T00:00:00Z",
  "stats": {
    "total_submissions": 15,
    "best_ranking": 3,
    "total_tokens_saved": 12500
  }
}
```

#### PUT /users/me
Update current user profile.

**Request Body**:
```json
{
  "full_name": "John Doe Updated",
  "bio": "Senior AI developer",
  "avatar_url": "https://cdn.example.com/avatars/new-avatar.jpg"
}
```

#### GET /users/{user_id}
Get public user profile.

**Response (200)**:
```json
{
  "id": "uuid",
  "username": "johndoe",
  "full_name": "John Doe",
  "bio": "AI developer and prompt engineer",
  "avatar_url": "https://cdn.example.com/avatars/johndoe.jpg",
  "joined_at": "2024-01-01T00:00:00Z",
  "stats": {
    "total_submissions": 15,
    "best_ranking": 3,
    "public_submissions": 12
  }
}
```

## Workflow Management

#### POST /workflows/submit
Submit a new workflow for evaluation.

**Content-Type**: `multipart/form-data`

**Form Fields**:
- `file`: Workflow file (JSON, TXT, ZIP)
- `title`: Workflow title
- `description`: Workflow description
- `tool_type`: AI tool used (claude_code, cursor, bolt44, etc.)
- `task_id`: Benchmark task ID
- `is_public`: Boolean for public visibility

**Response (201)**:
```json
{
  "id": "uuid",
  "title": "Efficient React Component Creation",
  "description": "Using Claude Code to build a data visualization component",
  "tool_type": "claude_code",
  "task_id": "uuid",
  "status": "processing",
  "file_size": 1024000,
  "submitted_at": "2024-01-01T00:00:00Z",
  "estimated_processing_time": 120
}
```

#### GET /workflows
Get list of workflows with filtering and pagination.

**Query Parameters**:
- `page`: Page number (default: 1)
- `limit`: Items per page (default: 20, max: 100)
- `tool_type`: Filter by AI tool
- `task_id`: Filter by benchmark task
- `user_id`: Filter by user
- `status`: Filter by processing status
- `sort_by`: Sort field (created_at, ranking, tokens_used)
- `sort_order`: asc or desc

**Response (200)**:
```json
{
  "items": [
    {
      "id": "uuid",
      "title": "Efficient React Component Creation",
      "user": {
        "id": "uuid",
        "username": "johndoe",
        "avatar_url": "https://cdn.example.com/avatars/johndoe.jpg"
      },
      "tool_type": "claude_code",
      "task_name": "Build React Component",
      "status": "evaluated",
      "metrics": {
        "total_tokens": 2500,
        "tool_calls": 8,
        "execution_time": 145,
        "estimated_cost": 0.025
      },
      "ranking": 7,
      "submitted_at": "2024-01-01T00:00:00Z",
      "votes": {
        "upvotes": 12,
        "downvotes": 2
      }
    }
  ],
  "total": 150,
  "page": 1,
  "limit": 20,
  "total_pages": 8
}
```

#### GET /workflows/{workflow_id}
Get detailed workflow information.

**Response (200)**:
```json
{
  "id": "uuid",
  "title": "Efficient React Component Creation",
  "description": "Using Claude Code to build a data visualization component with minimal token usage",
  "user": {
    "id": "uuid",
    "username": "johndoe",
    "full_name": "John Doe",
    "avatar_url": "https://cdn.example.com/avatars/johndoe.jpg"
  },
  "tool_type": "claude_code",
  "task": {
    "id": "uuid",
    "name": "Build React Component",
    "description": "Create a reusable React component for data visualization",
    "category": "frontend_development"
  },
  "status": "evaluated",
  "metrics": {
    "total_tokens": 2500,
    "input_tokens": 1200,
    "output_tokens": 1300,
    "tool_calls": 8,
    "execution_time": 145,
    "estimated_cost": 0.025,
    "iterations": 3
  },
  "ranking": {
    "overall": 7,
    "by_tokens": 3,
    "by_speed": 12,
    "by_cost": 5
  },
  "workflow_steps": [
    {
      "step": 1,
      "type": "user_prompt",
      "content": "Create a React component for displaying charts",
      "timestamp": "2024-01-01T10:00:00Z"
    },
    {
      "step": 2,
      "type": "tool_call",
      "tool": "create_file",
      "input": {"path": "Chart.jsx", "content": "..."},
      "output": "File created successfully",
      "tokens_used": 450,
      "timestamp": "2024-01-01T10:00:15Z"
    }
  ],
  "submitted_at": "2024-01-01T00:00:00Z",
  "evaluated_at": "2024-01-01T00:02:30Z",
  "is_public": true,
  "votes": {
    "upvotes": 12,
    "downvotes": 2,
    "user_vote": "upvote"
  },
  "download_url": "https://cdn.example.com/workflows/uuid/original.zip"
}
```

#### POST /workflows/{workflow_id}/vote
Vote on a workflow (upvote/downvote).

**Request Body**:
```json
{
  "vote_type": "upvote"
}
```

#### DELETE /workflows/{workflow_id}
Delete own workflow submission.

**Response (204)**: No content

## Leaderboard

#### GET /leaderboard
Get leaderboard rankings with filtering options.

**Query Parameters**:
- `task_id`: Filter by specific benchmark task
- `tool_type`: Filter by AI tool
- `metric`: Ranking metric (tokens, speed, cost, overall)
- `period`: Time period (week, month, all_time)
- `limit`: Number of results (default: 50)

**Response (200)**:
```json
{
  "rankings": [
    {
      "rank": 1,
      "workflow_id": "uuid",
      "title": "Minimal Token React Component",
      "user": {
        "username": "efficiency_expert",
        "avatar_url": "https://cdn.example.com/avatars/expert.jpg"
      },
      "tool_type": "claude_code",
      "metrics": {
        "total_tokens": 1200,
        "tool_calls": 4,
        "execution_time": 89,
        "estimated_cost": 0.012
      },
      "score": 98.5,
      "submitted_at": "2024-01-01T00:00:00Z"
    }
  ],
  "metadata": {
    "task_name": "Build React Component",
    "total_submissions": 45,
    "ranking_metric": "tokens",
    "last_updated": "2024-01-01T12:00:00Z"
  }
}
```

#### GET /leaderboard/tasks
Get list of all benchmark tasks.

**Response (200)**:
```json
{
  "tasks": [
    {
      "id": "uuid",
      "name": "Build Python CLI",
      "description": "Create a command-line tool that fetches Hacker News stories",
      "category": "backend_development",
      "difficulty": "intermediate",
      "total_submissions": 23,
      "avg_tokens": 3400,
      "success_rate": 0.87
    }
  ]
}
```

## Comments

#### GET /workflows/{workflow_id}/comments
Get comments for a workflow.

**Query Parameters**:
- `page`: Page number
- `limit`: Comments per page

**Response (200)**:
```json
{
  "comments": [
    {
      "id": "uuid",
      "user": {
        "username": "commenter",
        "avatar_url": "https://cdn.example.com/avatars/commenter.jpg"
      },
      "content": "Great approach! The token efficiency is impressive.",
      "created_at": "2024-01-01T01:00:00Z",
      "upvotes": 5,
      "replies": [
        {
          "id": "uuid",
          "user": {
            "username": "johndoe",
            "avatar_url": "https://cdn.example.com/avatars/johndoe.jpg"
          },
          "content": "Thanks! I focused on minimal prompting.",
          "created_at": "2024-01-01T01:15:00Z",
          "upvotes": 2
        }
      ]
    }
  ],
  "total": 8,
  "page": 1
}
```

#### POST /workflows/{workflow_id}/comments
Add a comment to a workflow.

**Request Body**:
```json
{
  "content": "This workflow shows excellent token efficiency!",
  "parent_id": null
}
```

## Search

#### GET /search
Search workflows by title, description, and tags.

**Query Parameters**:
- `q`: Search query
- `tool_type`: Filter by AI tool
- `task_id`: Filter by task
- `sort_by`: Sort by relevance, date, ranking
- `page`: Page number
- `limit`: Results per page

**Response (200)**:
```json
{
  "results": [
    {
      "id": "uuid",
      "title": "Efficient React Component Creation",
      "description": "...",
      "user": {
        "username": "johndoe"
      },
      "tool_type": "claude_code",
      "ranking": 7,
      "relevance_score": 0.89
    }
  ],
  "total": 25,
  "query": "react component efficient",
  "suggestions": ["react hooks", "component optimization"]
}
```

## Admin Endpoints

#### POST /admin/tasks
Create a new benchmark task (admin only).

**Request Body**:
```json
{
  "name": "Build FastAPI Endpoint",
  "description": "Create a REST API endpoint with validation and error handling",
  "category": "backend_development",
  "difficulty": "intermediate",
  "success_criteria": [
    "Endpoint returns correct HTTP status codes",
    "Input validation works properly",
    "Error handling is comprehensive"
  ],
  "test_cases": [
    {
      "input": {"name": "test", "age": 25},
      "expected_output": {"status": "success", "id": "uuid"}
    }
  ]
}
```

## Error Responses

All error responses follow this format:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  },
  "request_id": "uuid"
}
```

### Common Error Codes

- `400 BAD_REQUEST`: Invalid request data
- `401 UNAUTHORIZED`: Missing or invalid authentication
- `403 FORBIDDEN`: Insufficient permissions
- `404 NOT_FOUND`: Resource not found
- `409 CONFLICT`: Resource conflict (e.g., duplicate username)
- `413 PAYLOAD_TOO_LARGE`: File size exceeds limit
- `422 VALIDATION_ERROR`: Request validation failed
- `429 RATE_LIMIT_EXCEEDED`: Too many requests
- `500 INTERNAL_ERROR`: Server error

## Rate Limiting

- **General API**: 1000 requests per hour per user
- **File Upload**: 10 uploads per hour per user
- **Comment Creation**: 50 comments per hour per user
- **Search**: 500 searches per hour per user

Rate limit headers included in responses:
- `X-RateLimit-Limit`: Request limit per window
- `X-RateLimit-Remaining`: Requests remaining in window
- `X-RateLimit-Reset`: Window reset time (Unix timestamp)

## Pagination

List endpoints use cursor-based pagination:

**Response Headers**:
- `X-Total-Count`: Total number of items
- `X-Page-Count`: Total number of pages

**Response Format**:
```json
{
  "items": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "total_pages": 8,
    "has_next": true,
    "has_previous": false
  }
}
```

## WebSocket Events

Real-time updates for workflow processing and leaderboard changes:

**Connection**: `wss://api.agenticarena.com/ws`

**Authentication**: JWT token as query parameter

**Event Types**:
```json
{
  "type": "workflow_status_update",
  "data": {
    "workflow_id": "uuid",
    "status": "evaluated",
    "ranking": 5
  }
}

{
  "type": "leaderboard_update",
  "data": {
    "task_id": "uuid",
    "new_leader": {
      "username": "speedmaster",
      "workflow_id": "uuid",
      "score": 99.2
    }
  }
}
```

## API Versioning

- Current version: `v1`
- Version specified in URL path: `/api/v1/`
- Backward compatibility maintained for at least 12 months
- Deprecation notices sent via `X-API-Deprecation-Warning` header

## CORS Policy

- Allowed origins: Frontend domains (configured per environment)
- Allowed methods: GET, POST, PUT, DELETE, OPTIONS
- Allowed headers: Authorization, Content-Type, X-Requested-With
- Credentials: Allowed for authenticated requests