# LMS Database Optimization & Educational Data Patterns (2025)

## 🔄 Fundamental Principle: Constant Research & Web Updates
**Critical Success Factor**: This guide must be continuously updated with latest 2025+ web research on database optimization patterns for educational platforms. All team members must regularly research and update this knowledge base.

## Database Architecture for Educational Platforms

### 1. Educational Data Schema Design

#### ✅ Good Example: Optimized Educational Database Schema
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ EDUCATIONAL DATABASE DESIGN ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
**Database Technology**: PostgreSQL 16+ with Prisma ORM for type-safe educational data operations

**Educational Schema Architecture**:
```typescript
// Good: Normalized educational database schema with performance optimization
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
  previewFeatures = ["fullTextSearch", "jsonProtocol"]
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Educational institution hierarchy
model Institution {
  id          String   @id @default(cuid())
  name        String   @db.VarChar(200)
  domain      String   @unique @db.VarChar(100)
  type        InstitutionType
  settings    Json     @default("{}")
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  // Educational relationships
  departments Department[]
  courses     Course[]
  users       User[]
  
  @@map("institutions")
  @@index([domain])
  @@index([type, createdAt])
}

// Educational user model with role-based structure
model User {
  id            String   @id @default(cuid())
  email         String   @unique @db.VarChar(255)
  name          String   @db.VarChar(100)
  role          UserRole
  institutionId String
  profileData   Json     @default("{}")
  preferences   Json     @default("{}")
  isActive      Boolean  @default(true)
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
  lastLoginAt   DateTime?
  
  // Educational relationships
  institution   Institution @relation(fields: [institutionId], references: [id], onDelete: Cascade)
  
  // Student-specific relationships
  enrollments   Enrollment[] @relation("StudentEnrollments")
  submissions   Submission[]
  progressRecords LessonProgress[]
  
  // Instructor-specific relationships
  instructedCourses Course[] @relation("InstructorCourses")
  assignedGrades    Grade[]
  
  @@map("users")
  @@index([email])
  @@index([institutionId, role])
  @@index([role, isActive])
  @@index([lastLoginAt])
}

// Optimized course structure for educational content
model Course {
  id              String       @id @default(cuid())
  title           String       @db.VarChar(200)
  description     String       @db.Text
  slug            String       @unique @db.VarChar(150)
  category        CourseCategory
  difficulty      DifficultyLevel
  status          CourseStatus @default(DRAFT)
  price           Decimal      @db.Decimal(10,2)
  estimatedHours  Int
  institutionId   String
  instructorId    String
  metadata        Json         @default("{}")
  settings        Json         @default("{}")
  createdAt       DateTime     @default(now())
  updatedAt       DateTime     @updatedAt
  publishedAt     DateTime?
  
  // Educational relationships
  institution     Institution @relation(fields: [institutionId], references: [id])
  instructor      User        @relation("InstructorCourses", fields: [instructorId], references: [id])
  
  modules         Module[]
  enrollments     Enrollment[]
  assignments     Assignment[]
  analytics       CourseAnalytics[]
  prerequisites   CoursePrerequisite[] @relation("PrerequisiteCourse")
  dependentCourses CoursePrerequisite[] @relation("DependentCourse")
  
  @@map("courses")
  @@index([slug])
  @@index([institutionId, status])
  @@index([category, difficulty])
  @@index([instructorId, status])
  @@index([publishedAt, status])
  @@fulltext([title, description])
}

// Hierarchical content structure for educational modules
model Module {
  id          String   @id @default(cuid())
  title       String   @db.VarChar(150)
  description String   @db.Text
  orderIndex  Int
  courseId    String
  settings    Json     @default("{}")
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  course      Course   @relation(fields: [courseId], references: [id], onDelete: Cascade)
  lessons     Lesson[]
  
  @@map("modules")
  @@index([courseId, orderIndex])
  @@unique([courseId, orderIndex])
}

// Optimized lesson content with type-specific handling
model Lesson {
  id          String     @id @default(cuid())
  title       String     @db.VarChar(150)
  type        LessonType
  content     Json       // Polymorphic content based on type
  duration    Int        // Duration in minutes
  orderIndex  Int
  moduleId    String
  settings    Json       @default("{}")
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt
  
  module      Module     @relation(fields: [moduleId], references: [id], onDelete: Cascade)
  progress    LessonProgress[]
  
  @@map("lessons")
  @@index([moduleId, orderIndex])
  @@unique([moduleId, orderIndex])
  @@index([type])
}

// Performance-optimized enrollment tracking
model Enrollment {
  id          String            @id @default(cuid())
  studentId   String
  courseId    String
  status      EnrollmentStatus  @default(ACTIVE)
  progress    Int               @default(0) // Percentage 0-100
  grade       Decimal?          @db.Decimal(5,2)
  enrolledAt  DateTime          @default(now())
  completedAt DateTime?
  lastAccessAt DateTime?
  
  student     User              @relation("StudentEnrollments", fields: [studentId], references: [id])
  course      Course            @relation(fields: [courseId], references: [id])
  
  @@map("enrollments")
  @@unique([studentId, courseId])
  @@index([studentId, status])
  @@index([courseId, status])
  @@index([enrolledAt])
  @@index([completedAt])
  @@index([lastAccessAt])
}

// Granular progress tracking for educational analytics
model LessonProgress {
  id          String    @id @default(cuid())
  studentId   String
  lessonId    String
  courseId    String    // Denormalized for query performance
  timeSpent   Int       // Time in seconds
  score       Decimal?  @db.Decimal(5,2)
  completedAt DateTime?
  attempts    Int       @default(1)
  metadata    Json      @default("{}")
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
  
  student     User      @relation(fields: [studentId], references: [id])
  lesson      Lesson    @relation(fields: [lessonId], references: [id])
  
  @@map("lesson_progress")
  @@unique([studentId, lessonId])
  @@index([studentId, courseId])
  @@index([courseId, completedAt])
  @@index([lessonId, completedAt])
  @@index([completedAt])
}

// Educational analytics aggregation tables
model CourseAnalytics {
  id              String   @id @default(cuid())
  courseId        String
  date            DateTime @db.Date
  enrollmentCount Int      @default(0)
  completionCount Int      @default(0)
  avgTimeSpent    Int      @default(0) // Average time in minutes
  avgScore        Decimal? @db.Decimal(5,2)
  engagement      Json     @default("{}")
  createdAt       DateTime @default(now())
  
  course          Course   @relation(fields: [courseId], references: [id])
  
  @@map("course_analytics")
  @@unique([courseId, date])
  @@index([date])
  @@index([courseId, date])
}

// Enums for type safety and query optimization
enum UserRole {
  STUDENT
  INSTRUCTOR
  ADMIN
  TEACHING_ASSISTANT
}

enum CourseStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

enum LessonType {
  VIDEO
  TEXT
  QUIZ
  ASSIGNMENT
  INTERACTIVE
}

enum EnrollmentStatus {
  ACTIVE
  COMPLETED
  DROPPED
  SUSPENDED
}
```
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ EDUCATIONAL DATABASE DESIGN ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#### ❌ Bad Example: Poorly Designed Educational Schema
```sql
-- Bad: Denormalized, unoptimized educational database
CREATE TABLE everything (
  id SERIAL PRIMARY KEY,
  user_data TEXT,           -- Storing JSON in TEXT field
  course_data TEXT,         -- No normalization
  progress_data TEXT,       -- No proper indexing
  created_at TIMESTAMP     -- Missing educational context and relationships
);

-- No proper relationships, indexing, or educational data structure
```

### 2. Query Optimization for Educational Data

#### ✅ Good Example: Optimized Educational Queries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ EDUCATIONAL QUERY OPTIMIZATION ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
**Performance Focus**: Sub-200ms response times for educational dashboards and analytics

**Educational Query Patterns**:
```typescript
// Good: Optimized queries for educational data retrieval
export class EducationalQueryService {
  private prisma: PrismaClient;
  
  constructor() {
    this.prisma = new PrismaClient({
      log: ['query', 'info', 'warn', 'error'],
      datasources: {
        db: {
          url: process.env.DATABASE_URL
        }
      }
    });
  }
  
  // Optimized student dashboard query with minimal N+1 queries
  async getStudentDashboard(studentId: string): Promise<StudentDashboard> {
    const [student, enrollments, recentProgress] = await Promise.all([
      // Student basic info
      this.prisma.user.findUnique({
        where: { id: studentId },
        select: {
          id: true,
          name: true,
          email: true,
          preferences: true
        }
      }),
      
      // Active enrollments with course details
      this.prisma.enrollment.findMany({
        where: {
          studentId,
          status: 'ACTIVE'
        },
        include: {
          course: {
            select: {
              id: true,
              title: true,
              description: true,
              category: true,
              difficulty: true,
              estimatedHours: true,
              instructor: {
                select: {
                  name: true,
                  email: true
                }
              }
            }
          }
        },
        orderBy: {
          lastAccessAt: 'desc'
        }
      }),
      
      // Recent learning progress (last 7 days)
      this.prisma.lessonProgress.findMany({
        where: {
          studentId,
          completedAt: {
            gte: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000)
          }
        },
        include: {
          lesson: {
            select: {
              title: true,
              type: true,
              module: {
                select: {
                  title: true,
                  course: {
                    select: {
                      title: true
                    }
                  }
                }
              }
            }
          }
        },
        orderBy: {
          completedAt: 'desc'
        },
        take: 10
      })
    ]);
    
    return {
      student,
      enrollments,
      recentProgress,
      totalCourses: enrollments.length,
      completedLessons: recentProgress.length
    };
  }
  
  // Highly optimized course analytics query
  async getCourseAnalytics(
    courseId: string,
    dateRange: { start: Date; end: Date }
  ): Promise<CourseAnalyticsData> {
    // Use raw SQL for complex analytical queries
    const analyticsQuery = this.prisma.$queryRaw<AnalyticsResult[]>`
      WITH enrollment_stats AS (
        SELECT 
          DATE(enrolled_at) as date,
          COUNT(*) as daily_enrollments,
          COUNT(*) FILTER (WHERE completed_at IS NOT NULL) as daily_completions
        FROM enrollments 
        WHERE course_id = ${courseId}
          AND enrolled_at BETWEEN ${dateRange.start} AND ${dateRange.end}
        GROUP BY DATE(enrolled_at)
      ),
      progress_stats AS (
        SELECT 
          DATE(completed_at) as date,
          AVG(time_spent) as avg_time_spent,
          AVG(score) as avg_score,
          COUNT(*) as lessons_completed
        FROM lesson_progress lp
        WHERE course_id = ${courseId}
          AND completed_at BETWEEN ${dateRange.start} AND ${dateRange.end}
        GROUP BY DATE(completed_at)
      )
      SELECT 
        COALESCE(e.date, p.date) as date,
        COALESCE(e.daily_enrollments, 0) as enrollments,
        COALESCE(e.daily_completions, 0) as completions,
        COALESCE(p.avg_time_spent, 0) as avg_time_spent,
        COALESCE(p.avg_score, 0) as avg_score,
        COALESCE(p.lessons_completed, 0) as lessons_completed
      FROM enrollment_stats e
      FULL OUTER JOIN progress_stats p ON e.date = p.date
      ORDER BY date;
    `;
    
    const results = await analyticsQuery;
    return this.processAnalyticsResults(results);
  }
  
  // Efficient course search with full-text search
  async searchCourses(params: {
    query?: string;
    category?: string;
    difficulty?: string;
    institutionId: string;
    page: number;
    limit: number;
  }): Promise<SearchResults> {
    const { query, category, difficulty, institutionId, page, limit } = params;
    const offset = (page - 1) * limit;
    
    // Build where clause for complex filtering
    const whereClause: any = {
      institutionId,
      status: 'PUBLISHED'
    };
    
    if (category) {
      whereClause.category = category;
    }
    
    if (difficulty) {
      whereClause.difficulty = difficulty;
    }
    
    if (query) {
      // Use PostgreSQL full-text search for educational content
      whereClause.OR = [
        { title: { search: query } },
        { description: { search: query } }
      ];
    }
    
    const [courses, totalCount] = await Promise.all([
      this.prisma.course.findMany({
        where: whereClause,
        select: {
          id: true,
          title: true,
          description: true,
          category: true,
          difficulty: true,
          price: true,
          estimatedHours: true,
          instructor: {
            select: {
              name: true
            }
          },
          _count: {
            select: {
              enrollments: true
            }
          }
        },
        orderBy: [
          { publishedAt: 'desc' },
          { title: 'asc' }
        ],
        skip: offset,
        take: limit
      }),
      
      this.prisma.course.count({
        where: whereClause
      })
    ]);
    
    return {
      courses,
      pagination: {
        page,
        limit,
        total: totalCount,
        hasMore: offset + limit < totalCount
      }
    };
  }
  
  // Optimized progress tracking update with batch operations
  async updateLearningProgress(progressUpdates: LearningProgressUpdate[]): Promise<void> {
    await this.prisma.$transaction(async (tx) => {
      // Batch update lesson progress
      const progressPromises = progressUpdates.map(update => 
        tx.lessonProgress.upsert({
          where: {
            studentId_lessonId: {
              studentId: update.studentId,
              lessonId: update.lessonId
            }
          },
          update: {
            timeSpent: { increment: update.timeSpent },
            score: update.score,
            completedAt: update.completedAt,
            attempts: { increment: 1 }
          },
          create: {
            studentId: update.studentId,
            lessonId: update.lessonId,
            courseId: update.courseId,
            timeSpent: update.timeSpent,
            score: update.score,
            completedAt: update.completedAt,
            attempts: 1
          }
        })
      );
      
      await Promise.all(progressPromises);
      
      // Update enrollment progress efficiently
      const enrollmentUpdates = new Map<string, { progress: number; lastAccess: Date }>();
      
      for (const update of progressUpdates) {
        const key = `${update.studentId}-${update.courseId}`;
        
        if (!enrollmentUpdates.has(key)) {
          // Calculate updated progress percentage
          const courseProgress = await this.calculateCourseProgress(
            update.studentId,
            update.courseId,
            tx
          );
          
          enrollmentUpdates.set(key, {
            progress: courseProgress,
            lastAccess: new Date()
          });
        }
      }
      
      // Batch update enrollments
      const enrollmentPromises = Array.from(enrollmentUpdates.entries()).map(
        ([key, data]) => {
          const [studentId, courseId] = key.split('-');
          return tx.enrollment.update({
            where: {
              studentId_courseId: {
                studentId,
                courseId
              }
            },
            data: {
              progress: data.progress,
              lastAccessAt: data.lastAccess,
              ...(data.progress === 100 && { completedAt: new Date() })
            }
          });
        }
      );
      
      await Promise.all(enrollmentPromises);
    });
  }
  
  private async calculateCourseProgress(
    studentId: string,
    courseId: string,
    tx: any
  ): Promise<number> {
    const [completedLessons, totalLessons] = await Promise.all([
      tx.lessonProgress.count({
        where: {
          studentId,
          courseId,
          completedAt: { not: null }
        }
      }),
      tx.lesson.count({
        where: {
          module: {
            courseId
          }
        }
      })
    ]);
    
    return totalLessons > 0 ? Math.round((completedLessons / totalLessons) * 100) : 0;
  }
}
```
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ EDUCATIONAL QUERY OPTIMIZATION ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#### ❌ Bad Example: Unoptimized Educational Queries
```typescript
// Bad: N+1 queries and inefficient data retrieval
async getStudentDashboard(studentId: string) {
  const student = await prisma.user.findUnique({ where: { id: studentId } });
  
  const enrollments = await prisma.enrollment.findMany({
    where: { studentId }
  });
  
  // N+1 query problem - fetching courses one by one
  const courses = [];
  for (const enrollment of enrollments) {
    const course = await prisma.course.findUnique({
      where: { id: enrollment.courseId }
    });
    courses.push(course);
  }
  
  // No optimization, missing indexes, inefficient data structure
  return { student, enrollments, courses };
}
```

### 3. Database Performance Optimization

#### ✅ Good Example: Educational Database Performance Tuning
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ DATABASE PERFORMANCE OPTIMIZATION ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
**Performance Target**: <100ms for educational dashboard queries, <500ms for complex analytics

**Database Optimization Strategies**:
```sql
-- Good: Optimized PostgreSQL configuration for educational workloads

-- Educational-specific indexes for common query patterns
CREATE INDEX CONCURRENTLY idx_enrollments_student_status_performance 
ON enrollments (student_id, status) 
INCLUDE (progress, last_access_at, completed_at);

CREATE INDEX CONCURRENTLY idx_lesson_progress_course_completion 
ON lesson_progress (course_id, completed_at) 
WHERE completed_at IS NOT NULL;

CREATE INDEX CONCURRENTLY idx_courses_search_performance 
ON courses (institution_id, status, category, difficulty)
INCLUDE (title, description, price);

-- Full-text search index for educational content
CREATE INDEX CONCURRENTLY idx_courses_fulltext_search 
ON courses USING gin(to_tsvector('english', title || ' ' || description));

-- Partial indexes for active educational data
CREATE INDEX CONCURRENTLY idx_users_active_educational 
ON users (institution_id, role) 
WHERE is_active = true;

-- Composite indexes for educational analytics
CREATE INDEX CONCURRENTLY idx_course_analytics_date_range 
ON course_analytics (course_id, date DESC)
INCLUDE (enrollment_count, completion_count, avg_score);

-- Partitioning for large educational datasets
CREATE TABLE lesson_progress_partitioned (
  LIKE lesson_progress INCLUDING ALL
) PARTITION BY RANGE (created_at);

-- Create monthly partitions for educational progress data
CREATE TABLE lesson_progress_y2025m01 PARTITION OF lesson_progress_partitioned
FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');

CREATE TABLE lesson_progress_y2025m02 PARTITION OF lesson_progress_partitioned
FOR VALUES FROM ('2025-02-01') TO ('2025-03-01');

-- Materialized views for educational analytics
CREATE MATERIALIZED VIEW course_performance_summary AS
SELECT 
  c.id as course_id,
  c.title,
  c.category,
  c.difficulty,
  COUNT(e.id) as total_enrollments,
  COUNT(e.id) FILTER (WHERE e.completed_at IS NOT NULL) as completions,
  ROUND(AVG(e.progress), 2) as avg_progress,
  ROUND(AVG(lp.score), 2) as avg_score,
  SUM(lp.time_spent) / 3600 as total_hours_spent
FROM courses c
LEFT JOIN enrollments e ON c.id = e.course_id
LEFT JOIN lesson_progress lp ON c.id = lp.course_id
WHERE c.status = 'PUBLISHED'
GROUP BY c.id, c.title, c.category, c.difficulty;

-- Refresh strategy for materialized views
CREATE OR REPLACE FUNCTION refresh_educational_analytics()
RETURNS void AS $$
BEGIN
  REFRESH MATERIALIZED VIEW CONCURRENTLY course_performance_summary;
  -- Add other educational materialized views here
END;
$$ LANGUAGE plpgsql;

-- Schedule regular refresh of educational analytics
SELECT cron.schedule('refresh-educational-analytics', '0 */6 * * *', 'SELECT refresh_educational_analytics();');
```

```typescript
// Database connection optimization for educational workloads
export class EducationalDatabaseConfig {
  static getOptimizedPrismaConfig(): PrismaClientOptions {
    return {
      datasources: {
        db: {
          url: process.env.DATABASE_URL
        }
      },
      log: [
        { level: 'query', emit: 'event' },
        { level: 'error', emit: 'event' },
        { level: 'info', emit: 'event' },
        { level: 'warn', emit: 'event' }
      ],
      // Educational workload optimizations
      __internal: {
        engine: {
          // Optimize for educational read-heavy workloads
          connectionLimit: 100,
          poolTimeout: 30000,
          // Educational-specific query timeout
          transactionTimeout: 10000
        }
      }
    };
  }
  
  // Connection pooling for educational microservices
  static setupConnectionPooling(): Pool {
    return new Pool({
      connectionString: process.env.DATABASE_URL,
      // Educational platform connection settings
      min: 10,           // Minimum connections for educational services
      max: 50,           // Maximum connections for peak educational usage
      idleTimeoutMillis: 30000,
      connectionTimeoutMillis: 5000,
      // Educational query timeout
      query_timeout: 10000,
      statement_timeout: 15000,
      // Educational-specific pool configuration
      application_name: 'educational-lms-service'
    });
  }
  
  // Query monitoring for educational performance
  static setupQueryMonitoring(prisma: PrismaClient): void {
    prisma.$on('query', (e) => {
      // Monitor slow educational queries (>500ms)
      if (e.duration > 500) {
        console.warn('Slow Educational Query Detected:', {
          query: e.query,
          duration: e.duration,
          params: e.params,
          timestamp: new Date().toISOString(),
          context: 'educational-database-operation'
        });
        
        // Send alert for very slow queries (>2s)
        if (e.duration > 2000) {
          this.sendSlowQueryAlert(e);
        }
      }
    });
    
    // Monitor educational database errors
    prisma.$on('error', (e) => {
      console.error('Educational Database Error:', {
        message: e.message,
        target: e.target,
        timestamp: new Date().toISOString(),
        context: 'educational-database-error'
      });
    });
  }
  
  private static async sendSlowQueryAlert(queryEvent: any): Promise<void> {
    // Implementation for alerting on slow educational queries
    await AlertingService.send({
      type: 'SLOW_EDUCATIONAL_QUERY',
      severity: 'HIGH',
      message: `Educational query exceeded 2s threshold: ${queryEvent.duration}ms`,
      metadata: {
        query: queryEvent.query,
        duration: queryEvent.duration,
        educational_context: true
      }
    });
  }
}

// Caching layer for educational data
export class EducationalCacheService {
  private redis: Redis;
  
  constructor() {
    this.redis = new Redis({
      host: process.env.REDIS_HOST,
      port: parseInt(process.env.REDIS_PORT || '6379'),
      // Educational cache configuration
      maxRetriesPerRequest: 3,
      retryDelayOnFailover: 100,
      keyPrefix: 'edu:',
      lazyConnect: true
    });
  }
  
  // Cache educational dashboard data
  async cacheStudentDashboard(
    studentId: string,
    dashboardData: StudentDashboard,
    ttl: number = 300 // 5 minutes for educational data
  ): Promise<void> {
    const key = `student:dashboard:${studentId}`;
    await this.redis.setex(
      key,
      ttl,
      JSON.stringify({
        ...dashboardData,
        cachedAt: new Date().toISOString(),
        version: 'edu-cache-v1.0'
      })
    );
  }
  
  // Cache course analytics with longer TTL
  async cacheCourseAnalytics(
    courseId: string,
    analyticsData: CourseAnalyticsData,
    ttl: number = 1800 // 30 minutes for analytics
  ): Promise<void> {
    const key = `course:analytics:${courseId}`;
    await this.redis.setex(key, ttl, JSON.stringify(analyticsData));
  }
  
  // Invalidate educational cache patterns
  async invalidateEducationalCache(pattern: string): Promise<void> {
    const keys = await this.redis.keys(`edu:${pattern}*`);
    if (keys.length > 0) {
      await this.redis.del(...keys);
    }
  }
  
  // Cache warm-up for popular educational content
  async warmUpEducationalCache(): Promise<void> {
    // Pre-cache popular courses
    const popularCourses = await this.getPopularCourses();
    
    const cachePromises = popularCourses.map(course =>
      this.cacheCourseData(course.id, course, 3600) // 1 hour TTL
    );
    
    await Promise.all(cachePromises);
  }
}
```
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ DATABASE PERFORMANCE OPTIMIZATION ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#### ❌ Bad Example: Poor Database Performance
```typescript
// Bad: No optimization, caching, or performance considerations
class BadEducationalService {
  async getStudentData(studentId: string) {
    // No caching, direct database hit every time
    const student = await db.query('SELECT * FROM users WHERE id = ?', [studentId]);
    
    // No connection pooling or query optimization
    const courses = await db.query('SELECT * FROM courses');
    
    // No indexes, full table scans
    const progress = await db.query('SELECT * FROM progress WHERE student_id = ?', [studentId]);
    
    return { student, courses, progress };
  }
}
```

## Database Optimization Best Practices Summary

### ✅ DO's for Educational Database Optimization:
- ✅ Design normalized schema with educational relationship modeling
- ✅ Create composite indexes for common educational query patterns
- ✅ Use PostgreSQL full-text search for educational content discovery
- ✅ Implement connection pooling optimized for educational workloads
- ✅ Use materialized views for complex educational analytics
- ✅ Partition large tables by time for educational progress data
- ✅ Cache frequently accessed educational data with appropriate TTL
- ✅ Monitor query performance and educational-specific bottlenecks
- ✅ Use batch operations for educational progress updates
- ✅ Implement proper data retention for educational compliance

### ❌ DON'Ts for Educational Database Optimization:
- ❌ Use denormalized schemas without proper educational context
- ❌ Skip indexing on frequently queried educational fields
- ❌ Ignore connection pooling for educational microservices
- ❌ Run complex analytics queries without optimization
- ❌ Cache educational data without considering privacy requirements
- ❌ Use synchronous operations for batch educational updates
- ❌ Ignore query monitoring for educational performance issues
- ❌ Store educational media files directly in database
- ❌ Skip database migrations for educational schema changes
- ❌ Use generic optimization without educational workload consideration

Remember: Keep this database guide updated with latest PostgreSQL optimization techniques and educational data management patterns. Research and validate these patterns against current 2025+ database performance standards and educational platform requirements. 