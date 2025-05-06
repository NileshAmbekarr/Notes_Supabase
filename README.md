# Supabase Notes Service

A minimal Supabase backend for a personal note-taking service, providing secure storage and retrieval of notes with user authentication.

## Schema Design

The database schema is defined in `schema.sql` and includes:

```sql
CREATE TABLE notes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id),
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  status TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'archived')),
  
  CONSTRAINT user_owns_note FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
);
```

### Why This Schema Design?

- **UUID Primary Key**: Provides globally unique identifiers without revealing sequence information, suitable for distributed systems and better for security.
- **User Association**: Each note is linked to a user via `user_id` foreign key, enabling Row Level Security for access control.
- **TEXT for content**: Most appropriate for variable-length note content without unnecessary size constraints.
- **TIMESTAMPTZ for timestamps**: Stores timestamps with timezone information for global consistency.
- **Status field with CHECK constraint**: Provides a controlled vocabulary for note status with validation.
- **CASCADE deletion**: Automatically removes a user's notes when their account is deleted.

## Setup & Deployment

### Prerequisites
- Supabase account
- Supabase CLI installed

### Setup Steps

1. Create a new Supabase project:
   ```
   supabase init
   ```

2. Apply the database schema:
   ```
   supabase db push
   ```

3. Deploy the edge functions:
   ```
   supabase functions deploy post_notes
   supabase functions deploy get_notes
   ```

4. Set required environment variables:
   ```
   supabase secrets set SUPABASE_URL=your-project-url
   supabase secrets set SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
   ```

## API Endpoints

### POST /notes
Creates a new note for the authenticated user.

Example curl command:
```bash
curl -X POST https://your-project-ref.supabase.co/functions/v1/notes \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{"title":"Meeting Notes","content":"Discussed project timeline and deliverables"}'
```

Expected response:
```json
{
  "note": {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "user_id": "98765432-abcd-efgh-ijkl-123456789012",
    "title": "Meeting Notes",
    "content": "Discussed project timeline and deliverables",
    "created_at": "2023-05-06T12:34:56.789Z",
    "updated_at": "2023-05-06T12:34:56.789Z",
    "status": "active"
  }
}
```

### GET /notes
Retrieves all notes for the authenticated user.

Example curl command:
```bash
curl -X GET https://your-project-ref.supabase.co/functions/v1/notes \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

Expected response:
```json
{
  "notes": [
    {
      "id": "123e4567-e89b-12d3-a456-426614174000",
      "user_id": "98765432-abcd-efgh-ijkl-123456789012",
      "title": "Meeting Notes",
      "content": "Discussed project timeline and deliverables",
      "created_at": "2023-05-06T12:34:56.789Z",
      "updated_at": "2023-05-06T12:34:56.789Z",
      "status": "active"
    },
    {
      "id": "223e4567-e89b-12d3-a456-426614174001",
      "user_id": "98765432-abcd-efgh-ijkl-123456789012",
      "title": "Shopping List",
      "content": "Milk, eggs, bread",
      "created_at": "2023-05-05T10:20:30.456Z",
      "updated_at": "2023-05-05T10:20:30.456Z",
      "status": "active"
    }
  ],
  "pagination": {
    "total": 2,
    "limit": 10,
    "offset": 0
  }
}
```

## Security Features

This implementation includes:

1. Row Level Security (RLS) policies to ensure users can only access their own notes
2. JWT validation in edge functions
3. Environment variables for secure credential management
4. Input validation to prevent injection attacks
5. Proper HTTP status codes and error handling 