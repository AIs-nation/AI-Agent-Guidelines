# LMS Express.js Backend Architecture & API Patterns (2025)

## 🔄 Fundamental Principle: Constant Research & Web Updates
**Critical Success Factor**: This guide must be continuously updated with latest 2025+ web research on Express.js patterns for educational platforms. All team members must regularly research and update this knowledge base.

## Express.js Architecture for Educational Platforms

### 1. Educational API Design Patterns

#### ✅ Good Example: RESTful Educational API Structure
```typescript
// Good: Structured educational API with clear separation
// File: src/routes/courses.ts
import express from 'express';
import { authenticate, authorize } from '../middleware/auth';
import { validateCourseData } from '../middleware/validation';
import { trackAPIUsage } from '../middleware/analytics';
import { CourseController } from '../controllers/CourseController';

const router = express.Router();

// Educational-specific middleware stack
router.use(authenticate);
router.use(trackAPIUsage);

// Course management endpoints
router.get('/courses', 
  authorize(['student', 'instructor', 'admin']), 
  CourseController.getAllCourses
);

router.get('/courses/:courseId', 
  authorize(['student', 'instructor', 'admin']), 
  CourseController.getCourseById
);

router.post('/courses', 
  authorize(['instructor', 'admin']), 
  validateCourseData,
  CourseController.createCourse
);

router.put('/courses/:courseId', 
  authorize(['instructor', 'admin']), 
  validateCourseData,
  CourseController.updateCourse
);

// Learning progress endpoints
router.post('/courses/:courseId/enroll', 
  authorize(['student']), 
  CourseController.enrollStudent
);

router.get('/courses/:courseId/progress', 
  authorize(['student', 'instructor']), 
  CourseController.getProgress
);

router.post('/courses/:courseId/lessons/:lessonId/complete', 
  authorize(['student']), 
  CourseController.markLessonComplete
);

export default router;
```

#### ❌ Bad Example: Monolithic API Without Structure
```typescript
// Bad: No clear separation, mixed concerns
app.get('/api/everything', (req, res) => {
  // 200+ lines handling courses, users, progress, payments
  // No authentication, validation, or proper error handling
  // Violates single responsibility principle
});
```

### 2. Educational Data Models & Validation

#### ✅ Good Example: Comprehensive Educational Schema Validation
```typescript
// Good: Structured educational data validation
import Joi from 'joi';

// Course creation schema
export const courseSchema = Joi.object({
  title: Joi.string().min(5).max(100).required()
    .messages({
      'string.min': 'Course title must be at least 5 characters',
      'string.max': 'Course title cannot exceed 100 characters'
    }),
  
  description: Joi.string().min(20).max(2000).required()
    .messages({
      'string.min': 'Course description must be at least 20 characters'
    }),
  
  category: Joi.string().valid(
    'programming', 'design', 'business', 'language', 'science'
  ).required(),
  
  difficulty: Joi.string().valid('beginner', 'intermediate', 'advanced').required(),
  
  modules: Joi.array().items(
    Joi.object({
      title: Joi.string().min(5).max(50).required(),
      lessons: Joi.array().items(
        Joi.object({
          title: Joi.string().min(5).max(80).required(),
          type: Joi.string().valid('video', 'text', 'quiz', 'exercise').required(),
          duration: Joi.number().min(1).max(300).required(), // minutes
          content: Joi.object({
            videoUrl: Joi.when('type', {
              is: 'video',
              then: Joi.string().uri().required(),
              otherwise: Joi.optional()
            }),
            textContent: Joi.when('type', {
              is: 'text',
              then: Joi.string().min(100).required(),
              otherwise: Joi.optional()
            }),
            quiz: Joi.when('type', {
              is: 'quiz',
              then: Joi.object({
                questions: Joi.array().items(
                  Joi.object({
                    question: Joi.string().min(10).required(),
                    options: Joi.array().items(Joi.string()).min(2).max(6).required(),
                    correct: Joi.number().min(0).required(),
                    explanation: Joi.string().optional()
                  })
                ).min(1).required()
              }).required(),
              otherwise: Joi.optional()
            })
          }).required()
        })
      ).min(1).required()
    })
  ).min(1).required(),
  
  prerequisites: Joi.array().items(Joi.string().uuid()).optional(),
  price: Joi.number().min(0).max(10000).required(),
  tags: Joi.array().items(Joi.string().max(30)).max(10).optional()
});

// Enrollment validation
export const enrollmentSchema = Joi.object({
  courseId: Joi.string().uuid().required(),
  studentId: Joi.string().uuid().required(),
  enrollmentDate: Joi.date().iso().default(Date.now),
  paymentMethod: Joi.string().valid('credit_card', 'paypal', 'free').required()
});

// Progress tracking validation
export const progressSchema = Joi.object({
  studentId: Joi.string().uuid().required(),
  courseId: Joi.string().uuid().required(),
  lessonId: Joi.string().uuid().required(),
  completedAt: Joi.date().iso().required(),
  timeSpent: Joi.number().min(0).required(), // seconds
  score: Joi.number().min(0).max(100).optional(), // for quizzes
  notes: Joi.string().max(1000).optional()
});
```

#### ❌ Bad Example: No Validation or Weak Schema
```typescript
// Bad: No validation, accepts any data
app.post('/course', (req, res) => {
  const course = req.body; // No validation
  // Direct database insertion without checks
  db.courses.insert(course);
});
```

### 3. Authentication & Authorization for Educational Platforms

#### ✅ Good Example: Role-Based Educational Access Control
```typescript
// Good: Comprehensive educational authentication system
import jwt from 'jsonwebtoken';
import bcrypt from 'bcrypt';
import { Request, Response, NextFunction } from 'express';

// Educational user roles
export enum UserRole {
  STUDENT = 'student',
  INSTRUCTOR = 'instructor',
  ADMIN = 'admin',
  CONTENT_CREATOR = 'content_creator'
}

// Educational permissions matrix
const PERMISSIONS = {
  [UserRole.STUDENT]: [
    'course:read',
    'course:enroll',
    'lesson:read',
    'progress:create',
    'quiz:submit'
  ],
  [UserRole.INSTRUCTOR]: [
    'course:read',
    'course:create',
    'course:update_own',
    'lesson:create',
    'student:view_progress',
    'quiz:create'
  ],
  [UserRole.CONTENT_CREATOR]: [
    'course:read',
    'course:create',
    'lesson:create',
    'content:upload'
  ],
  [UserRole.ADMIN]: [
    'course:*',
    'user:*',
    'system:*'
  ]
};

// Authentication middleware
export const authenticate = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const token = req.headers.authorization?.replace('Bearer ', '');
    
    if (!token) {
      return res.status(401).json({
        error: 'Authentication required',
        code: 'AUTH_REQUIRED'
      });
    }
    
    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as any;
    const user = await User.findById(decoded.userId).select('-password');
    
    if (!user || !user.isActive) {
      return res.status(401).json({
        error: 'Invalid or inactive user',
        code: 'INVALID_USER'
      });
    }
    
    req.user = user;
    next();
  } catch (error) {
    return res.status(401).json({
      error: 'Invalid token',
      code: 'INVALID_TOKEN'
    });
  }
};

// Authorization middleware with educational context
export const authorize = (allowedRoles: UserRole[], permission?: string) => {
  return (req: Request, res: Response, next: NextFunction) => {
    const user = req.user;
    
    if (!user) {
      return res.status(401).json({
        error: 'Authentication required',
        code: 'AUTH_REQUIRED'
      });
    }
    
    // Check role authorization
    if (!allowedRoles.includes(user.role)) {
      return res.status(403).json({
        error: 'Insufficient permissions',
        code: 'INSUFFICIENT_PERMISSIONS',
        required: allowedRoles,
        current: user.role
      });
    }
    
    // Check specific permission if provided
    if (permission) {
      const userPermissions = PERMISSIONS[user.role] || [];
      const hasPermission = userPermissions.includes(permission) || 
                           userPermissions.includes(permission.split(':')[0] + ':*');
      
      if (!hasPermission) {
        return res.status(403).json({
          error: 'Permission denied',
          code: 'PERMISSION_DENIED',
          required: permission
        });
      }
    }
    
    next();
  };
};

// Course ownership middleware
export const checkCourseOwnership = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const { courseId } = req.params;
    const course = await Course.findById(courseId);
    
    if (!course) {
      return res.status(404).json({
        error: 'Course not found',
        code: 'COURSE_NOT_FOUND'
      });
    }
    
    // Admin can access any course
    if (req.user.role === UserRole.ADMIN) {
      return next();
    }
    
    // Instructor can only access their own courses
    if (req.user.role === UserRole.INSTRUCTOR && course.instructorId.toString() !== req.user.id) {
      return res.status(403).json({
        error: 'You can only modify your own courses',
        code: 'NOT_COURSE_OWNER'
      });
    }
    
    req.course = course;
    next();
  } catch (error) {
    return res.status(500).json({
      error: 'Error checking course ownership',
      code: 'OWNERSHIP_CHECK_ERROR'
    });
  }
};
```

#### ❌ Bad Example: Weak Authentication
```typescript
// Bad: No proper authentication or authorization
app.use((req, res, next) => {
  // Weak authentication
  if (req.headers.user) {
    req.user = { id: req.headers.user };
  }
  next();
});
```

### 4. Educational Data Processing & AI Integration

#### ✅ Good Example: AI-Powered Educational Content Generation
```typescript
// Good: Structured AI integration for educational content
import { Anthropic } from '@anthropic-ai/sdk';
import { OpenAI } from 'openai';

export class EducationalAIService {
  private claude: Anthropic;
  private openai: OpenAI;
  
  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY!
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY!
    });
  }
  
  async generateCourse(prompt: string, level: string): Promise<CourseStructure> {
    try {
      const coursePrompt = `
        Create a comprehensive ${level} level course on: ${prompt}
        
        Requirements:
        - 5-10 modules with clear learning objectives
        - Each module should have 3-7 lessons
        - Include diverse content types: videos, readings, quizzes, exercises
        - Provide learning assessments for each module
        - Ensure progressive difficulty
        - Include practical projects
        
        Format as structured JSON with:
        - Course metadata (title, description, prerequisites)
        - Detailed module breakdown
        - Learning outcomes
        - Assessment criteria
      `;
      
      const response = await this.claude.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 4000,
        messages: [{
          role: 'user',
          content: coursePrompt
        }]
      });
      
      const courseData = JSON.parse(response.content[0].text);
      
      // Validate generated content
      const { error, value } = courseSchema.validate(courseData);
      if (error) {
        throw new Error(`Invalid course structure: ${error.message}`);
      }
      
      return value;
    } catch (error) {
      console.error('Course generation error:', error);
      throw new Error('Failed to generate course content');
    }
  }
  
  async generateQuizQuestions(topic: string, difficulty: string, count: number): Promise<QuizQuestion[]> {
    try {
      const quizPrompt = `
        Generate ${count} ${difficulty} level quiz questions about: ${topic}
        
        Requirements:
        - Multiple choice with 4 options each
        - One correct answer per question
        - Include detailed explanations for correct answers
        - Avoid trick questions
        - Focus on understanding rather than memorization
        
        Format as JSON array with: question, options[], correct (index), explanation
      `;
      
      const response = await this.claude.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 2000,
        messages: [{
          role: 'user',
          content: quizPrompt
        }]
      });
      
      const questions = JSON.parse(response.content[0].text);
      
      // Validate each question
      questions.forEach((q: any, index: number) => {
        if (!q.question || !q.options || !Array.isArray(q.options) || 
            q.options.length !== 4 || typeof q.correct !== 'number') {
          throw new Error(`Invalid question format at index ${index}`);
        }
      });
      
      return questions;
    } catch (error) {
      console.error('Quiz generation error:', error);
      throw new Error('Failed to generate quiz questions');
    }
  }
  
  async generateVisualContent(description: string): Promise<string> {
    try {
      const response = await this.openai.images.generate({
        model: 'dall-e-3',
        prompt: `Educational illustration: ${description}. Style: clean, modern, suitable for learning materials, professional`,
        size: '1024x1024',
        quality: 'standard',
        n: 1
      });
      
      return response.data[0].url!;
    } catch (error) {
      console.error('Image generation error:', error);
      throw new Error('Failed to generate visual content');
    }
  }
}

// Course generation endpoint
export const generateCourseContent = async (req: Request, res: Response) => {
  try {
    const { topic, level, includeImages } = req.body;
    
    const aiService = new EducationalAIService();
    
    // Generate course structure
    const course = await aiService.generateCourse(topic, level);
    
    // Generate images for modules if requested
    if (includeImages) {
      for (const module of course.modules) {
        try {
          module.imageUrl = await aiService.generateVisualContent(
            `${module.title} educational concept illustration`
          );
        } catch (error) {
          console.warn(`Failed to generate image for module: ${module.title}`);
        }
      }
    }
    
    // Save to database
    const savedCourse = await Course.create({
      ...course,
      instructorId: req.user.id,
      status: 'draft',
      createdAt: new Date()
    });
    
    // Track course generation analytics
    await Analytics.track('course_generated', {
      instructorId: req.user.id,
      courseId: savedCourse.id,
      topic,
      level,
      moduleCount: course.modules.length
    });
    
    res.status(201).json({
      success: true,
      course: savedCourse,
      message: 'Course generated successfully'
    });
    
  } catch (error) {
    console.error('Course generation error:', error);
    res.status(500).json({
      error: 'Failed to generate course',
      code: 'COURSE_GENERATION_ERROR'
    });
  }
};
```

#### ❌ Bad Example: Unstructured AI Integration
```typescript
// Bad: No error handling, validation, or structure
app.post('/generate', async (req, res) => {
  const result = await fetch('https://api.openai.com/v1/completions', {
    method: 'POST',
    body: JSON.stringify({ prompt: req.body.prompt })
  });
  res.json(result);
});
```

### 5. Educational Database Operations & Optimization

#### ✅ Good Example: Optimized Educational Queries
```typescript
// Good: Efficient database operations for educational data
import { PrismaClient } from '@prisma/client';

export class EducationalRepository {
  private prisma: PrismaClient;
  
  constructor() {
    this.prisma = new PrismaClient({
      log: ['query', 'info', 'warn', 'error']
    });
  }
  
  // Optimized course listing with pagination and filtering
  async getCourses(params: {
    page: number;
    limit: number;
    category?: string;
    difficulty?: string;
    search?: string;
    userId?: string;
  }): Promise<{ courses: Course[]; total: number; hasMore: boolean }> {
    const { page, limit, category, difficulty, search, userId } = params;
    const offset = (page - 1) * limit;
    
    const where: any = {
      status: 'published'
    };
    
    if (category) {
      where.category = category;
    }
    
    if (difficulty) {
      where.difficulty = difficulty;
    }
    
    if (search) {
      where.OR = [
        { title: { contains: search, mode: 'insensitive' } },
        { description: { contains: search, mode: 'insensitive' } },
        { tags: { hasSome: [search] } }
      ];
    }
    
    // Parallel queries for performance
    const [courses, total] = await Promise.all([
      this.prisma.course.findMany({
        where,
        select: {
          id: true,
          title: true,
          description: true,
          category: true,
          difficulty: true,
          price: true,
          rating: true,
          studentCount: true,
          thumbnailUrl: true,
          instructor: {
            select: {
              id: true,
              name: true,
              avatar: true
            }
          },
          // Include enrollment status if user provided
          ...(userId && {
            enrollments: {
              where: { studentId: userId },
              select: { id: true, enrolledAt: true }
            }
          })
        },
        orderBy: [
          { featured: 'desc' },
          { rating: 'desc' },
          { createdAt: 'desc' }
        ],
        skip: offset,
        take: limit
      }),
      this.prisma.course.count({ where })
    ]);
    
    return {
      courses,
      total,
      hasMore: total > offset + limit
    };
  }
  
  // Efficient progress tracking
  async updateLearningProgress(params: {
    studentId: string;
    courseId: string;
    lessonId: string;
    timeSpent: number;
    completedAt: Date;
    score?: number;
  }): Promise<void> {
    const { studentId, courseId, lessonId, timeSpent, completedAt, score } = params;
    
    await this.prisma.$transaction(async (tx) => {
      // Update or create lesson progress
      await tx.lessonProgress.upsert({
        where: {
          studentId_lessonId: {
            studentId,
            lessonId
          }
        },
        update: {
          completedAt,
          timeSpent: { increment: timeSpent },
          score: score ?? undefined
        },
        create: {
          studentId,
          lessonId,
          courseId,
          completedAt,
          timeSpent,
          score
        }
      });
      
      // Calculate and update course progress
      const courseProgress = await tx.lessonProgress.aggregate({
        where: {
          studentId,
          courseId
        },
        _count: {
          id: true
        }
      });
      
      const totalLessons = await tx.lesson.count({
        where: { courseId }
      });
      
      const progressPercentage = Math.round((courseProgress._count.id / totalLessons) * 100);
      
      // Update enrollment progress
      await tx.enrollment.update({
        where: {
          studentId_courseId: {
            studentId,
            courseId
          }
        },
        data: {
          progress: progressPercentage,
          lastAccessedAt: new Date()
        }
      });
      
      // Update course completion if 100%
      if (progressPercentage === 100) {
        await tx.enrollment.update({
          where: {
            studentId_courseId: {
              studentId,
              courseId
            }
          },
          data: {
            completedAt: new Date(),
            certificateIssued: true
          }
        });
      }
    });
  }
  
  // Analytics query for educational insights
  async getEducationalAnalytics(instructorId: string, timeRange: string): Promise<any> {
    const dateFilter = this.getDateFilter(timeRange);
    
    const [
      enrollmentStats,
      completionStats,
      engagementStats,
      revenueStats
    ] = await Promise.all([
      // Enrollment analytics
      this.prisma.enrollment.groupBy({
        by: ['courseId'],
        where: {
          course: { instructorId },
          enrolledAt: dateFilter
        },
        _count: { id: true }
      }),
      
      // Completion analytics
      this.prisma.enrollment.groupBy({
        by: ['courseId'],
        where: {
          course: { instructorId },
          completedAt: { not: null }
        },
        _count: { id: true }
      }),
      
      // Engagement analytics
      this.prisma.lessonProgress.groupBy({
        by: ['courseId'],
        where: {
          course: { instructorId },
          completedAt: dateFilter
        },
        _sum: { timeSpent: true },
        _avg: { timeSpent: true }
      }),
      
      // Revenue analytics
      this.prisma.enrollment.groupBy({
        by: ['courseId'],
        where: {
          course: { instructorId },
          enrolledAt: dateFilter
        },
        _sum: { amountPaid: true }
      })
    ]);
    
    return {
      enrollments: enrollmentStats,
      completions: completionStats,
      engagement: engagementStats,
      revenue: revenueStats
    };
  }
  
  private getDateFilter(timeRange: string) {
    const now = new Date();
    switch (timeRange) {
      case 'week':
        return { gte: new Date(now.getTime() - 7 * 24 * 60 * 60 * 1000) };
      case 'month':
        return { gte: new Date(now.getTime() - 30 * 24 * 60 * 60 * 1000) };
      case 'year':
        return { gte: new Date(now.getTime() - 365 * 24 * 60 * 60 * 1000) };
      default:
        return { gte: new Date(now.getTime() - 30 * 24 * 60 * 60 * 1000) };
    }
  }
}
```

#### ❌ Bad Example: Inefficient Database Operations
```typescript
// Bad: N+1 queries, no optimization
app.get('/courses', async (req, res) => {
  const courses = await Course.findAll();
  for (const course of courses) {
    course.instructor = await User.findById(course.instructorId);
    course.enrollments = await Enrollment.findAll({ courseId: course.id });
  }
  res.json(courses);
});
```

### 6. Error Handling & Educational Context

#### ✅ Good Example: Educational Error Handling
```typescript
// Good: Comprehensive educational error handling
export class EducationalError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number,
    public context?: any
  ) {
    super(message);
    this.name = 'EducationalError';
  }
}

// Educational error types
export const ErrorCodes = {
  // Course errors
  COURSE_NOT_FOUND: 'COURSE_NOT_FOUND',
  COURSE_NOT_PUBLISHED: 'COURSE_NOT_PUBLISHED',
  COURSE_ENROLLMENT_CLOSED: 'COURSE_ENROLLMENT_CLOSED',
  
  // Enrollment errors
  ALREADY_ENROLLED: 'ALREADY_ENROLLED',
  PREREQUISITES_NOT_MET: 'PREREQUISITES_NOT_MET',
  PAYMENT_REQUIRED: 'PAYMENT_REQUIRED',
  
  // Progress errors
  LESSON_NOT_ACCESSIBLE: 'LESSON_NOT_ACCESSIBLE',
  QUIZ_ALREADY_COMPLETED: 'QUIZ_ALREADY_COMPLETED',
  INVALID_PROGRESS_DATA: 'INVALID_PROGRESS_DATA'
};

// Global error handler for educational APIs
export const educationalErrorHandler = (
  error: Error,
  req: Request,
  res: Response,
  next: NextFunction
) => {
  console.error('Educational API Error:', {
    error: error.message,
    stack: error.stack,
    url: req.url,
    method: req.method,
    user: req.user?.id,
    timestamp: new Date().toISOString()
  });
  
  if (error instanceof EducationalError) {
    return res.status(error.statusCode).json({
      error: error.message,
      code: error.code,
      context: error.context,
      timestamp: new Date().toISOString()
    });
  }
  
  // Validation errors
  if (error.name === 'ValidationError') {
    return res.status(400).json({
      error: 'Invalid input data',
      code: 'VALIDATION_ERROR',
      details: error.message,
      timestamp: new Date().toISOString()
    });
  }
  
  // Database errors
  if (error.name === 'PrismaClientKnownRequestError') {
    return res.status(400).json({
      error: 'Database operation failed',
      code: 'DATABASE_ERROR',
      timestamp: new Date().toISOString()
    });
  }
  
  // Default error
  return res.status(500).json({
    error: 'Internal server error',
    code: 'INTERNAL_ERROR',
    timestamp: new Date().toISOString()
  });
};

// Educational business logic errors
export const validateEnrollment = async (courseId: string, studentId: string) => {
  const course = await Course.findById(courseId);
  if (!course) {
    throw new EducationalError(
      'Course not found',
      ErrorCodes.COURSE_NOT_FOUND,
      404,
      { courseId }
    );
  }
  
  if (course.status !== 'published') {
    throw new EducationalError(
      'Course is not available for enrollment',
      ErrorCodes.COURSE_NOT_PUBLISHED,
      400,
      { courseId, status: course.status }
    );
  }
  
  const existingEnrollment = await Enrollment.findOne({ courseId, studentId });
  if (existingEnrollment) {
    throw new EducationalError(
      'Student is already enrolled in this course',
      ErrorCodes.ALREADY_ENROLLED,
      400,
      { courseId, studentId }
    );
  }
  
  // Check prerequisites
  if (course.prerequisites?.length > 0) {
    const completedCourses = await Enrollment.find({
      studentId,
      courseId: { $in: course.prerequisites },
      completedAt: { $ne: null }
    });
    
    if (completedCourses.length < course.prerequisites.length) {
      throw new EducationalError(
        'Prerequisites not met for this course',
        ErrorCodes.PREREQUISITES_NOT_MET,
        400,
        {
          required: course.prerequisites,
          completed: completedCourses.map(e => e.courseId)
        }
      );
    }
  }
};
```

#### ❌ Bad Example: Poor Error Handling
```typescript
// Bad: Generic error handling without context
app.use((error, req, res, next) => {
  res.status(500).json({ error: 'Something went wrong' });
});
```

## Express.js LMS Patterns Summary

### ✅ DO's for Educational Backend Development:
- ✅ Implement structured RESTful APIs for educational resources
- ✅ Use comprehensive authentication and role-based authorization
- ✅ Validate all educational data with detailed schemas
- ✅ Optimize database queries for educational analytics
- ✅ Integrate AI services for content generation
- ✅ Implement proper error handling with educational context
- ✅ Track learning progress and engagement metrics
- ✅ Use middleware for cross-cutting educational concerns
- ✅ Implement caching for frequently accessed educational content
- ✅ Follow security best practices for educational data protection

### ❌ DON'Ts for Educational Backend Development:
- ❌ Create monolithic endpoints without separation of concerns
- ❌ Skip authentication and authorization for educational resources
- ❌ Allow unvalidated data into educational systems
- ❌ Use inefficient database queries for educational analytics
- ❌ Integrate AI services without proper error handling
- ❌ Ignore educational business logic in error messages
- ❌ Skip tracking of educational interactions and progress
- ❌ Mix authentication logic with business logic
- ❌ Expose sensitive educational data without proper security
- ❌ Use generic patterns without educational context

## Implementation Guidelines for LMS Backend

### API Response Standards
```typescript
// Consistent educational API responses
interface EducationalAPIResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
  code?: string;
  pagination?: {
    page: number;
    limit: number;
    total: number;
    hasMore: boolean;
  };
  meta?: {
    timestamp: string;
    version: string;
    requestId: string;
  };
}

// Success response helper
export const successResponse = <T>(data: T, pagination?: any): EducationalAPIResponse<T> => ({
  success: true,
  data,
  pagination,
  meta: {
    timestamp: new Date().toISOString(),
    version: '1.0',
    requestId: generateRequestId()
  }
});

// Error response helper
export const errorResponse = (error: string, code: string): EducationalAPIResponse<null> => ({
  success: false,
  error,
  code,
  meta: {
    timestamp: new Date().toISOString(),
    version: '1.0',
    requestId: generateRequestId()
  }
});
```

Remember: Keep this guide updated with latest Express.js patterns and educational platform requirements. Research and validate these patterns against current 2025+ Node.js best practices and LMS development standards. 