# Agentic Arena - Database Schema

## Overview
The database schema is designed for PostgreSQL with JSON support for flexible metadata storage. The schema prioritizes data integrity, query performance, and scalability.

## Entity Relationship Diagram
```
Users ──┐
        │
        ├─ Workflows ──┐
        │              │
        ├─ Comments    ├─ Submissions ──┐
        │              │                │
        ├─ Votes       │                ├─ Evaluations
        │              │                │
        └─ Follows     └─ Tasks ────────┘
```

## Core Tables

### users
Stores user account information and profile data.

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(50) UNIQUE NOT NULL,
    full_name VARCHAR(255),
    bio TEXT,
    avatar_url TEXT,
    email_verified BOOLEAN DEFAULT FALSE,
    password_hash VARCHAR(255),
    oauth_provider VARCHAR(50),
    oauth_id VARCHAR(255),
    is_active BOOLEAN DEFAULT TRUE,
    is_admin BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    CONSTRAINT username_format CHECK (username ~ '^[a-zA-Z0-9_-]{3,50}$'),
    CONSTRAINT email_format CHECK (email ~ '^[^@\s]+@[^@\s]+\.[^@\s]+$')
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_created_at ON users(created_at);
```

### benchmark_tasks
Defines standardized tasks for workflow evaluation.

```sql
CREATE TABLE benchmark_tasks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT NOT NULL,
    category VARCHAR(100) NOT NULL,
    difficulty VARCHAR(20) CHECK (difficulty IN ('beginner', 'intermediate', 'advanced')),
    success_criteria JSONB NOT NULL,
    test_cases JSONB,
    max_tokens INTEGER,
    max_time_minutes INTEGER,
    is_active BOOLEAN DEFAULT TRUE,
    created_by UUID REFERENCES users(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_tasks_category ON benchmark_tasks(category);
CREATE INDEX idx_tasks_difficulty ON benchmark_tasks(difficulty);
CREATE INDEX idx_tasks_active ON benchmark_tasks(is_active);
```

### workflows
Primary workflow submission records.

```sql
CREATE TABLE workflows (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    task_id UUID NOT NULL REFERENCES benchmark_tasks(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    tool_type VARCHAR(50) NOT NULL,
    tool_version VARCHAR(50),
    status VARCHAR(20) DEFAULT 'submitted' CHECK (
        status IN ('submitted', 'processing', 'evaluated', 'failed', 'rejected')
    ),
    is_public BOOLEAN DEFAULT TRUE,
    file_path TEXT NOT NULL,
    file_size BIGINT NOT NULL,
    file_hash VARCHAR(64),
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_workflows_user_id ON workflows(user_id);
CREATE INDEX idx_workflows_task_id ON workflows(task_id);
CREATE INDEX idx_workflows_tool_type ON workflows(tool_type);
CREATE INDEX idx_workflows_status ON workflows(status);
CREATE INDEX idx_workflows_public ON workflows(is_public);
CREATE INDEX idx_workflows_created_at ON workflows(created_at);
CREATE UNIQUE INDEX idx_workflows_file_hash ON workflows(file_hash) WHERE file_hash IS NOT NULL;
```

### submissions
Detailed submission data parsed from workflow files.

```sql
CREATE TABLE submissions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    workflow_id UUID NOT NULL REFERENCES workflows(id) ON DELETE CASCADE,
    raw_data JSONB NOT NULL,
    parsed_steps JSONB NOT NULL,
    total_tokens INTEGER NOT NULL DEFAULT 0,
    input_tokens INTEGER NOT NULL DEFAULT 0,
    output_tokens INTEGER NOT NULL DEFAULT 0,
    tool_calls INTEGER NOT NULL DEFAULT 0,
    execution_time INTEGER, -- in seconds
    iterations INTEGER DEFAULT 1,
    model_name VARCHAR(100),
    estimated_cost DECIMAL(10, 6) DEFAULT 0.00,
    validation_status VARCHAR(20) DEFAULT 'pending' CHECK (
        validation_status IN ('pending', 'valid', 'invalid', 'suspicious')
    ),
    validation_errors JSONB DEFAULT '[]',
    trust_score DECIMAL(3, 2) DEFAULT 1.00 CHECK (trust_score >= 0 AND trust_score <= 1),
    processed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_submissions_workflow_id ON submissions(workflow_id);
CREATE INDEX idx_submissions_total_tokens ON submissions(total_tokens);
CREATE INDEX idx_submissions_tool_calls ON submissions(tool_calls);
CREATE INDEX idx_submissions_execution_time ON submissions(execution_time);
CREATE INDEX idx_submissions_validation_status ON submissions(validation_status);
CREATE INDEX idx_submissions_trust_score ON submissions(trust_score);
```

### evaluations
Evaluation results and scoring for submissions.

```sql
CREATE TABLE evaluations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    submission_id UUID NOT NULL REFERENCES submissions(id) ON DELETE CASCADE,
    task_id UUID NOT NULL REFERENCES benchmark_tasks(id),
    overall_score DECIMAL(5, 2) NOT NULL DEFAULT 0.00,
    efficiency_score DECIMAL(5, 2) NOT NULL DEFAULT 0.00,
    speed_score DECIMAL(5, 2) NOT NULL DEFAULT 0.00,
    cost_score DECIMAL(5, 2) NOT NULL DEFAULT 0.00,
    correctness_score DECIMAL(5, 2) NOT NULL DEFAULT 0.00,
    overall_ranking INTEGER,
    efficiency_ranking INTEGER,
    speed_ranking INTEGER,
    cost_ranking INTEGER,
    percentiles JSONB DEFAULT '{}',
    evaluation_details JSONB DEFAULT '{}',
    evaluator_version VARCHAR(20) DEFAULT '1.0',
    evaluated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    CONSTRAINT valid_scores CHECK (
        overall_score >= 0 AND overall_score <= 100 AND
        efficiency_score >= 0 AND efficiency_score <= 100 AND
        speed_score >= 0 AND speed_score <= 100 AND
        cost_score >= 0 AND cost_score <= 100 AND
        correctness_score >= 0 AND correctness_score <= 100
    )
);

CREATE INDEX idx_evaluations_submission_id ON evaluations(submission_id);
CREATE INDEX idx_evaluations_task_id ON evaluations(task_id);
CREATE INDEX idx_evaluations_overall_score ON evaluations(overall_score DESC);
CREATE INDEX idx_evaluations_efficiency_score ON evaluations(efficiency_score DESC);
CREATE INDEX idx_evaluations_speed_score ON evaluations(speed_score DESC);
CREATE INDEX idx_evaluations_cost_score ON evaluations(cost_score DESC);
CREATE INDEX idx_evaluations_overall_ranking ON evaluations(overall_ranking);
```

## Social Features Tables

### votes
User votes on workflows (upvote/downvote).

```sql
CREATE TABLE votes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    workflow_id UUID NOT NULL REFERENCES workflows(id) ON DELETE CASCADE,
    vote_type VARCHAR(10) NOT NULL CHECK (vote_type IN ('upvote', 'downvote')),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    UNIQUE(user_id, workflow_id)
);

CREATE INDEX idx_votes_workflow_id ON votes(workflow_id);
CREATE INDEX idx_votes_user_id ON votes(user_id);
CREATE INDEX idx_votes_type ON votes(vote_type);
```

### comments
Comments on workflows with threading support.

```sql
CREATE TABLE comments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    workflow_id UUID NOT NULL REFERENCES workflows(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    parent_id UUID REFERENCES comments(id) ON DELETE CASCADE,
    content TEXT NOT NULL,
    is_edited BOOLEAN DEFAULT FALSE,
    is_deleted BOOLEAN DEFAULT FALSE,
    upvotes INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    CONSTRAINT content_length CHECK (char_length(content) >= 1 AND char_length(content) <= 2000)
);

CREATE INDEX idx_comments_workflow_id ON comments(workflow_id);
CREATE INDEX idx_comments_user_id ON comments(user_id);
CREATE INDEX idx_comments_parent_id ON comments(parent_id);
CREATE INDEX idx_comments_created_at ON comments(created_at);
```

### comment_votes
Votes on individual comments.

```sql
CREATE TABLE comment_votes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    comment_id UUID NOT NULL REFERENCES comments(id) ON DELETE CASCADE,
    vote_type VARCHAR(10) NOT NULL CHECK (vote_type IN ('upvote', 'downvote')),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    UNIQUE(user_id, comment_id)
);

CREATE INDEX idx_comment_votes_comment_id ON comment_votes(comment_id);
```

### follows
User following relationships.

```sql
CREATE TABLE follows (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    follower_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    following_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    UNIQUE(follower_id, following_id),
    CONSTRAINT no_self_follow CHECK (follower_id != following_id)
);

CREATE INDEX idx_follows_follower ON follows(follower_id);
CREATE INDEX idx_follows_following ON follows(following_id);
```

## System Tables

### user_sessions
Active user sessions for JWT token management.

```sql
CREATE TABLE user_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    session_token VARCHAR(255) NOT NULL UNIQUE,
    refresh_token VARCHAR(255) NOT NULL UNIQUE,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_activity TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    ip_address INET,
    user_agent TEXT
);

CREATE INDEX idx_sessions_user_id ON user_sessions(user_id);
CREATE INDEX idx_sessions_expires_at ON user_sessions(expires_at);
CREATE INDEX idx_sessions_last_activity ON user_sessions(last_activity);
```

### audit_logs
System audit trail for important actions.

```sql
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50),
    resource_id UUID,
    ip_address INET,
    user_agent TEXT,
    details JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_audit_logs_user_id ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_action ON audit_logs(action);
CREATE INDEX idx_audit_logs_resource ON audit_logs(resource_type, resource_id);
CREATE INDEX idx_audit_logs_created_at ON audit_logs(created_at);
```

### notifications
User notifications system.

```sql
CREATE TABLE notifications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    type VARCHAR(50) NOT NULL,
    title VARCHAR(255) NOT NULL,
    message TEXT NOT NULL,
    data JSONB DEFAULT '{}',
    is_read BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_notifications_user_id ON notifications(user_id);
CREATE INDEX idx_notifications_unread ON notifications(user_id, is_read) WHERE is_read = FALSE;
CREATE INDEX idx_notifications_created_at ON notifications(created_at);
```

## Statistics and Caching Tables

### leaderboard_cache
Cached leaderboard results for performance.

```sql
CREATE TABLE leaderboard_cache (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    cache_key VARCHAR(255) UNIQUE NOT NULL,
    task_id UUID REFERENCES benchmark_tasks(id),
    tool_type VARCHAR(50),
    metric_type VARCHAR(50),
    period VARCHAR(20),
    data JSONB NOT NULL,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_leaderboard_cache_key ON leaderboard_cache(cache_key);
CREATE INDEX idx_leaderboard_cache_expires ON leaderboard_cache(expires_at);
```

### user_stats
Precomputed user statistics for quick access.

```sql
CREATE TABLE user_stats (
    user_id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
    total_submissions INTEGER DEFAULT 0,
    public_submissions INTEGER DEFAULT 0,
    best_overall_ranking INTEGER,
    best_efficiency_ranking INTEGER,
    total_tokens_used BIGINT DEFAULT 0,
    total_tool_calls INTEGER DEFAULT 0,
    average_score DECIMAL(5, 2) DEFAULT 0.00,
    total_upvotes INTEGER DEFAULT 0,
    follower_count INTEGER DEFAULT 0,
    following_count INTEGER DEFAULT 0,
    last_updated TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_user_stats_best_ranking ON user_stats(best_overall_ranking);
CREATE INDEX idx_user_stats_total_submissions ON user_stats(total_submissions);
CREATE INDEX idx_user_stats_average_score ON user_stats(average_score DESC);
```

## Views

### workflow_details_view
Comprehensive workflow information with related data.

```sql
CREATE VIEW workflow_details_view AS
SELECT 
    w.id,
    w.title,
    w.description,
    w.tool_type,
    w.status,
    w.is_public,
    w.created_at,
    u.username,
    u.full_name,
    u.avatar_url,
    t.name as task_name,
    t.category as task_category,
    s.total_tokens,
    s.tool_calls,
    s.execution_time,
    s.estimated_cost,
    e.overall_score,
    e.overall_ranking,
    COALESCE(vote_stats.upvotes, 0) as upvotes,
    COALESCE(vote_stats.downvotes, 0) as downvotes,
    COALESCE(comment_stats.comment_count, 0) as comment_count
FROM workflows w
JOIN users u ON w.user_id = u.id
JOIN benchmark_tasks t ON w.task_id = t.id
LEFT JOIN submissions s ON w.id = s.workflow_id
LEFT JOIN evaluations e ON s.id = e.submission_id
LEFT JOIN (
    SELECT 
        workflow_id,
        SUM(CASE WHEN vote_type = 'upvote' THEN 1 ELSE 0 END) as upvotes,
        SUM(CASE WHEN vote_type = 'downvote' THEN 1 ELSE 0 END) as downvotes
    FROM votes
    GROUP BY workflow_id
) vote_stats ON w.id = vote_stats.workflow_id
LEFT JOIN (
    SELECT 
        workflow_id,
        COUNT(*) as comment_count
    FROM comments
    WHERE is_deleted = FALSE
    GROUP BY workflow_id
) comment_stats ON w.id = comment_stats.workflow_id;
```

### leaderboard_view
Optimized view for leaderboard queries.

```sql
CREATE VIEW leaderboard_view AS
SELECT 
    ROW_NUMBER() OVER (PARTITION BY e.task_id ORDER BY e.overall_score DESC) as rank,
    w.id as workflow_id,
    w.title,
    w.tool_type,
    u.username,
    u.avatar_url,
    t.name as task_name,
    s.total_tokens,
    s.tool_calls,
    s.execution_time,
    s.estimated_cost,
    e.overall_score,
    e.efficiency_score,
    e.speed_score,
    e.cost_score,
    w.created_at
FROM evaluations e
JOIN submissions s ON e.submission_id = s.id
JOIN workflows w ON s.workflow_id = w.id
JOIN users u ON w.user_id = u.id
JOIN benchmark_tasks t ON e.task_id = t.id
WHERE w.is_public = TRUE 
AND w.status = 'evaluated'
AND s.validation_status = 'valid';
```

## Functions and Triggers

### Update user stats trigger
```sql
CREATE OR REPLACE FUNCTION update_user_stats()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' OR TG_OP = 'UPDATE' THEN
        INSERT INTO user_stats (user_id) VALUES (NEW.user_id)
        ON CONFLICT (user_id) DO NOTHING;
    END IF;
    
    -- Recalculate stats for the affected user
    WITH stats AS (
        SELECT 
            w.user_id,
            COUNT(*) as total_submissions,
            COUNT(*) FILTER (WHERE w.is_public = TRUE) as public_submissions,
            MIN(e.overall_ranking) as best_overall_ranking,
            MIN(e.efficiency_ranking) as best_efficiency_ranking,
            COALESCE(SUM(s.total_tokens), 0) as total_tokens_used,
            COALESCE(SUM(s.tool_calls), 0) as total_tool_calls,
            COALESCE(AVG(e.overall_score), 0) as average_score
        FROM workflows w
        LEFT JOIN submissions s ON w.id = s.workflow_id
        LEFT JOIN evaluations e ON s.id = e.submission_id
        WHERE w.user_id = COALESCE(NEW.user_id, OLD.user_id)
        GROUP BY w.user_id
    )
    UPDATE user_stats 
    SET 
        total_submissions = stats.total_submissions,
        public_submissions = stats.public_submissions,
        best_overall_ranking = stats.best_overall_ranking,
        best_efficiency_ranking = stats.best_efficiency_ranking,
        total_tokens_used = stats.total_tokens_used,
        total_tool_calls = stats.total_tool_calls,
        average_score = stats.average_score,
        last_updated = NOW()
    FROM stats
    WHERE user_stats.user_id = stats.user_id;
    
    RETURN COALESCE(NEW, OLD);
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_update_user_stats
    AFTER INSERT OR UPDATE OR DELETE ON workflows
    FOR EACH ROW EXECUTE FUNCTION update_user_stats();
```

### Update timestamps trigger
```sql
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_users_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER trigger_workflows_updated_at
    BEFORE UPDATE ON workflows
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER trigger_comments_updated_at
    BEFORE UPDATE ON comments
    FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

## Database Configuration

### Extensions
```sql
-- Enable UUID generation
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Enable full-text search
CREATE EXTENSION IF NOT EXISTS "pg_trgm";

-- Enable JSON operators
CREATE EXTENSION IF NOT EXISTS "btree_gin";
```

### Performance Optimization
```sql
-- Partial indexes for common query patterns
CREATE INDEX idx_workflows_public_recent 
ON workflows(created_at DESC) 
WHERE is_public = TRUE AND status = 'evaluated';

CREATE INDEX idx_submissions_valid_tokens 
ON submissions(total_tokens) 
WHERE validation_status = 'valid';

-- Full-text search index
CREATE INDEX idx_workflows_search 
ON workflows 
USING GIN(to_tsvector('english', title || ' ' || COALESCE(description, '')));

-- Composite index for leaderboard queries
CREATE INDEX idx_evaluations_leaderboard 
ON evaluations(task_id, overall_score DESC, evaluated_at DESC);
```

### Data Retention Policies
```sql
-- Clean up expired sessions
DELETE FROM user_sessions WHERE expires_at < NOW() - INTERVAL '7 days';

-- Archive old audit logs (older than 1 year)
DELETE FROM audit_logs WHERE created_at < NOW() - INTERVAL '1 year';

-- Clean up failed workflows (older than 30 days)
DELETE FROM workflows 
WHERE status = 'failed' 
AND created_at < NOW() - INTERVAL '30 days';
```

## Backup and Recovery

### Backup Strategy
- Full database backup daily at 2 AM UTC
- Incremental backups every 6 hours
- Point-in-time recovery enabled with WAL archiving
- Cross-region backup replication for disaster recovery

### Critical Tables Priority
1. **High Priority**: users, workflows, submissions, evaluations
2. **Medium Priority**: comments, votes, benchmark_tasks
3. **Low Priority**: audit_logs, notifications, cache tables

## Security Considerations

### Row Level Security (RLS)
```sql
-- Enable RLS on sensitive tables
ALTER TABLE workflows ENABLE ROW LEVEL SECURITY;
ALTER TABLE submissions ENABLE ROW LEVEL SECURITY;
ALTER TABLE comments ENABLE ROW LEVEL SECURITY;

-- Policy examples
CREATE POLICY workflows_owner_policy ON workflows
    FOR ALL TO authenticated
    USING (user_id = auth.uid() OR is_public = TRUE);

CREATE POLICY comments_visibility_policy ON comments
    FOR SELECT TO authenticated
    USING (
        workflow_id IN (
            SELECT id FROM workflows 
            WHERE is_public = TRUE OR user_id = auth.uid()
        )
    );
```

### Data Encryption
- Sensitive fields encrypted at application level
- Full database encryption at rest
- SSL/TLS for all connections
- Regular security audits and vulnerability assessments