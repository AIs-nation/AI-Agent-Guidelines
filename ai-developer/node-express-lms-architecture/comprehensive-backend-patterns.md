# Node.js/Express LMS Backend Architecture Guide

## Table of Contents
1. [LMS Backend Architecture Overview](#lms-backend-architecture-overview)
2. [Educational API Design Patterns](#educational-api-design-patterns)
3. [Database Schema for Learning Platforms](#database-schema-for-learning-platforms)
4. [Course Generation & Content Management](#course-generation--content-management)
5. [Progress Tracking & Analytics](#progress-tracking--analytics)
6. [Authentication & Authorization](#authentication--authorization)
7. [Performance Optimization](#performance-optimization)
8. [Error Handling & Validation](#error-handling--validation)

## LMS Backend Architecture Overview

### 1. Layered Architecture for Educational Systems

**Clean Architecture Structure:**
```javascript
// ✅ Good: Layered architecture for LMS backend
project/
├── src/
│   ├── controllers/         // API endpoints and request handling
│   │   ├── courseController.js
│   │   ├── lessonController.js
│   │   ├── progressController.js
│   │   └── authController.js
│   ├── services/           // Business logic layer
│   │   ├── courseService.js
│   │   ├── aiService.js
│   │   ├── progressService.js
│   │   └── analyticsService.js
│   ├── repositories/       // Data access layer
│   │   ├── courseRepository.js
│   │   ├── userRepository.js
│   │   └── progressRepository.js
│   ├── models/            // Database models
│   │   ├── Course.js
│   │   ├── Lesson.js
│   │   ├── User.js
│   │   └── Progress.js
│   ├── middleware/        // Custom middleware
│   │   ├── auth.js
│   │   ├── validation.js
│   │   └── errorHandler.js
│   ├── utils/            // Utility functions
│   │   ├── logger.js
│   │   ├── validators.js
│   │   └── helpers.js
│   └── config/           // Configuration
│       ├── database.js
│       ├── ai.js
│       └── server.js
```

**Express Application Setup:**
```javascript
// ✅ Good: Production-ready Express setup for LMS
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const compression = require('compression');
const { PrismaClient } = require('@prisma/client');

const app = express();
const prisma = new PrismaClient();

// Security middleware
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      scriptSrc: ["'self'"],
      imgSrc: ["'self'", "data:", "https:"],
    },
  },
}));

// CORS configuration for educational platform
app.use(cors({
  origin: process.env.FRONTEND_URL || 'http://localhost:3000',
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization'],
}));

// Rate limiting for educational APIs
const createRateLimit = (windowMs, max, message) => rateLimit({
  windowMs,
  max,
  message: { error: message },
  standardHeaders: true,
  legacyHeaders: false,
});

// Different rate limits for different endpoints
app.use('/api/courses/generate', createRateLimit(
  15 * 60 * 1000, // 15 minutes
  5, // 5 course generations per 15 minutes
  'Too many course generation requests, please try again later'
));

app.use('/api/auth', createRateLimit(
  15 * 60 * 1000, // 15 minutes
  5, // 5 login attempts per 15 minutes
  'Too many authentication attempts'
));

// General API rate limit
app.use('/api/', createRateLimit(
  15 * 60 * 1000, // 15 minutes
  100, // 100 requests per 15 minutes
  'Too many requests'
));

// Body parsing and compression
app.use(compression());
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true, limit: '10mb' }));

// Custom middleware for LMS
app.use(require('./middleware/requestLogger'));
app.use(require('./middleware/errorHandler'));

// Routes
app.use('/api/courses', require('./routes/courses'));
app.use('/api/lessons', require('./routes/lessons'));
app.use('/api/progress', require('./routes/progress'));
app.use('/api/auth', require('./routes/auth'));
app.use('/api/analytics', require('./routes/analytics'));

// Health check endpoint
app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    environment: process.env.NODE_ENV
  });
});

module.exports = app;
```

### 2. Microservices Considerations

**Service Separation for LMS:**
```javascript
// ✅ Good: Service-oriented architecture for large LMS platforms
const services = {
  // Core learning services
  courseService: 'http://course-service:3001',
  progressService: 'http://progress-service:3002',
  analyticsService: 'http://analytics-service:3003',
  
  // Content and AI services
  aiGenerationService: 'http://ai-service:3004',
  contentService: 'http://content-service:3005',
  mediaService: 'http://media-service:3006',
  
  // User and authentication services
  authService: 'http://auth-service:3007',
  notificationService: 'http://notification-service:3008',
  
  // External integrations
  paymentService: 'http://payment-service:3009',
  emailService: 'http://email-service:3010'
};

// Service communication helper
class ServiceCommunicator {
  constructor() {
    this.httpClient = axios.create({
      timeout: 30000,
      headers: {
        'Content-Type': 'application/json'
      }
    });
  }

  async callService(serviceName, endpoint, method = 'GET', data = null) {
    try {
      const baseURL = services[serviceName];
      if (!baseURL) {
        throw new Error(`Service ${serviceName} not found`);
      }

      const config = {
        method,
        url: `${baseURL}${endpoint}`,
        ...(data && { data })
      };

      const response = await this.httpClient(config);
      return response.data;
    } catch (error) {
      console.error(`Service call failed: ${serviceName}${endpoint}`, error);
      throw new Error(`${serviceName} service unavailable`);
    }
  }
}
```

## Educational API Design Patterns

### 1. Course Management APIs

**RESTful Course Endpoints:**
```javascript
// ✅ Good: Comprehensive course API design
const express = require('express');
const router = express.Router();
const courseController = require('../controllers/courseController');
const { authenticate, authorize } = require('../middleware/auth');
const { validateCourse, validateCourseUpdate } = require('../middleware/validation');

// Course CRUD operations
router.get('/', courseController.getAllCourses);
router.get('/:id', courseController.getCourseById);
router.post('/', authenticate, validateCourse, courseController.createCourse);
router.put('/:id', authenticate, authorize(['instructor', 'admin']), validateCourseUpdate, courseController.updateCourse);
router.delete('/:id', authenticate, authorize(['admin']), courseController.deleteCourse);

// Course generation and AI integration
router.post('/generate', authenticate, courseController.generateCourse);
router.get('/generate/:jobId/status', authenticate, courseController.getGenerationStatus);

// Course enrollment and student management
router.post('/:id/enroll', authenticate, courseController.enrollStudent);
router.delete('/:id/enroll', authenticate, courseController.unenrollStudent);
router.get('/:id/students', authenticate, authorize(['instructor', 'admin']), courseController.getCourseStudents);

// Course content and lessons
router.get('/:id/lessons', courseController.getCourseLessons);
router.post('/:id/lessons', authenticate, authorize(['instructor', 'admin']), courseController.addLessonToCourse);

// Course analytics and progress
router.get('/:id/analytics', authenticate, authorize(['instructor', 'admin']), courseController.getCourseAnalytics);
router.get('/:id/progress', authenticate, courseController.getCourseProgress);

module.exports = router;
```

**Course Controller Implementation:**
```javascript
// ✅ Good: Clean controller with proper error handling
const courseService = require('../services/courseService');
const { validationResult } = require('express-validator');
const { AppError } = require('../utils/errors');

class CourseController {
  async getAllCourses(req, res, next) {
    try {
      const { page = 1, limit = 10, category, difficulty, search } = req.query;
      
      const filters = {
        ...(category && { category }),
        ...(difficulty && { difficulty }),
        ...(search && { 
          OR: [
            { title: { contains: search, mode: 'insensitive' } },
            { description: { contains: search, mode: 'insensitive' } }
          ]
        })
      };

      const courses = await courseService.getPaginatedCourses({
        page: parseInt(page),
        limit: parseInt(limit),
        filters
      });

      res.status(200).json({
        success: true,
        data: courses,
        pagination: {
          page: parseInt(page),
          limit: parseInt(limit),
          total: courses.total,
          pages: Math.ceil(courses.total / limit)
        }
      });
    } catch (error) {
      next(error);
    }
  }

  async getCourseById(req, res, next) {
    try {
      const { id } = req.params;
      const { includeProgress = false } = req.query;
      const userId = req.user?.id;

      const course = await courseService.getCourseById(id, {
        includeProgress: includeProgress && userId,
        userId
      });

      if (!course) {
        throw new AppError('Course not found', 404);
      }

      res.status(200).json({
        success: true,
        data: course
      });
    } catch (error) {
      next(error);
    }
  }

  async generateCourse(req, res, next) {
    try {
      const errors = validationResult(req);
      if (!errors.isEmpty()) {
        throw new AppError('Validation failed', 400, errors.array());
      }

      const { topic, difficulty = 'intermediate', duration, requirements } = req.body;
      const userId = req.user.id;

      // Start asynchronous course generation
      const generationJob = await courseService.startCourseGeneration({
        topic,
        difficulty,
        duration,
        requirements,
        createdBy: userId
      });

      res.status(202).json({
        success: true,
        data: {
          jobId: generationJob.id,
          status: 'started',
          estimatedCompletion: generationJob.estimatedCompletion
        },
        message: 'Course generation started'
      });
    } catch (error) {
      next(error);
    }
  }

  async enrollStudent(req, res, next) {
    try {
      const { id: courseId } = req.params;
      const studentId = req.user.id;

      const enrollment = await courseService.enrollStudent(courseId, studentId);

      res.status(201).json({
        success: true,
        data: enrollment,
        message: 'Successfully enrolled in course'
      });
    } catch (error) {
      next(error);
    }
  }
}

module.exports = new CourseController();
```

### 2. Progress Tracking APIs

**Progress API Design:**
```javascript
// ✅ Good: Comprehensive progress tracking API
const express = require('express');
const router = express.Router();
const progressController = require('../controllers/progressController');
const { authenticate } = require('../middleware/auth');

// Student progress endpoints
router.get('/courses/:courseId', authenticate, progressController.getCourseProgress);
router.get('/lessons/:lessonId', authenticate, progressController.getLessonProgress);
router.post('/lessons/:lessonId/sections/:sectionId/complete', authenticate, progressController.markSectionComplete);
router.post('/lessons/:lessonId/complete', authenticate, progressController.markLessonComplete);

// Time tracking
router.post('/time', authenticate, progressController.recordTimeSpent);
router.get('/time/summary', authenticate, progressController.getTimeSpentSummary);

// Achievement and streak tracking
router.get('/achievements', authenticate, progressController.getUserAchievements);
router.get('/streak', authenticate, progressController.getLearningStreak);

// Analytics and insights
router.get('/analytics/dashboard', authenticate, progressController.getProgressDashboard);
router.get('/analytics/learning-path', authenticate, progressController.getLearningPathAnalytics);

module.exports = router;
```

**Progress Service Implementation:**
```javascript
// ✅ Good: Robust progress tracking service
const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();

class ProgressService {
  async markSectionComplete(userId, lessonId, sectionId) {
    return await prisma.$transaction(async (tx) => {
      // Check if section is already completed
      const existingProgress = await tx.sectionProgress.findUnique({
        where: {
          userId_lessonId_sectionId: {
            userId,
            lessonId,
            sectionId
          }
        }
      });

      if (existingProgress) {
        return existingProgress;
      }

      // Mark section as complete
      const sectionProgress = await tx.sectionProgress.create({
        data: {
          userId,
          lessonId,
          sectionId,
          completedAt: new Date(),
          timeSpent: 0 // Will be updated by time tracking
        }
      });

      // Update lesson progress
      await this.updateLessonProgress(tx, userId, lessonId);
      
      // Check for course completion
      await this.updateCourseProgress(tx, userId, lessonId);
      
      // Check for achievements
      await this.checkAndUnlockAchievements(tx, userId, lessonId, sectionId);

      return sectionProgress;
    });
  }

  async updateLessonProgress(tx, userId, lessonId) {
    // Get lesson details and section count
    const lesson = await tx.lesson.findUnique({
      where: { id: lessonId },
      include: { sections: true }
    });

    // Count completed sections
    const completedSections = await tx.sectionProgress.count({
      where: {
        userId,
        lessonId
      }
    });

    const completionPercentage = Math.round(
      (completedSections / lesson.sections.length) * 100
    );

    // Update or create lesson progress
    await tx.lessonProgress.upsert({
      where: {
        userId_lessonId: {
          userId,
          lessonId
        }
      },
      update: {
        completedSections,
        completionPercentage,
        isCompleted: completedSections === lesson.sections.length,
        updatedAt: new Date()
      },
      create: {
        userId,
        lessonId,
        completedSections,
        completionPercentage,
        isCompleted: completedSections === lesson.sections.length,
        startedAt: new Date()
      }
    });
  }

  async getCourseProgress(userId, courseId) {
    const courseProgress = await prisma.courseProgress.findUnique({
      where: {
        userId_courseId: {
          userId,
          courseId
        }
      },
      include: {
        course: {
          include: {
            lessons: {
              include: {
                progress: {
                  where: { userId }
                },
                sections: {
                  include: {
                    progress: {
                      where: { userId }
                    }
                  }
                }
              }
            }
          }
        }
      }
    });

    if (!courseProgress) {
      return null;
    }

    // Calculate detailed progress
    const lessonsProgress = courseProgress.course.lessons.map(lesson => {
      const lessonProg = lesson.progress[0];
      const sectionsProgress = lesson.sections.map(section => ({
        id: section.id,
        title: section.title,
        completed: section.progress.length > 0,
        completedAt: section.progress[0]?.completedAt || null
      }));

      return {
        id: lesson.id,
        title: lesson.title,
        completed: lessonProg?.isCompleted || false,
        completionPercentage: lessonProg?.completionPercentage || 0,
        sectionsProgress
      };
    });

    return {
      ...courseProgress,
      lessonsProgress,
      overallProgress: this.calculateOverallProgress(lessonsProgress)
    };
  }

  calculateOverallProgress(lessonsProgress) {
    if (lessonsProgress.length === 0) return 0;
    
    const totalPercentage = lessonsProgress.reduce(
      (sum, lesson) => sum + lesson.completionPercentage, 0
    );
    
    return Math.round(totalPercentage / lessonsProgress.length);
  }

  async checkAndUnlockAchievements(tx, userId, lessonId, sectionId) {
    // Get user's progress statistics
    const stats = await this.getUserProgressStats(tx, userId);
    
    const achievements = [];

    // First lesson completion
    if (stats.completedLessons === 1) {
      achievements.push({
        type: 'FIRST_LESSON',
        title: 'Getting Started',
        description: 'Completed your first lesson'
      });
    }

    // Learning streak achievements
    if (stats.learningStreak >= 7) {
      achievements.push({
        type: 'WEEK_STREAK',
        title: 'Week Warrior',
        description: '7-day learning streak'
      });
    }

    // Course completion achievements
    if (stats.completedCourses > 0 && stats.completedCourses % 5 === 0) {
      achievements.push({
        type: 'COURSE_MILESTONE',
        title: `${stats.completedCourses} Courses Completed`,
        description: `Completed ${stats.completedCourses} courses`
      });
    }

    // Unlock achievements
    for (const achievement of achievements) {
      await tx.userAchievement.upsert({
        where: {
          userId_type: {
            userId,
            type: achievement.type
          }
        },
        update: {},
        create: {
          userId,
          type: achievement.type,
          title: achievement.title,
          description: achievement.description,
          unlockedAt: new Date()
        }
      });
    }

    return achievements;
  }
}

module.exports = new ProgressService();
```

## Database Schema for Learning Platforms

### 1. Core Educational Data Models

**Prisma Schema for LMS:**
```prisma
// ✅ Good: Comprehensive LMS database schema
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String
  role      UserRole @default(STUDENT)
  avatar    String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Learning-related fields
  learningStreak    Int      @default(0)
  lastActiveDate    DateTime?
  totalTimeSpent    Int      @default(0) // in minutes
  skillPoints       Int      @default(0)

  // Relations
  createdCourses    Course[]           @relation("CourseCreator")
  enrollments       CourseEnrollment[]
  lessonProgress    LessonProgress[]
  sectionProgress   SectionProgress[]
  courseProgress    CourseProgress[]
  achievements      UserAchievement[]
  timeEntries       TimeEntry[]
  quizAttempts      QuizAttempt[]

  @@map("users")
}

model Course {
  id          String      @id @default(cuid())
  title       String
  description String
  difficulty  Difficulty  @default(INTERMEDIATE)
  category    String
  tags        String[]
  thumbnail   String?
  estimatedDuration Int   // in minutes
  
  // AI Generation fields
  generatedBy     String?
  generationPrompt String?
  aiMetadata      Json?
  
  // Status and visibility
  status      CourseStatus @default(DRAFT)
  isPublished Boolean      @default(false)
  
  // Timestamps
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  publishedAt DateTime?

  // Relations
  creator      User               @relation("CourseCreator", fields: [createdBy], references: [id])
  createdBy    String
  lessons      Lesson[]
  enrollments  CourseEnrollment[]
  progress     CourseProgress[]

  @@map("courses")
}

model Lesson {
  id          String @id @default(cuid())
  title       String
  description String
  order       Int
  estimatedDuration Int // in minutes
  
  // Content metadata
  contentType    ContentType @default(MIXED)
  learningObjectives String[]
  prerequisites  String[]
  
  // Timestamps
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  // Relations
  course      Course             @relation(fields: [courseId], references: [id], onDelete: Cascade)
  courseId    String
  sections    Section[]
  progress    LessonProgress[]
  quizzes     Quiz[]

  @@map("lessons")
}

model Section {
  id        String      @id @default(cuid())
  title     String
  content   String      // Rich text content
  type      SectionType @default(TEXT)
  order     Int
  
  // Media and interactive content
  videoUrl    String?
  imageUrl    String?
  codeSnippet String?
  quiz        Json?      // Inline quiz data
  
  // Timestamps
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  lesson    Lesson            @relation(fields: [lessonId], references: [id], onDelete: Cascade)
  lessonId  String
  progress  SectionProgress[]

  @@map("sections")
}

model CourseEnrollment {
  id         String   @id @default(cuid())
  enrolledAt DateTime @default(now())
  
  // Relations
  user     User   @relation(fields: [userId], references: [id])
  userId   String
  course   Course @relation(fields: [courseId], references: [id])
  courseId String

  @@unique([userId, courseId])
  @@map("course_enrollments")
}

model CourseProgress {
  id                 String   @id @default(cuid())
  completionPercentage Int    @default(0)
  completedLessons   Int      @default(0)
  totalLessons       Int      @default(0)
  isCompleted        Boolean  @default(false)
  startedAt          DateTime @default(now())
  completedAt        DateTime?
  lastAccessedAt     DateTime @default(now())

  // Relations
  user     User   @relation(fields: [userId], references: [id])
  userId   String
  course   Course @relation(fields: [courseId], references: [id])
  courseId String

  @@unique([userId, courseId])
  @@map("course_progress")
}

model LessonProgress {
  id                 String   @id @default(cuid())
  completedSections  Int      @default(0)
  totalSections      Int      @default(0)
  completionPercentage Int    @default(0)
  isCompleted        Boolean  @default(false)
  startedAt          DateTime @default(now())
  completedAt        DateTime?
  lastAccessedAt     DateTime @default(now())

  // Relations
  user     User   @relation(fields: [userId], references: [id])
  userId   String
  lesson   Lesson @relation(fields: [lessonId], references: [id])
  lessonId String

  @@unique([userId, lessonId])
  @@map("lesson_progress")
}

model SectionProgress {
  id          String    @id @default(cuid())
  completedAt DateTime  @default(now())
  timeSpent   Int       @default(0) // in seconds
  
  // Relations
  user      User    @relation(fields: [userId], references: [id])
  userId    String
  lesson    Lesson  @relation(fields: [lessonId], references: [id])
  lessonId  String
  section   Section @relation(fields: [sectionId], references: [id])
  sectionId String

  @@unique([userId, lessonId, sectionId])
  @@map("section_progress")
}

model TimeEntry {
  id        String   @id @default(cuid())
  duration  Int      // in seconds
  startTime DateTime
  endTime   DateTime
  activity  String   // e.g., "lesson", "quiz", "reading"
  
  // Context
  courseId  String?
  lessonId  String?
  sectionId String?
  
  // Relations
  user   User   @relation(fields: [userId], references: [id])
  userId String

  @@map("time_entries")
}

model UserAchievement {
  id          String   @id @default(cuid())
  type        String
  title       String
  description String
  points      Int      @default(0)
  unlockedAt  DateTime @default(now())

  // Relations
  user   User   @relation(fields: [userId], references: [id])
  userId String

  @@unique([userId, type])
  @@map("user_achievements")
}

model Quiz {
  id          String     @id @default(cuid())
  title       String
  description String?
  timeLimit   Int?       // in seconds
  questions   Question[]
  attempts    QuizAttempt[]
  
  // Relations
  lesson   Lesson @relation(fields: [lessonId], references: [id])
  lessonId String

  @@map("quizzes")
}

model Question {
  id           String       @id @default(cuid())
  type         QuestionType
  question     String
  options      String[]     // For multiple choice
  correctAnswer String
  explanation  String?
  points       Int          @default(1)
  order        Int

  // Relations
  quiz   Quiz   @relation(fields: [quizId], references: [id])
  quizId String

  @@map("questions")
}

model QuizAttempt {
  id          String   @id @default(cuid())
  answers     Json     // Stores user answers
  score       Int
  maxScore    Int
  completedAt DateTime @default(now())
  timeSpent   Int      // in seconds

  // Relations
  user   User @relation(fields: [userId], references: [id])
  userId String
  quiz   Quiz @relation(fields: [quizId], references: [id])
  quizId String

  @@map("quiz_attempts")
}

// Enums
enum UserRole {
  STUDENT
  INSTRUCTOR
  ADMIN
}

enum Difficulty {
  BEGINNER
  INTERMEDIATE
  ADVANCED
}

enum CourseStatus {
  DRAFT
  REVIEW
  PUBLISHED
  ARCHIVED
}

enum ContentType {
  TEXT
  VIDEO
  INTERACTIVE
  MIXED
}

enum SectionType {
  TEXT
  VIDEO
  QUIZ
  CODE
  IMAGE
  INTERACTIVE
}

enum QuestionType {
  MULTIPLE_CHOICE
  TRUE_FALSE
  SHORT_ANSWER
  ESSAY
  CODE
}
```

### 2. Database Optimization Strategies

**Indexing for Performance:**
```sql
-- ✅ Good: Strategic indexes for LMS queries
-- Course discovery and search
CREATE INDEX idx_courses_published_category ON courses (is_published, category);
CREATE INDEX idx_courses_difficulty_created ON courses (difficulty, created_at);
CREATE INDEX idx_courses_search ON courses USING gin (to_tsvector('english', title || ' ' || description));

-- Progress tracking optimization
CREATE INDEX idx_course_progress_user_completion ON course_progress (user_id, completion_percentage);
CREATE INDEX idx_lesson_progress_user_lesson ON lesson_progress (user_id, lesson_id, last_accessed_at);
CREATE INDEX idx_section_progress_user_completed ON section_progress (user_id, completed_at);

-- Time tracking and analytics
CREATE INDEX idx_time_entries_user_date ON time_entries (user_id, start_time);
CREATE INDEX idx_time_entries_course_duration ON time_entries (course_id, duration);

-- Achievement and gamification
CREATE INDEX idx_user_achievements_type_unlocked ON user_achievements (type, unlocked_at);
CREATE INDEX idx_users_streak_points ON users (learning_streak, skill_points);

-- Course enrollment and access patterns
CREATE INDEX idx_enrollments_user_enrolled ON course_enrollments (user_id, enrolled_at);
CREATE INDEX idx_enrollments_course_count ON course_enrollments (course_id, enrolled_at);
```

**Database Connection Management:**
```javascript
// ✅ Good: Production-ready database configuration
const { PrismaClient } = require('@prisma/client');

const prisma = new PrismaClient({
  log: process.env.NODE_ENV === 'development' ? ['query', 'info', 'warn', 'error'] : ['error'],
  datasources: {
    db: {
      url: process.env.DATABASE_URL,
    },
  },
});

// Connection pool configuration for high-load LMS
const databaseConfig = {
  connectionLimit: process.env.DB_CONNECTION_LIMIT || 20,
  acquireTimeoutMillis: 60000,
  createTimeoutMillis: 30000,
  destroyTimeoutMillis: 5000,
  idleTimeoutMillis: 30000,
  reapIntervalMillis: 1000,
  createRetryIntervalMillis: 200,
};

// Database health check
async function checkDatabaseHealth() {
  try {
    await prisma.$queryRaw`SELECT 1`;
    return { status: 'healthy', timestamp: new Date() };
  } catch (error) {
    console.error('Database health check failed:', error);
    return { status: 'unhealthy', error: error.message, timestamp: new Date() };
  }
}

// Graceful shutdown
process.on('SIGINT', async () => {
  console.log('Shutting down gracefully...');
  await prisma.$disconnect();
  process.exit(0);
});

module.exports = { prisma, checkDatabaseHealth };
```

## Course Generation & Content Management

### 1. AI-Powered Course Generation

**Course Generation Service:**
```javascript
// ✅ Good: Robust AI course generation service
const { OpenAI } = require('openai');
const { PrismaClient } = require('@prisma/client');

class CourseGenerationService {
  constructor() {
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
    this.prisma = new PrismaClient();
  }

  async generateCourse({ topic, difficulty, duration, requirements, createdBy }) {
    const generationJob = await this.createGenerationJob({
      topic, difficulty, duration, requirements, createdBy
    });

    // Start background processing
    this.processGenerationJob(generationJob.id);

    return generationJob;
  }

  async processGenerationJob(jobId) {
    try {
      const job = await this.prisma.courseGenerationJob.findUnique({
        where: { id: jobId }
      });

      if (!job) throw new Error('Generation job not found');

      // Update status to processing
      await this.updateJobStatus(jobId, 'PROCESSING');

      // Step 1: Generate course outline
      const courseOutline = await this.generateCourseOutline(job.parameters);
      
      // Step 2: Create course record
      const course = await this.createCourseFromOutline(courseOutline, job);
      
      // Step 3: Generate detailed lessons
      for (const lessonOutline of courseOutline.lessons) {
        const lesson = await this.generateDetailedLesson(course.id, lessonOutline);
        
        // Step 4: Generate sections for each lesson
        for (const sectionOutline of lessonOutline.sections) {
          await this.generateLessonSection(lesson.id, sectionOutline);
        }
      }

      // Step 5: Generate course thumbnail
      await this.generateCourseThumbnail(course.id, courseOutline);

      // Complete generation
      await this.updateJobStatus(jobId, 'COMPLETED', { courseId: course.id });
      
    } catch (error) {
      console.error('Course generation failed:', error);
      await this.updateJobStatus(jobId, 'FAILED', { error: error.message });
    }
  }

  async generateCourseOutline({ topic, difficulty, duration, requirements }) {
    const prompt = `
Create a comprehensive course outline for the topic: "${topic}"

Requirements:
- Difficulty level: ${difficulty}
- Estimated duration: ${duration} hours
- Additional requirements: ${requirements || 'None'}

Please provide a detailed course structure with:
1. Course title and description
2. Learning objectives
3. 6-8 lessons with titles and descriptions
4. 2-3 sections per lesson
5. Estimated time for each lesson

Format as JSON with this structure:
{
  "title": "Course Title",
  "description": "Course description",
  "objectives": ["objective1", "objective2"],
  "lessons": [
    {
      "title": "Lesson Title",
      "description": "Lesson description",
      "estimatedDuration": 30,
      "sections": [
        {
          "title": "Section Title",
          "type": "text|video|quiz|interactive",
          "description": "Section description"
        }
      ]
    }
  ]
}
    `;

    const response = await this.openai.chat.completions.create({
      model: "gpt-4",
      messages: [{ role: "user", content: prompt }],
      temperature: 0.7,
      max_tokens: 2000
    });

    return JSON.parse(response.choices[0].message.content);
  }

  async generateDetailedLesson(courseId, lessonOutline) {
    const lesson = await this.prisma.lesson.create({
      data: {
        courseId,
        title: lessonOutline.title,
        description: lessonOutline.description,
        estimatedDuration: lessonOutline.estimatedDuration,
        order: lessonOutline.order || 0
      }
    });

    return lesson;
  }

  async generateLessonSection(lessonId, sectionOutline) {
    let content = '';
    
    if (sectionOutline.type === 'text') {
      content = await this.generateTextContent(sectionOutline);
    } else if (sectionOutline.type === 'quiz') {
      content = await this.generateQuizContent(sectionOutline);
    } else if (sectionOutline.type === 'interactive') {
      content = await this.generateInteractiveContent(sectionOutline);
    }

    const section = await this.prisma.section.create({
      data: {
        lessonId,
        title: sectionOutline.title,
        content,
        type: sectionOutline.type.toUpperCase(),
        order: sectionOutline.order || 0
      }
    });

    return section;
  }

  async generateTextContent(sectionOutline) {
    const prompt = `
Create detailed educational content for the section: "${sectionOutline.title}"

Description: ${sectionOutline.description}

Please provide:
1. Clear explanations with examples
2. Key concepts and definitions
3. Practical applications
4. Important points to remember

Format as structured markdown with proper headings, lists, and emphasis.
Aim for 800-1200 words of comprehensive content.
    `;

    const response = await this.openai.chat.completions.create({
      model: "gpt-4",
      messages: [{ role: "user", content: prompt }],
      temperature: 0.6,
      max_tokens: 1500
    });

    return response.choices[0].message.content;
  }

  async generateQuizContent(sectionOutline) {
    const prompt = `
Create a quiz for the section: "${sectionOutline.title}"

Description: ${sectionOutline.description}

Generate 5-7 multiple choice questions that test understanding of the key concepts.

Format as JSON:
{
  "questions": [
    {
      "question": "Question text",
      "options": ["Option A", "Option B", "Option C", "Option D"],
      "correctAnswer": "Option A",
      "explanation": "Why this answer is correct"
    }
  ]
}
    `;

    const response = await this.openai.chat.completions.create({
      model: "gpt-4",
      messages: [{ role: "user", content: prompt }],
      temperature: 0.5,
      max_tokens: 1000
    });

    return response.choices[0].message.content;
  }

  async generateCourseThumbnail(courseId, courseOutline) {
    try {
      const prompt = `A modern, professional educational illustration for a course titled "${courseOutline.title}". Clean design, vibrant colors, educational theme, minimalist style, high quality`;

      const response = await this.openai.images.generate({
        model: "dall-e-3",
        prompt,
        size: "1024x1024",
        quality: "standard",
        n: 1,
      });

      const imageUrl = response.data[0].url;
      
      // In production, save the image to your storage service
      // const savedImageUrl = await this.saveImageToStorage(imageUrl, courseId);

      await this.prisma.course.update({
        where: { id: courseId },
        data: { thumbnail: imageUrl }
      });

      return imageUrl;
    } catch (error) {
      console.error('Failed to generate course thumbnail:', error);
      // Continue without thumbnail
    }
  }
}

module.exports = new CourseGenerationService();
```

### 2. Content Management and Versioning

**Content Version Control:**
```javascript
// ✅ Good: Content versioning for educational materials
class ContentVersionService {
  async createContentVersion(contentId, contentType, newContent, userId) {
    return await this.prisma.$transaction(async (tx) => {
      // Create version record
      const version = await tx.contentVersion.create({
        data: {
          contentId,
          contentType,
          content: newContent,
          version: await this.getNextVersionNumber(contentId),
          createdBy: userId,
          createdAt: new Date()
        }
      });

      // Update current content
      await this.updateCurrentContent(tx, contentId, contentType, newContent);

      return version;
    });
  }

  async getContentHistory(contentId) {
    return await this.prisma.contentVersion.findMany({
      where: { contentId },
      orderBy: { version: 'desc' },
      include: {
        creator: {
          select: { id: true, name: true, email: true }
        }
      }
    });
  }

  async revertToVersion(contentId, versionNumber, userId) {
    const version = await this.prisma.contentVersion.findFirst({
      where: { contentId, version: versionNumber }
    });

    if (!version) {
      throw new Error('Version not found');
    }

    // Create new version with reverted content
    return await this.createContentVersion(
      contentId,
      version.contentType,
      version.content,
      userId
    );
  }
}
```

This comprehensive guide provides the foundation for building a robust, scalable LMS backend with Node.js and Express, covering all critical aspects from API design to database optimization and AI integration. 