# LMS Database Architecture and Design Patterns

## Core LMS Database Schema Design

### Educational Content Hierarchy
**Design database schema that reflects the educational content structure**

✅ **Good Example: Hierarchical Course Structure**
```prisma
// Proper educational content hierarchy
model Course {
  id            String    @id @default(cuid())
  title         String
  description   String?
  difficulty    Difficulty @default(BEGINNER)
  estimatedHours Int      @default(0)
  imageUrl      String?
  isPublished   Boolean   @default(false)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  // Relationships
  lessons       Lesson[]
  enrollments   Enrollment[]
  reviews       Review[]
  categories    CourseCategory[]
  
  // Metadata
  metadata      Json?     // Flexible storage for AI-generated data
  
  @@map("courses")
  @@index([isPublished, difficulty])
  @@index([createdAt])
}

model Lesson {
  id          String  @id @default(cuid())
  title       String
  description String?
  order       Int
  isRequired  Boolean @default(true)
  courseId    String
  
  // Relationships
  course      Course    @relation(fields: [courseId], references: [id], onDelete: Cascade)
  sections    Section[]
  
  @@map("lessons")
  @@unique([courseId, order])
  @@index([courseId])
}

model Section {
  id          String      @id @default(cuid())
  title       String
  content     String      // Rich text content
  type        SectionType @default(TEXT)
  order       Int
  duration    Int         @default(0) // in minutes
  isOptional  Boolean     @default(false)
  lessonId    String
  
  // Relationships
  lesson      Lesson @relation(fields: [lessonId], references: [id], onDelete: Cascade)
  
  @@map("sections")
  @@unique([lessonId, order])
  @@index([lessonId])
}

enum SectionType {
  TEXT
  VIDEO
  AUDIO
  QUIZ
  ASSIGNMENT
  INTERACTIVE
  CODE_EXERCISE
}

enum Difficulty {
  BEGINNER
  INTERMEDIATE
  ADVANCED
  EXPERT
}
```

❌ **Bad Example: Flat Structure Without Hierarchy**
```prisma
// Poor design - no clear hierarchy
model Content {
  id      String @id
  title   String
  text    String
  type    String // String instead of enum
  courseId String? // Optional relationship
  
  @@map("content")
}
```

### User Management and Roles
✅ **Good Example: Flexible User Role System**
```prisma
model User {
  id          String    @id @default(cuid())
  email       String    @unique
  username    String?   @unique
  firstName   String
  lastName    String
  avatar      String?
  isActive    Boolean   @default(true)
  lastLoginAt DateTime?
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
  
  // Relationships
  enrollments Enrollment[]
  progress    Progress[]
  reviews     Review[]
  roles       UserRole[]
  
  // Profile data
  profile     UserProfile?
  
  @@map("users")
  @@index([email])
  @@index([isActive])
}

model Role {
  id          String @id @default(cuid())
  name        String @unique
  description String?
  permissions Json   // Flexible permissions storage
  
  // Relationships
  users       UserRole[]
  
  @@map("roles")
}

model UserRole {
  id     String @id @default(cuid())
  userId String
  roleId String
  
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  role Role @relation(fields: [roleId], references: [id], onDelete: Cascade)
  
  @@unique([userId, roleId])
  @@map("user_roles")
}

model UserProfile {
  id           String  @id @default(cuid())
  userId       String  @unique
  bio          String?
  location     String?
  website      String?
  socialLinks  Json?   // LinkedIn, Twitter, etc.
  preferences  Json?   // Learning preferences, notifications
  
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@map("user_profiles")
}
```

### Progress Tracking and Analytics
✅ **Good Example: Comprehensive Progress Tracking**
```prisma
model Enrollment {
  id             String           @id @default(cuid())
  userId         String
  courseId       String
  status         EnrollmentStatus @default(ACTIVE)
  enrolledAt     DateTime         @default(now())
  completedAt    DateTime?
  certificateUrl String?
  finalGrade     Float?
  
  // Relationships
  user   User   @relation(fields: [userId], references: [id], onDelete: Cascade)
  course Course @relation(fields: [courseId], references: [id], onDelete: Cascade)
  
  @@unique([userId, courseId])
  @@map("enrollments")
  @@index([userId])
  @@index([courseId])
  @@index([status])
}

model Progress {
  id            String   @id @default(cuid())
  userId        String
  courseId      String
  lessonId      String
  sectionId     String
  completedAt   DateTime @default(now())
  timeSpent     Int      @default(0) // seconds
  attemptsCount Int      @default(1)
  score         Float?   // For quizzes/assignments
  
  // Relationships - no direct relations to avoid circular dependencies
  // Use courseId, lessonId, sectionId for queries
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@unique([userId, sectionId])
  @@map("progress")
  @@index([userId, courseId])
  @@index([courseId])
  @@index([completedAt])
}

model LearningSession {
  id        String   @id @default(cuid())
  userId    String
  courseId  String
  startedAt DateTime @default(now())
  endedAt   DateTime?
  duration  Int?     // calculated duration in seconds
  activity  Json?    // Detailed activity log
  
  @@map("learning_sessions")
  @@index([userId, courseId])
  @@index([startedAt])
}

enum EnrollmentStatus {
  ACTIVE
  COMPLETED
  PAUSED
  DROPPED
  EXPIRED
}
```

## Advanced Database Patterns

### Soft Deletes and Audit Trail
✅ **Good Example: Audit Trail Implementation**
```prisma
model Course {
  id        String    @id @default(cuid())
  title     String
  // ... other fields
  
  // Soft delete
  deletedAt DateTime?
  deletedBy String?
  
  // Audit trail
  createdBy String
  updatedBy String?
  version   Int      @default(1)
  
  // Relationships
  auditLogs AuditLog[]
  
  @@map("courses")
  @@index([deletedAt]) // For filtering active records
}

model AuditLog {
  id         String   @id @default(cuid())
  tableName  String
  recordId   String
  action     AuditAction
  oldValues  Json?
  newValues  Json?
  userId     String
  timestamp  DateTime @default(now())
  
  @@map("audit_logs")
  @@index([tableName, recordId])
  @@index([userId])
  @@index([timestamp])
}

enum AuditAction {
  CREATE
  UPDATE
  DELETE
  RESTORE
}
```

### Content Versioning
✅ **Good Example: Course Content Versioning**
```prisma
model CourseVersion {
  id          String   @id @default(cuid())
  courseId    String
  version     Int
  title       String
  description String?
  content     Json     // Snapshot of entire course structure
  publishedAt DateTime?
  createdAt   DateTime @default(now())
  createdBy   String
  
  // Relationships
  course Course @relation(fields: [courseId], references: [id])
  
  @@unique([courseId, version])
  @@map("course_versions")
  @@index([courseId])
  @@index([publishedAt])
}
```

## Performance Optimization Patterns

### Database Indexing Strategy
✅ **Good Example: Strategic Index Design**
```sql
-- Performance-critical indexes for LMS queries

-- User authentication (most frequent)
CREATE INDEX CONCURRENTLY idx_users_email_active 
ON users(email) WHERE is_active = true;

-- Course discovery
CREATE INDEX CONCURRENTLY idx_courses_published_difficulty 
ON courses(is_published, difficulty, created_at DESC) 
WHERE deleted_at IS NULL;

-- Progress tracking (very frequent)
CREATE INDEX CONCURRENTLY idx_progress_user_course 
ON progress(user_id, course_id, completed_at DESC);

-- Learning analytics
CREATE INDEX CONCURRENTLY idx_learning_sessions_user_time 
ON learning_sessions(user_id, started_at DESC) 
WHERE ended_at IS NOT NULL;

-- Content hierarchy
CREATE INDEX CONCURRENTLY idx_sections_lesson_order 
ON sections(lesson_id, "order");

-- Search functionality
CREATE INDEX CONCURRENTLY idx_courses_text_search 
ON courses USING gin(to_tsvector('english', title || ' ' || description));
```

### Query Optimization Patterns
✅ **Good Example: Optimized LMS Queries**
```javascript
// courseRepository.js - Optimized queries
class CourseRepository {
  // Get course with progress for specific user
  async getCourseWithUserProgress(courseId, userId) {
    return await this.prisma.course.findUnique({
      where: { 
        id: courseId,
        deletedAt: null,
        isPublished: true 
      },
      select: {
        id: true,
        title: true,
        description: true,
        difficulty: true,
        estimatedHours: true,
        lessons: {
          select: {
            id: true,
            title: true,
            order: true,
            sections: {
              select: {
                id: true,
                title: true,
                type: true,
                order: true,
                duration: true
              },
              orderBy: { order: 'asc' }
            }
          },
          orderBy: { order: 'asc' }
        }
      }
    });
  }

  // Optimized dashboard query - single query with aggregations
  async getUserDashboardData(userId) {
    const [activeEnrollments, progressStats, recentActivity] = await Promise.all([
      // Active courses
      this.prisma.enrollment.findMany({
        where: {
          userId,
          status: 'ACTIVE'
        },
        include: {
          course: {
            select: {
              id: true,
              title: true,
              imageUrl: true,
              estimatedHours: true
            }
          }
        },
        take: 5
      }),

      // Progress statistics
      this.prisma.progress.groupBy({
        by: ['courseId'],
        where: { userId },
        _count: {
          sectionId: true
        }
      }),

      // Recent learning activity
      this.prisma.learningSession.findMany({
        where: {
          userId,
          endedAt: { not: null }
        },
        orderBy: { endedAt: 'desc' },
        take: 10,
        select: {
          id: true,
          courseId: true,
          duration: true,
          endedAt: true
        }
      })
    ]);

    return { activeEnrollments, progressStats, recentActivity };
  }
}
```

❌ **Bad Example: N+1 Query Problem**
```javascript
// Poor query pattern - N+1 problem
async getCoursesWithProgress(userId) {
  const courses = await this.prisma.course.findMany();
  
  // This creates N+1 queries!
  for (const course of courses) {
    course.progress = await this.prisma.progress.findMany({
      where: { courseId: course.id, userId }
    });
    
    course.lessons = await this.prisma.lesson.findMany({
      where: { courseId: course.id }
    });
  }
  
  return courses;
}
```

## Data Integrity and Validation

### Database Constraints
✅ **Good Example: Comprehensive Constraints**
```prisma
model Progress {
  id            String   @id @default(cuid())
  userId        String
  courseId      String
  lessonId      String
  sectionId     String
  completedAt   DateTime @default(now())
  timeSpent     Int      @default(0)
  score         Float?
  
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  // Constraints
  @@unique([userId, sectionId]) // One progress record per user per section
  @@map("progress")
  @@index([userId, courseId])
  
  // Custom constraints in migration
  // CHECK (time_spent >= 0)
  // CHECK (score >= 0 AND score <= 100)
}

// Custom migration for additional constraints
/*
-- migration.sql
ALTER TABLE progress 
ADD CONSTRAINT check_positive_time_spent 
CHECK (time_spent >= 0);

ALTER TABLE progress 
ADD CONSTRAINT check_valid_score 
CHECK (score IS NULL OR (score >= 0 AND score <= 100));

ALTER TABLE enrollments 
ADD CONSTRAINT check_completion_date 
CHECK (completed_at IS NULL OR completed_at >= enrolled_at);
*/
```

### Data Validation Patterns
✅ **Good Example: Application-Level Validation**
```javascript
// validationService.js
class ValidationService {
  static validateProgressUpdate(progressData) {
    const schema = Joi.object({
      userId: Joi.string().required(),
      courseId: Joi.string().required(),
      lessonId: Joi.string().required(),
      sectionId: Joi.string().required(),
      timeSpent: Joi.number().integer().min(0).max(86400), // Max 24 hours
      score: Joi.number().min(0).max(100).allow(null)
    });

    return schema.validate(progressData);
  }

  static async validateEnrollmentEligibility(userId, courseId) {
    const [user, course, existingEnrollment] = await Promise.all([
      prisma.user.findUnique({ where: { id: userId } }),
      prisma.course.findUnique({ where: { id: courseId } }),
      prisma.enrollment.findUnique({ 
        where: { userId_courseId: { userId, courseId } } 
      })
    ]);

    const errors = [];

    if (!user || !user.isActive) {
      errors.push('User not found or inactive');
    }

    if (!course || !course.isPublished || course.deletedAt) {
      errors.push('Course not available for enrollment');
    }

    if (existingEnrollment && existingEnrollment.status === 'ACTIVE') {
      errors.push('User already enrolled in this course');
    }

    return {
      isValid: errors.length === 0,
      errors
    };
  }
}
```

## Advanced LMS Database Patterns

### Multi-Tenant Architecture
✅ **Good Example: Organization-Based Multi-Tenancy**
```prisma
model Organization {
  id       String @id @default(cuid())
  name     String
  domain   String @unique
  settings Json?  // Org-specific configurations
  isActive Boolean @default(true)
  
  // Relationships
  users    User[]
  courses  Course[]
  
  @@map("organizations")
}

model User {
  id             String  @id @default(cuid())
  email          String
  organizationId String
  
  organization Organization @relation(fields: [organizationId], references: [id])
  
  @@unique([email, organizationId])
  @@map("users")
}

model Course {
  id             String  @id @default(cuid())
  title          String
  organizationId String
  
  organization Organization @relation(fields: [organizationId], references: [id])
  
  @@map("courses")
  @@index([organizationId])
}
```

### Content Categorization and Tagging
✅ **Good Example: Flexible Tagging System**
```prisma
model Category {
  id          String @id @default(cuid())
  name        String
  slug        String @unique
  description String?
  parentId    String?
  color       String?
  icon        String?
  
  // Self-referential relationship for hierarchy
  parent   Category?  @relation("CategoryHierarchy", fields: [parentId], references: [id])
  children Category[] @relation("CategoryHierarchy")
  
  // Relationships
  courses CourseCategory[]
  
  @@map("categories")
}

model Tag {
  id     String @id @default(cuid())
  name   String @unique
  color  String?
  
  courses CourseTag[]
  
  @@map("tags")
}

model CourseCategory {
  courseId   String
  categoryId String
  
  course   Course   @relation(fields: [courseId], references: [id], onDelete: Cascade)
  category Category @relation(fields: [categoryId], references: [id], onDelete: Cascade)
  
  @@id([courseId, categoryId])
  @@map("course_categories")
}

model CourseTag {
  courseId String
  tagId    String
  
  course Course @relation(fields: [courseId], references: [id], onDelete: Cascade)
  tag    Tag    @relation(fields: [tagId], references: [id], onDelete: Cascade)
  
  @@id([courseId, tagId])
  @@map("course_tags")
}
```

## Database Migration and Maintenance

### Migration Best Practices
✅ **Good Example: Safe Migration Strategy**
```sql
-- migration_001_add_course_rating.sql
-- Add rating system to courses

-- Step 1: Add new columns with defaults
ALTER TABLE courses 
ADD COLUMN average_rating DECIMAL(3,2) DEFAULT 0.0,
ADD COLUMN rating_count INTEGER DEFAULT 0;

-- Step 2: Create supporting tables
CREATE TABLE course_ratings (
  id SERIAL PRIMARY KEY,
  course_id VARCHAR(25) NOT NULL,
  user_id VARCHAR(25) NOT NULL,
  rating INTEGER NOT NULL CHECK (rating >= 1 AND rating <= 5),
  review TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  
  FOREIGN KEY (course_id) REFERENCES courses(id) ON DELETE CASCADE,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  UNIQUE(course_id, user_id)
);

-- Step 3: Create indexes
CREATE INDEX idx_course_ratings_course_id ON course_ratings(course_id);
CREATE INDEX idx_course_ratings_user_id ON course_ratings(user_id);
CREATE INDEX idx_course_ratings_rating ON course_ratings(rating);

-- Step 4: Create trigger to update average rating
CREATE OR REPLACE FUNCTION update_course_rating()
RETURNS TRIGGER AS $$
BEGIN
  UPDATE courses 
  SET 
    average_rating = (
      SELECT COALESCE(AVG(rating), 0) 
      FROM course_ratings 
      WHERE course_id = COALESCE(NEW.course_id, OLD.course_id)
    ),
    rating_count = (
      SELECT COUNT(*) 
      FROM course_ratings 
      WHERE course_id = COALESCE(NEW.course_id, OLD.course_id)
    )
  WHERE id = COALESCE(NEW.course_id, OLD.course_id);
  
  RETURN COALESCE(NEW, OLD);
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_update_course_rating
AFTER INSERT OR UPDATE OR DELETE ON course_ratings
FOR EACH ROW EXECUTE FUNCTION update_course_rating();
```

### Database Monitoring and Analytics
✅ **Good Example: Performance Monitoring Queries**
```sql
-- Monitor slow queries
SELECT 
  query,
  calls,
  total_time,
  mean_time,
  rows
FROM pg_stat_statements 
WHERE mean_time > 100 -- Queries taking more than 100ms on average
ORDER BY mean_time DESC;

-- Monitor table sizes and growth
SELECT 
  schemaname,
  tablename,
  attname,
  n_distinct,
  correlation
FROM pg_stats 
WHERE tablename IN ('courses', 'users', 'progress', 'enrollments')
ORDER BY tablename, attname;

-- Check index usage
SELECT 
  schemaname,
  tablename,
  indexname,
  idx_scan,
  idx_tup_read,
  idx_tup_fetch
FROM pg_stat_user_indexes 
WHERE idx_scan = 0 -- Unused indexes
ORDER BY schemaname, tablename;
```

## Key Takeaways for LMS Database Design

1. **Design for Educational Hierarchy** - Course → Lesson → Section structure with proper relationships
2. **Optimize for Progress Tracking** - Efficient queries for user progress and analytics
3. **Implement Proper Constraints** - Data integrity at database level with application validation
4. **Plan for Scale** - Strategic indexing and query optimization from the start
5. **Audit Everything** - Track changes to educational content and user progress
6. **Flexible Content Management** - Support for different content types and versioning
7. **Multi-Tenant Ready** - Design for organizational separation if needed
8. **Performance First** - Index strategy and query optimization for educational workflows

These patterns ensure a robust, scalable, and maintainable database architecture specifically designed for learning management system requirements. 