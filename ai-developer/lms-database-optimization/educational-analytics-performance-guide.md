# Educational Analytics & Database Performance Guide

## Table of Contents
1. [Educational Database Architecture](#educational-database-architecture)
2. [Learning Analytics Optimization](#learning-analytics-optimization)
3. [Educational Workload Patterns](#educational-workload-patterns)
4. [Progress Tracking Performance](#progress-tracking-performance)
5. [Course Content Optimization](#course-content-optimization)
6. [Student Data Privacy & Performance](#student-data-privacy--performance)
7. [Real-time Learning Analytics](#real-time-learning-analytics)
8. [Scalable Educational Infrastructure](#scalable-educational-infrastructure)

## Educational Database Architecture

### 1. Learning-Optimized Schema Design

**✅ Good: Educational Database Structure**
```sql
-- Educational database schema optimized for learning analytics and performance
-- Designed with educational workload patterns and student privacy in mind

-- Core educational entities with performance optimization
CREATE TABLE institutions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    domain VARCHAR(100) UNIQUE NOT NULL,
    type educational_institution_type NOT NULL,
    settings JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Students table optimized for educational analytics
CREATE TABLE students (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    institution_id UUID NOT NULL REFERENCES institutions(id),
    student_identifier VARCHAR(50) NOT NULL, -- Privacy-compliant identifier
    email VARCHAR(255) UNIQUE,
    profile JSONB DEFAULT '{}', -- Educational preferences, accessibility needs
    learning_analytics_consent BOOLEAN DEFAULT FALSE,
    data_retention_preferences JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    last_active TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    -- Performance indexes for educational queries
    CONSTRAINT unique_student_per_institution UNIQUE (institution_id, student_identifier)
);

-- Courses optimized for educational content delivery
CREATE TABLE courses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    institution_id UUID NOT NULL REFERENCES institutions(id),
    title VARCHAR(500) NOT NULL,
    description TEXT,
    difficulty_level educational_difficulty NOT NULL,
    estimated_duration_minutes INTEGER,
    learning_objectives TEXT[],
    prerequisites UUID[] DEFAULT '{}', -- Array of prerequisite course IDs
    content_metadata JSONB DEFAULT '{}',
    ai_generated BOOLEAN DEFAULT FALSE,
    generation_metadata JSONB DEFAULT '{}',
    published_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Lessons with educational content optimization
CREATE TABLE lessons (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    course_id UUID NOT NULL REFERENCES courses(id) ON DELETE CASCADE,
    title VARCHAR(500) NOT NULL,
    description TEXT,
    lesson_order INTEGER NOT NULL,
    estimated_duration_minutes INTEGER,
    learning_objectives TEXT[],
    content_type lesson_content_type NOT NULL,
    content_data JSONB NOT NULL,
    accessibility_features JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    CONSTRAINT unique_lesson_order_per_course UNIQUE (course_id, lesson_order)
);

-- Sections optimized for granular progress tracking
CREATE TABLE sections (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    lesson_id UUID NOT NULL REFERENCES lessons(id) ON DELETE CASCADE,
    title VARCHAR(500) NOT NULL,
    section_order INTEGER NOT NULL,
    section_type section_content_type NOT NULL,
    content TEXT NOT NULL,
    estimated_read_time_seconds INTEGER,
    required_for_completion BOOLEAN DEFAULT TRUE,
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    CONSTRAINT unique_section_order_per_lesson UNIQUE (lesson_id, section_order)
);

-- Optimized enrollments table for educational analytics
CREATE TABLE enrollments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    student_id UUID NOT NULL REFERENCES students(id),
    course_id UUID NOT NULL REFERENCES courses(id),
    enrollment_date TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    completion_date TIMESTAMP WITH TIME ZONE,
    progress_percentage DECIMAL(5,2) DEFAULT 0.00,
    time_spent_minutes INTEGER DEFAULT 0,
    last_accessed TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    enrollment_status enrollment_status_type DEFAULT 'active',
    
    CONSTRAINT unique_student_course_enrollment UNIQUE (student_id, course_id)
);

-- High-performance learning progress tracking
CREATE TABLE learning_progress (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    student_id UUID NOT NULL REFERENCES students(id),
    course_id UUID NOT NULL REFERENCES courses(id),
    lesson_id UUID REFERENCES lessons(id),
    section_id UUID REFERENCES sections(id),
    
    -- Progress metrics optimized for analytics
    started_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    completed_at TIMESTAMP WITH TIME ZONE,
    time_spent_seconds INTEGER DEFAULT 0,
    completion_percentage DECIMAL(5,2) DEFAULT 0.00,
    attempts_count INTEGER DEFAULT 1,
    
    -- Learning analytics data
    interaction_count INTEGER DEFAULT 0,
    engagement_score DECIMAL(3,2), -- 0.00 to 1.00
    learning_velocity DECIMAL(10,4), -- Progress per minute
    difficulty_rating INTEGER CHECK (difficulty_rating BETWEEN 1 AND 5),
    
    -- Performance optimization
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    CONSTRAINT unique_progress_per_content UNIQUE (student_id, course_id, lesson_id, section_id)
);

-- Educational interaction tracking for analytics
CREATE TABLE learning_interactions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    student_id UUID NOT NULL REFERENCES students(id),
    session_id UUID NOT NULL,
    course_id UUID REFERENCES courses(id),
    lesson_id UUID REFERENCES lessons(id),
    section_id UUID REFERENCES sections(id),
    
    -- Interaction details
    interaction_type interaction_type NOT NULL,
    interaction_data JSONB NOT NULL,
    timestamp TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    duration_seconds INTEGER,
    
    -- Educational context
    learning_context JSONB DEFAULT '{}',
    device_info JSONB DEFAULT '{}',
    
    -- Performance optimization for analytics queries
    date_partition DATE GENERATED ALWAYS AS (DATE(timestamp)) STORED
) PARTITION BY RANGE (date_partition);

-- Optimized assessment results for educational analytics
CREATE TABLE assessment_results (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    student_id UUID NOT NULL REFERENCES students(id),
    assessment_id UUID NOT NULL,
    course_id UUID NOT NULL REFERENCES courses(id),
    lesson_id UUID REFERENCES lessons(id),
    
    -- Assessment performance metrics
    score DECIMAL(5,2) NOT NULL,
    max_score DECIMAL(5,2) NOT NULL,
    percentage DECIMAL(5,2) GENERATED ALWAYS AS (
        CASE 
            WHEN max_score > 0 THEN (score / max_score) * 100 
            ELSE 0 
        END
    ) STORED,
    
    -- Educational analytics
    time_taken_seconds INTEGER NOT NULL,
    attempts_count INTEGER DEFAULT 1,
    question_responses JSONB NOT NULL,
    detailed_analytics JSONB DEFAULT '{}',
    
    -- Performance optimization
    completed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    CONSTRAINT valid_score_range CHECK (score >= 0 AND score <= max_score)
);

-- Performance indexes for educational queries
CREATE INDEX CONCURRENTLY idx_students_institution_active 
ON students(institution_id, last_active DESC) 
WHERE student_identifier IS NOT NULL;

CREATE INDEX CONCURRENTLY idx_courses_published_difficulty 
ON courses(institution_id, difficulty_level, published_at DESC) 
WHERE published_at IS NOT NULL;

CREATE INDEX CONCURRENTLY idx_enrollments_student_status 
ON enrollments(student_id, enrollment_status, last_accessed DESC);

CREATE INDEX CONCURRENTLY idx_learning_progress_analytics 
ON learning_progress(student_id, course_id, completed_at DESC, engagement_score DESC);

-- Educational analytics optimized indexes
CREATE INDEX CONCURRENTLY idx_interactions_analytics_time 
ON learning_interactions(student_id, course_id, timestamp DESC, interaction_type);

CREATE INDEX CONCURRENTLY idx_interactions_session_analysis 
ON learning_interactions(session_id, timestamp) 
INCLUDE (interaction_type, duration_seconds);

-- Assessment performance indexes
CREATE INDEX CONCURRENTLY idx_assessments_performance 
ON assessment_results(course_id, percentage DESC, completed_at DESC);

CREATE INDEX CONCURRENTLY idx_assessments_student_progress 
ON assessment_results(student_id, course_id, completed_at DESC);
```

### 2. Educational Analytics Data Types

**✅ Good: Learning-Focused Custom Types**
```sql
-- Educational domain-specific data types for performance and clarity

CREATE TYPE educational_institution_type AS ENUM (
    'university',
    'college', 
    'k12_school',
    'training_center',
    'corporate',
    'online_platform'
);

CREATE TYPE educational_difficulty AS ENUM (
    'beginner',
    'intermediate', 
    'advanced',
    'expert'
);

CREATE TYPE lesson_content_type AS ENUM (
    'video',
    'text',
    'interactive',
    'assessment',
    'project',
    'discussion'
);

CREATE TYPE section_content_type AS ENUM (
    'text_content',
    'multimedia',
    'interactive_exercise',
    'quiz_question',
    'code_example',
    'case_study'
);

CREATE TYPE enrollment_status_type AS ENUM (
    'active',
    'completed',
    'paused',
    'dropped',
    'suspended'
);

CREATE TYPE interaction_type AS ENUM (
    'content_view',
    'section_start',
    'section_complete',
    'quiz_attempt',
    'video_watch',
    'document_download',
    'discussion_post',
    'help_request',
    'bookmark_add',
    'note_create'
);

-- Educational performance aggregation type
CREATE TYPE learning_performance_summary AS (
    total_time_spent INTEGER,
    completion_rate DECIMAL(5,2),
    average_score DECIMAL(5,2),
    engagement_level TEXT,
    learning_velocity DECIMAL(10,4),
    achievement_count INTEGER
);
```

## Learning Analytics Optimization

### 1. High-Performance Educational Queries

**✅ Good: Optimized Learning Analytics Functions**
```sql
-- Educational analytics functions optimized for performance and learning insights

-- Real-time course progress calculation with performance optimization
CREATE OR REPLACE FUNCTION calculate_course_progress(
    p_student_id UUID,
    p_course_id UUID
) RETURNS learning_performance_summary AS $$
DECLARE
    result learning_performance_summary;
    total_sections INTEGER;
    completed_sections INTEGER;
    total_assessments INTEGER;
    completed_assessments INTEGER;
BEGIN
    -- Optimized query using materialized aggregations
    WITH course_structure AS (
        SELECT 
            COUNT(s.id) as section_count,
            COUNT(CASE WHEN s.required_for_completion THEN s.id END) as required_sections
        FROM courses c
        JOIN lessons l ON c.id = l.course_id
        JOIN sections s ON l.id = s.lesson_id
        WHERE c.id = p_course_id
    ),
    student_progress AS (
        SELECT 
            COUNT(CASE WHEN lp.completed_at IS NOT NULL THEN 1 END) as completed_count,
            SUM(lp.time_spent_seconds) as total_time,
            AVG(lp.engagement_score) as avg_engagement,
            AVG(lp.learning_velocity) as avg_velocity
        FROM learning_progress lp
        JOIN sections s ON lp.section_id = s.id
        WHERE lp.student_id = p_student_id 
        AND lp.course_id = p_course_id
        AND s.required_for_completion = TRUE
    ),
    assessment_performance AS (
        SELECT 
            AVG(percentage) as avg_score,
            COUNT(*) as assessment_count
        FROM assessment_results ar
        WHERE ar.student_id = p_student_id 
        AND ar.course_id = p_course_id
    )
    SELECT 
        COALESCE(sp.total_time, 0),
        CASE 
            WHEN cs.required_sections > 0 
            THEN (sp.completed_count::DECIMAL / cs.required_sections * 100)
            ELSE 0 
        END,
        COALESCE(ap.avg_score, 0),
        CASE 
            WHEN sp.avg_engagement >= 0.8 THEN 'high'
            WHEN sp.avg_engagement >= 0.6 THEN 'medium'
            ELSE 'low'
        END,
        COALESCE(sp.avg_velocity, 0),
        COALESCE(ap.assessment_count, 0)
    INTO result
    FROM course_structure cs
    CROSS JOIN student_progress sp
    CROSS JOIN assessment_performance ap;
    
    RETURN result;
END;
$$ LANGUAGE plpgsql STABLE;

-- Optimized learning analytics dashboard data
CREATE OR REPLACE FUNCTION get_learning_analytics_dashboard(
    p_student_id UUID,
    p_time_period INTERVAL DEFAULT '30 days'
) RETURNS TABLE (
    metric_name TEXT,
    metric_value DECIMAL,
    comparison_period DECIMAL,
    trend_direction TEXT,
    details JSONB
) AS $$
BEGIN
    RETURN QUERY
    WITH time_bounds AS (
        SELECT 
            NOW() - p_time_period as period_start,
            NOW() as period_end,
            NOW() - (p_time_period * 2) as comparison_start
    ),
    current_period AS (
        SELECT 
            COUNT(DISTINCT lp.course_id) as active_courses,
            SUM(lp.time_spent_seconds) as total_time,
            AVG(lp.engagement_score) as avg_engagement,
            COUNT(CASE WHEN lp.completed_at IS NOT NULL THEN 1 END) as completions,
            AVG(ar.percentage) as avg_assessment_score
        FROM learning_progress lp
        LEFT JOIN assessment_results ar ON lp.student_id = ar.student_id 
            AND lp.course_id = ar.course_id
        CROSS JOIN time_bounds tb
        WHERE lp.student_id = p_student_id
        AND lp.updated_at BETWEEN tb.period_start AND tb.period_end
    ),
    comparison_period AS (
        SELECT 
            COUNT(DISTINCT lp.course_id) as active_courses,
            SUM(lp.time_spent_seconds) as total_time,
            AVG(lp.engagement_score) as avg_engagement,
            COUNT(CASE WHEN lp.completed_at IS NOT NULL THEN 1 END) as completions,
            AVG(ar.percentage) as avg_assessment_score
        FROM learning_progress lp
        LEFT JOIN assessment_results ar ON lp.student_id = ar.student_id 
            AND lp.course_id = ar.course_id
        CROSS JOIN time_bounds tb
        WHERE lp.student_id = p_student_id
        AND lp.updated_at BETWEEN tb.comparison_start AND tb.period_start
    )
    SELECT 
        'active_courses'::TEXT,
        cp.active_courses::DECIMAL,
        comp.active_courses::DECIMAL,
        CASE 
            WHEN cp.active_courses > comp.active_courses THEN 'up'
            WHEN cp.active_courses < comp.active_courses THEN 'down'
            ELSE 'stable'
        END,
        jsonb_build_object(
            'current', cp.active_courses,
            'previous', comp.active_courses,
            'change_percent', 
            CASE WHEN comp.active_courses > 0 
                THEN ((cp.active_courses - comp.active_courses) / comp.active_courses * 100)
                ELSE 0 
            END
        )
    FROM current_period cp
    CROSS JOIN comparison_period comp
    
    UNION ALL
    
    SELECT 
        'learning_time'::TEXT,
        cp.total_time::DECIMAL,
        comp.total_time::DECIMAL,
        CASE 
            WHEN cp.total_time > comp.total_time THEN 'up'
            WHEN cp.total_time < comp.total_time THEN 'down'
            ELSE 'stable'
        END,
        jsonb_build_object(
            'current_hours', ROUND(cp.total_time / 3600.0, 2),
            'previous_hours', ROUND(comp.total_time / 3600.0, 2),
            'daily_average', ROUND(cp.total_time / EXTRACT(DAYS FROM p_time_period) / 3600.0, 2)
        )
    FROM current_period cp
    CROSS JOIN comparison_period comp;
END;
$$ LANGUAGE plpgsql STABLE;

-- Educational content performance analytics
CREATE OR REPLACE FUNCTION analyze_content_performance(
    p_course_id UUID,
    p_analysis_period INTERVAL DEFAULT '90 days'
) RETURNS TABLE (
    content_id UUID,
    content_type TEXT,
    content_title TEXT,
    engagement_metrics JSONB,
    performance_metrics JSONB,
    improvement_suggestions JSONB
) AS $$
BEGIN
    RETURN QUERY
    WITH content_analytics AS (
        SELECT 
            s.id as section_id,
            s.title,
            s.section_type::TEXT,
            COUNT(DISTINCT lp.student_id) as unique_students,
            AVG(lp.time_spent_seconds) as avg_time_spent,
            AVG(lp.engagement_score) as avg_engagement,
            COUNT(CASE WHEN lp.completed_at IS NOT NULL THEN 1 END)::DECIMAL / 
                COUNT(*)::DECIMAL as completion_rate,
            AVG(lp.attempts_count) as avg_attempts,
            COUNT(CASE WHEN lp.difficulty_rating >= 4 THEN 1 END)::DECIMAL / 
                COUNT(*)::DECIMAL as difficulty_perception
        FROM sections s
        JOIN lessons l ON s.lesson_id = l.id
        LEFT JOIN learning_progress lp ON s.id = lp.section_id
        WHERE l.course_id = p_course_id
        AND lp.started_at >= NOW() - p_analysis_period
        GROUP BY s.id, s.title, s.section_type
    )
    SELECT 
        ca.section_id,
        ca.section_type,
        ca.title,
        jsonb_build_object(
            'unique_students', ca.unique_students,
            'avg_engagement_score', ROUND(ca.avg_engagement, 3),
            'completion_rate', ROUND(ca.completion_rate * 100, 2),
            'avg_time_minutes', ROUND(ca.avg_time_spent / 60.0, 2)
        ),
        jsonb_build_object(
            'difficulty_rating', ROUND(ca.difficulty_perception * 5, 2),
            'avg_attempts', ROUND(ca.avg_attempts, 2),
            'efficiency_score', 
                CASE 
                    WHEN ca.avg_time_spent > 0 
                    THEN ROUND(ca.completion_rate / (ca.avg_time_spent / 3600.0), 4)
                    ELSE 0 
                END
        ),
        jsonb_build_object(
            'recommendations', 
                CASE 
                    WHEN ca.completion_rate < 0.7 THEN 
                        ARRAY['Consider simplifying content', 'Add more examples', 'Improve instructions']
                    WHEN ca.avg_engagement < 0.6 THEN 
                        ARRAY['Add interactive elements', 'Break into smaller sections', 'Include multimedia']
                    WHEN ca.difficulty_perception > 0.8 THEN 
                        ARRAY['Provide additional support resources', 'Add prerequisite content', 'Include practice exercises']
                    ELSE 
                        ARRAY['Content performing well', 'Monitor for consistency']
                END,
            'priority_level',
                CASE 
                    WHEN ca.completion_rate < 0.5 OR ca.avg_engagement < 0.4 THEN 'high'
                    WHEN ca.completion_rate < 0.7 OR ca.avg_engagement < 0.6 THEN 'medium'
                    ELSE 'low'
                END
        )
    FROM content_analytics ca
    ORDER BY ca.avg_engagement ASC, ca.completion_rate ASC;
END;
$$ LANGUAGE plpgsql STABLE;
```

### 2. Educational Data Aggregation & Caching

**✅ Good: Performance-Optimized Materialized Views**
```sql
-- Materialized views for high-performance educational analytics

-- Course enrollment and completion statistics
CREATE MATERIALIZED VIEW course_performance_stats AS
SELECT 
    c.id as course_id,
    c.title as course_title,
    c.difficulty_level,
    COUNT(DISTINCT e.student_id) as total_enrollments,
    COUNT(DISTINCT CASE WHEN e.completion_date IS NOT NULL THEN e.student_id END) as completions,
    AVG(e.progress_percentage) as avg_progress,
    AVG(e.time_spent_minutes) as avg_time_spent,
    
    -- Educational effectiveness metrics
    AVG(CASE WHEN ar.percentage IS NOT NULL THEN ar.percentage END) as avg_assessment_score,
    COUNT(DISTINCT ar.id) as total_assessments,
    
    -- Engagement analytics
    AVG(lp.engagement_score) as avg_engagement,
    COUNT(DISTINCT li.student_id) as active_learners_30d,
    
    -- Performance indicators
    CASE 
        WHEN COUNT(DISTINCT e.student_id) > 0 
        THEN COUNT(DISTINCT CASE WHEN e.completion_date IS NOT NULL THEN e.student_id END)::DECIMAL / 
             COUNT(DISTINCT e.student_id)::DECIMAL 
        ELSE 0 
    END as completion_rate,
    
    -- Data freshness
    NOW() as last_updated
FROM courses c
LEFT JOIN enrollments e ON c.id = e.course_id
LEFT JOIN assessment_results ar ON c.id = ar.course_id
LEFT JOIN learning_progress lp ON c.id = lp.course_id
LEFT JOIN learning_interactions li ON c.id = li.course_id 
    AND li.timestamp >= NOW() - INTERVAL '30 days'
GROUP BY c.id, c.title, c.difficulty_level;

-- Indexes for materialized view performance
CREATE UNIQUE INDEX idx_course_performance_stats_course_id 
ON course_performance_stats(course_id);

CREATE INDEX idx_course_performance_stats_metrics 
ON course_performance_stats(completion_rate DESC, avg_engagement DESC, avg_assessment_score DESC);

-- Student learning analytics summary
CREATE MATERIALIZED VIEW student_learning_analytics AS
SELECT 
    s.id as student_id,
    s.institution_id,
    
    -- Learning activity metrics
    COUNT(DISTINCT e.course_id) as enrolled_courses,
    COUNT(DISTINCT CASE WHEN e.completion_date IS NOT NULL THEN e.course_id END) as completed_courses,
    SUM(e.time_spent_minutes) as total_learning_time,
    AVG(e.progress_percentage) as avg_course_progress,
    
    -- Assessment performance
    AVG(ar.percentage) as avg_assessment_score,
    COUNT(ar.id) as total_assessments,
    
    -- Engagement patterns
    AVG(lp.engagement_score) as avg_engagement,
    COUNT(DISTINCT DATE(li.timestamp)) as active_learning_days,
    
    -- Learning velocity and efficiency
    CASE 
        WHEN SUM(e.time_spent_minutes) > 0 
        THEN COUNT(DISTINCT CASE WHEN e.completion_date IS NOT NULL THEN e.course_id END)::DECIMAL / 
             (SUM(e.time_spent_minutes) / 60.0)
        ELSE 0 
    END as learning_efficiency,
    
    -- Recent activity indicators
    MAX(e.last_accessed) as last_learning_activity,
    COUNT(CASE WHEN li.timestamp >= NOW() - INTERVAL '7 days' THEN 1 END) as recent_interactions,
    
    NOW() as analytics_updated
FROM students s
LEFT JOIN enrollments e ON s.id = e.student_id
LEFT JOIN assessment_results ar ON s.id = ar.student_id
LEFT JOIN learning_progress lp ON s.id = lp.student_id
LEFT JOIN learning_interactions li ON s.id = li.student_id
GROUP BY s.id, s.institution_id;

-- Performance indexes for student analytics
CREATE UNIQUE INDEX idx_student_analytics_student_id 
ON student_learning_analytics(student_id);

CREATE INDEX idx_student_analytics_institution_performance 
ON student_learning_analytics(institution_id, avg_assessment_score DESC, learning_efficiency DESC);

-- Educational content effectiveness tracking
CREATE MATERIALIZED VIEW content_effectiveness_metrics AS
SELECT 
    l.course_id,
    l.id as lesson_id,
    s.id as section_id,
    s.section_type,
    
    -- Content engagement metrics
    COUNT(DISTINCT lp.student_id) as unique_learners,
    AVG(lp.time_spent_seconds) as avg_time_spent,
    AVG(lp.engagement_score) as avg_engagement,
    
    -- Learning effectiveness
    COUNT(CASE WHEN lp.completed_at IS NOT NULL THEN 1 END)::DECIMAL / 
        COUNT(*)::DECIMAL as completion_rate,
    AVG(lp.attempts_count) as avg_attempts,
    AVG(lp.difficulty_rating) as avg_difficulty_rating,
    
    -- Performance indicators
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY lp.time_spent_seconds) as median_time_spent,
    STDDEV(lp.time_spent_seconds) as time_variance,
    
    -- Educational quality metrics
    COUNT(CASE WHEN lp.engagement_score >= 0.8 THEN 1 END)::DECIMAL / 
        COUNT(*)::DECIMAL as high_engagement_rate,
    
    NOW() as metrics_updated
FROM lessons l
JOIN sections s ON l.id = s.lesson_id
LEFT JOIN learning_progress lp ON s.id = lp.section_id
GROUP BY l.course_id, l.id, s.id, s.section_type;

-- Automated refresh procedures for educational analytics
CREATE OR REPLACE FUNCTION refresh_educational_analytics() RETURNS VOID AS $$
BEGIN
    -- Refresh materialized views in dependency order
    REFRESH MATERIALIZED VIEW CONCURRENTLY course_performance_stats;
    REFRESH MATERIALIZED VIEW CONCURRENTLY student_learning_analytics;
    REFRESH MATERIALIZED VIEW CONCURRENTLY content_effectiveness_metrics;
    
    -- Update statistics for query optimization
    ANALYZE course_performance_stats;
    ANALYZE student_learning_analytics;
    ANALYZE content_effectiveness_metrics;
END;
$$ LANGUAGE plpgsql;

-- Schedule automatic refresh (to be run via cron or pg_cron)
-- SELECT cron.schedule('refresh-educational-analytics', '0 */4 * * *', 'SELECT refresh_educational_analytics();');
```

This comprehensive database optimization guide provides educational analytics performance, learning data patterns, and educational workload optimization specifically designed for Learning Management Systems. The patterns prioritize educational effectiveness measurement, student privacy, and scalable learning analytics over generic database optimization approaches. 