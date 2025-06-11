# LMS Database Optimization with Prisma

## Advanced Query Patterns for Educational Platforms

### Course and Lesson Relationship Optimization
**Critical for LMS platform performance with complex course structures**

✅ **Good Example: Optimized Course Loading with Selective Includes**
```typescript
// services/courseService.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

// Efficient course listing with minimal data
export const getCourseList = async (page: number = 1, limit: number = 20) => {
  const skip = (page - 1) * limit;
  
  return await prisma.course.findMany({
    skip,
    take: limit,
    select: {
      id: true,
      title: true,
      description: true,
      difficulty: true,
      imageUrl: true,
      estimatedHours: true,
      isPublished: true,
      createdAt: true,
      _count: {
        select: {
          lessons: true,
          enrollments: true
        }
      }
    },
    where: {
      isPublished: true
    },
    orderBy: {
      createdAt: 'desc'
    }
  });
};

// Detailed course loading with optimized includes
export const getCourseWithLessons = async (courseId: string) => {
  return await prisma.course.findUnique({
    where: { id: courseId },
    include: {
      lessons: {
        orderBy: { order: 'asc' },
        include: {
          sections: {
            select: {
              id: true,
              title: true,
              type: true,
              order: true,
              duration: true
            },
            orderBy: { order: 'asc' }
          },
          _count: {
            select: { sections: true }
          }
        }
      },
      _count: {
        select: {
          enrollments: true,
          lessons: true
        }
      }
    }
  });
};

// Progress-aware course loading for authenticated users
export const getCourseWithProgress = async (courseId: string, userId: string) => {
  const [course, userProgress] = await Promise.all([
    prisma.course.findUnique({
      where: { id: courseId },
      include: {
        lessons: {
          orderBy: { order: 'asc' },
          include: {
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
          }
        }
      }
    }),
    prisma.progress.findMany({
      where: {
        userId,
        section: {
          lesson: {
            courseId
          }
        }
      },
      include: {
        section: {
          select: {
            id: true,
            lessonId: true
          }
        }
      }
    })
  ]);

  return {
    course,
    progress: userProgress
  };
};
```

❌ **Bad Example: Inefficient Course Loading**
```typescript
// Inefficient - loads unnecessary data
export const getCourseList = async () => {
  return await prisma.course.findMany({
    include: {
      lessons: {
        include: {
          sections: true // Loads all section content unnecessarily
        }
      },
      enrollments: true // Loads all enrollment data
    }
  });
};
```

### Progress Tracking Optimization Patterns
✅ **Good Example: Efficient Progress Queries with Aggregation**
```typescript
// services/progressService.ts

// Optimized progress calculation with raw SQL for performance
export const calculateCourseProgress = async (courseId: string, userId: string) => {
  const result = await prisma.$queryRaw`
    SELECT 
      c.id as course_id,
      c.title as course_title,
      COUNT(DISTINCT s.id) as total_sections,
      COUNT(DISTINCT p.section_id) as completed_sections,
      COALESCE(SUM(p.time_spent), 0) as total_time_spent,
      ROUND(
        (COUNT(DISTINCT p.section_id)::DECIMAL / COUNT(DISTINCT s.id)) * 100, 
        2
      ) as completion_percentage
    FROM courses c
    JOIN lessons l ON l.course_id = c.id
    JOIN sections s ON s.lesson_id = l.id
    LEFT JOIN progress p ON p.section_id = s.id AND p.user_id = ${userId}
    WHERE c.id = ${courseId}
    GROUP BY c.id, c.title
  `;

  return result[0];
};

// Batch progress updates for multiple sections
export const updateMultipleProgress = async (progressUpdates: Array<{
  userId: string;
  sectionId: string;
  timeSpent: number;
  completed: boolean;
}>) => {
  const operations = progressUpdates.map(update => 
    prisma.progress.upsert({
      where: {
        userId_sectionId: {
          userId: update.userId,
          sectionId: update.sectionId
        }
      },
      update: {
        timeSpent: { increment: update.timeSpent },
        completed: update.completed,
        updatedAt: new Date()
      },
      create: {
        userId: update.userId,
        sectionId: update.sectionId,
        timeSpent: update.timeSpent,
        completed: update.completed
      }
    })
  );

  return await prisma.$transaction(operations);
};

// Efficient lesson progress query
export const getLessonProgress = async (lessonId: string, userId: string) => {
  return await prisma.lesson.findUnique({
    where: { id: lessonId },
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
          duration: true,
          progress: {
            where: { userId },
            select: {
              completed: true,
              timeSpent: true,
              updatedAt: true
            }
          }
        },
        orderBy: { order: 'asc' }
      },
      _count: {
        select: { sections: true }
      }
    }
  });
};
```

### Advanced Query Optimization with Indexes
✅ **Good Example: Database Indexes for LMS Performance**
```sql
-- schema.prisma indexes for optimal query performance

model Course {
  id            String   @id @default(cuid())
  title         String
  description   String?
  difficulty    String
  isPublished   Boolean  @default(false)
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
  
  lessons       Lesson[]
  enrollments   Enrollment[]
  
  // Indexes for common queries
  @@index([isPublished, createdAt(sort: Desc)])
  @@index([difficulty])
  @@index([createdAt])
}

model Lesson {
  id          String    @id @default(cuid())
  title       String
  order       Int
  courseId    String
  createdAt   DateTime  @default(now())
  
  course      Course    @relation(fields: [courseId], references: [id], onDelete: Cascade)
  sections    Section[]

  // Compound index for course lesson ordering
  @@index([courseId, order])
  @@index([courseId])
}

model Section {
  id          String    @id @default(cuid())
  title       String
  content     String
  type        String
  order       Int
  duration    Int?
  lessonId    String
  
  lesson      Lesson    @relation(fields: [lessonId], references: [id], onDelete: Cascade)
  progress    Progress[]

  // Compound index for lesson section ordering
  @@index([lessonId, order])
}

model Progress {
  id          String    @id @default(cuid())
  userId      String
  sectionId   String
  completed   Boolean   @default(false)
  timeSpent   Int       @default(0)
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
  
  section     Section   @relation(fields: [sectionId], references: [id], onDelete: Cascade)
  
  // Unique constraint and indexes for progress queries
  @@unique([userId, sectionId])
  @@index([userId])
  @@index([sectionId])
  @@index([userId, completed])
}

model Enrollment {
  id          String    @id @default(cuid())
  userId      String
  courseId    String
  enrolledAt  DateTime  @default(now())
  
  course      Course    @relation(fields: [courseId], references: [id], onDelete: Cascade)
  
  @@unique([userId, courseId])
  @@index([userId])
  @@index([courseId])
}
```

### Batch Operations and Transaction Patterns
✅ **Good Example: Efficient Bulk Course Generation**
```typescript
// services/courseGenerationService.ts

interface CourseGenerationData {
  courseData: {
    title: string;
    description: string;
    difficulty: string;
    imageUrl?: string;
  };
  lessons: Array<{
    title: string;
    order: number;
    sections: Array<{
      title: string;
      content: string;
      type: string;
      order: number;
      duration?: number;
    }>;
  }>;
}

// Atomic course creation with all related data
export const createCourseWithContent = async (data: CourseGenerationData) => {
  return await prisma.$transaction(async (tx) => {
    // Create course
    const course = await tx.course.create({
      data: {
        title: data.courseData.title,
        description: data.courseData.description,
        difficulty: data.courseData.difficulty,
        imageUrl: data.courseData.imageUrl,
        isPublished: true
      }
    });

    // Create lessons with sections in batch
    const lessonCreations = data.lessons.map(lesson => 
      tx.lesson.create({
        data: {
          title: lesson.title,
          order: lesson.order,
          courseId: course.id,
          sections: {
            createMany: {
              data: lesson.sections.map(section => ({
                title: section.title,
                content: section.content,
                type: section.type,
                order: section.order,
                duration: section.duration
              }))
            }
          }
        }
      })
    );

    await Promise.all(lessonCreations);

    return course;
  });
};

// Bulk progress update with optimized queries
export const updateBulkProgress = async (updates: Array<{
  userId: string;
  sectionIds: string[];
  timeSpent: number;
}>) => {
  return await prisma.$transaction(async (tx) => {
    const operations = [];

    for (const update of updates) {
      // Batch upsert for multiple sections per user
      const upsertOperations = update.sectionIds.map(sectionId =>
        tx.progress.upsert({
          where: {
            userId_sectionId: {
              userId: update.userId,
              sectionId
            }
          },
          update: {
            timeSpent: { increment: update.timeSpent },
            updatedAt: new Date()
          },
          create: {
            userId: update.userId,
            sectionId,
            timeSpent: update.timeSpent,
            completed: false
          }
        })
      );

      operations.push(...upsertOperations);
    }

    return await Promise.all(operations);
  });
};
```

### Connection Pool and Performance Optimization
✅ **Good Example: Prisma Configuration for Production**
```typescript
// lib/prisma.ts
import { PrismaClient } from '@prisma/client';

// Global Prisma instance with optimized configuration
const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma = globalForPrisma.prisma ?? new PrismaClient({
  log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
  datasources: {
    db: {
      url: process.env.DATABASE_URL
    }
  },
  // Connection pool configuration for production
  connectionPool: {
    timeout: 20000,
    maxWait: 5000,
    maxConnections: 20
  }
});

// Prevent multiple instances in development
if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;

// Graceful shutdown
process.on('beforeExit', async () => {
  await prisma.$disconnect();
});

// Middleware for query performance monitoring
prisma.$use(async (params, next) => {
  const start = Date.now();
  const result = await next(params);
  const end = Date.now();
  
  // Log slow queries in production
  if (end - start > 1000) {
    console.warn(`Slow query detected: ${params.model}.${params.action} took ${end - start}ms`);
  }
  
  return result;
});

export default prisma;
```

### Caching Strategies for Database Queries
✅ **Good Example: Redis Caching for Course Data**
```typescript
// lib/cache.ts
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

export class CourseCache {
  private static TTL = {
    COURSE_LIST: 300, // 5 minutes
    COURSE_DETAIL: 600, // 10 minutes
    USER_PROGRESS: 60 // 1 minute (frequently updated)
  };

  static async getCourseList(page: number, limit: number) {
    const cacheKey = `courses:list:${page}:${limit}`;
    const cached = await redis.get(cacheKey);
    
    if (cached) {
      return JSON.parse(cached);
    }

    const courses = await getCourseList(page, limit);
    await redis.setex(cacheKey, this.TTL.COURSE_LIST, JSON.stringify(courses));
    
    return courses;
  }

  static async getCourseDetail(courseId: string) {
    const cacheKey = `course:detail:${courseId}`;
    const cached = await redis.get(cacheKey);
    
    if (cached) {
      return JSON.parse(cached);
    }

    const course = await getCourseWithLessons(courseId);
    if (course) {
      await redis.setex(cacheKey, this.TTL.COURSE_DETAIL, JSON.stringify(course));
    }
    
    return course;
  }

  static async getUserProgress(userId: string, courseId: string) {
    const cacheKey = `progress:${userId}:${courseId}`;
    const cached = await redis.get(cacheKey);
    
    if (cached) {
      return JSON.parse(cached);
    }

    const progress = await calculateCourseProgress(courseId, userId);
    await redis.setex(cacheKey, this.TTL.USER_PROGRESS, JSON.stringify(progress));
    
    return progress;
  }

  static async invalidateUserProgress(userId: string, courseId: string) {
    const cacheKey = `progress:${userId}:${courseId}`;
    await redis.del(cacheKey);
  }

  static async invalidateCourse(courseId: string) {
    const pattern = `course:detail:${courseId}`;
    await redis.del(pattern);
    
    // Also invalidate course list cache
    const listKeys = await redis.keys('courses:list:*');
    if (listKeys.length > 0) {
      await redis.del(...listKeys);
    }
  }
}
```

### Database Migration Best Practices
✅ **Good Example: Safe Migration Patterns**
```sql
-- migrations/20241201000001_add_course_analytics/migration.sql

-- Add new columns with safe defaults
ALTER TABLE "Course" ADD COLUMN "analytics_enabled" BOOLEAN NOT NULL DEFAULT true;
ALTER TABLE "Course" ADD COLUMN "completion_rate" DECIMAL(5,2) DEFAULT 0.00;

-- Create new analytics table
CREATE TABLE "CourseAnalytics" (
    "id" TEXT NOT NULL,
    "course_id" TEXT NOT NULL,
    "total_enrollments" INTEGER NOT NULL DEFAULT 0,
    "active_learners" INTEGER NOT NULL DEFAULT 0,
    "avg_completion_time" INTEGER NOT NULL DEFAULT 0,
    "completion_rate" DECIMAL(5,2) NOT NULL DEFAULT 0.00,
    "created_at" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "updated_at" TIMESTAMP(3) NOT NULL,

    CONSTRAINT "CourseAnalytics_pkey" PRIMARY KEY ("id")
);

-- Add indexes for performance
CREATE INDEX "CourseAnalytics_course_id_idx" ON "CourseAnalytics"("course_id");
CREATE INDEX "CourseAnalytics_completion_rate_idx" ON "CourseAnalytics"("completion_rate");

-- Add foreign key constraint
ALTER TABLE "CourseAnalytics" ADD CONSTRAINT "CourseAnalytics_course_id_fkey" 
FOREIGN KEY ("course_id") REFERENCES "Course"("id") ON DELETE CASCADE ON UPDATE CASCADE;

-- Backfill data for existing courses (optional)
INSERT INTO "CourseAnalytics" ("id", "course_id", "total_enrollments", "updated_at")
SELECT 
  gen_random_uuid()::text,
  c.id,
  COALESCE(enrollment_counts.count, 0),
  CURRENT_TIMESTAMP
FROM "Course" c
LEFT JOIN (
  SELECT course_id, COUNT(*) as count
  FROM "Enrollment"
  GROUP BY course_id
) enrollment_counts ON enrollment_counts.course_id = c.id;
```

## Key Database Optimization Takeaways

### Critical Database Performance Patterns ✅

1. **Selective Querying**
   - Use `select` to load only necessary fields
   - Implement proper `include` strategies for related data
   - Avoid N+1 queries with proper relationship loading

2. **Index Optimization**
   - Create compound indexes for common query patterns
   - Index foreign keys and frequently filtered columns
   - Monitor and optimize slow queries

3. **Batch Operations**
   - Use transactions for atomic multi-table operations
   - Implement bulk updates for progress tracking
   - Optimize course generation with batch creates

4. **Caching Strategies**
   - Cache frequently accessed course data
   - Implement proper cache invalidation
   - Use Redis for session and progress data

5. **Connection Management**
   - Configure proper connection pooling
   - Implement graceful shutdowns
   - Monitor database performance metrics

### Database Anti-Patterns to Avoid ❌

1. **Over-fetching Data** - Loading complete course structures when only metadata needed
2. **N+1 Query Problems** - Not using proper includes for related data fetching
3. **Missing Indexes** - Not indexing frequently queried fields and relationships
4. **Blocking Transactions** - Long-running transactions blocking other operations
5. **Cache Inconsistency** - Not properly invalidating cached data after updates

Proper database optimization is essential for LMS platforms handling thousands of courses, lessons, and user progress records. These patterns ensure scalable performance even with large educational datasets. 