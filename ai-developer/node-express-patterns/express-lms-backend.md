# Node.js Express Backend Patterns for LMS Development

## Express Application Architecture for Educational Platforms

### LMS-Specific Express Application Structure
✅ **Good Example: Scalable Express LMS Architecture**
```javascript
// app.js - Main application setup for LMS backend
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import rateLimit from 'express-rate-limit';
import { PrismaClient } from '@prisma/client';
import { createServer } from 'http';

// Route imports
import courseRoutes from './routes/courseRoutes.js';
import progressRoutes from './routes/progressRoutes.js';
import aiRoutes from './routes/aiRoutes.js';
import authRoutes from './routes/authRoutes.js';
import analyticsRoutes from './routes/analyticsRoutes.js';

// Middleware imports
import { errorHandler } from './middleware/errorHandler.js';
import { logger } from './middleware/logger.js';
import { apiKeyAuth } from './middleware/auth.js';
import { validateRequest } from './middleware/validation.js';

const app = express();
const prisma = new PrismaClient();

// Security middleware
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'", "https://api.anthropic.com"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:", "blob:"],
      connectSrc: ["'self'", "https://api.anthropic.com", "https://api.openai.com"]
    }
  }
}));

// CORS configuration for LMS frontend
app.use(cors({
  origin: process.env.NODE_ENV === 'production' 
    ? process.env.FRONTEND_URL 
    : ['http://localhost:3000', 'http://localhost:5173'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'x-api-key']
}));

// Rate limiting for API protection
const createRateLimit = (windowMs, max, message) => rateLimit({
  windowMs,
  max,
  message: { error: message },
  standardHeaders: true,
  legacyHeaders: false,
});

// Different rate limits for different endpoints
app.use('/api/courses/generate', createRateLimit(15 * 60 * 1000, 10, 'Too many course generation attempts'));
app.use('/api/', createRateLimit(15 * 60 * 1000, 1000, 'Too many API requests'));

// Body parsing middleware
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true, limit: '10mb' }));

// Custom middleware for LMS
app.use(logger);
app.use('/api/', apiKeyAuth);

// Health check endpoint
app.get('/health', (req, res) => {
  res.json({ 
    status: 'healthy', 
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    environment: process.env.NODE_ENV 
  });
});

// API routes with proper versioning
app.use('/api/v1/courses', courseRoutes);
app.use('/api/v1/progress', progressRoutes);
app.use('/api/v1/ai', aiRoutes);
app.use('/api/v1/auth', authRoutes);
app.use('/api/v1/analytics', analyticsRoutes);

// Legacy routes (for backward compatibility)
app.use('/api/courses', courseRoutes);
app.use('/api/progress', progressRoutes);

// Error handling middleware (must be last)
app.use(errorHandler);

// Graceful shutdown handling
const server = createServer(app);

const gracefulShutdown = async () => {
  console.log('Received shutdown signal, closing server gracefully...');
  
  server.close(async () => {
    console.log('HTTP server closed');
    
    try {
      await prisma.$disconnect();
      console.log('Database connection closed');
      process.exit(0);
    } catch (error) {
      console.error('Error during shutdown:', error);
      process.exit(1);
    }
  });
};

process.on('SIGTERM', gracefulShutdown);
process.on('SIGINT', gracefulShutdown);

export { app, server, prisma };
```

## Course Management API Patterns

### RESTful Course API Implementation
✅ **Good Example: Comprehensive Course Routes**
```javascript
// routes/courseRoutes.js - LMS course management
import express from 'express';
import { body, param, query, validationResult } from 'express-validator';
import { prisma } from '../app.js';
import { generateCourseWithAI } from '../services/aiService.js';
import { validateRequest } from '../middleware/validation.js';
import { asyncHandler } from '../utils/asyncHandler.js';

const router = express.Router();

// Validation schemas for course operations
const courseValidation = {
  create: [
    body('title').isLength({ min: 3, max: 200 }).trim().escape(),
    body('description').isLength({ min: 10, max: 1000 }).trim().escape(),
    body('difficulty').isIn(['beginner', 'intermediate', 'advanced']),
    body('tags').optional().isArray().custom((tags) => {
      return tags.every(tag => typeof tag === 'string' && tag.length <= 50);
    }),
  ],
  
  courseId: [
    param('courseId').isUUID().withMessage('Invalid course ID format')
  ],
  
  listQuery: [
    query('page').optional().isInt({ min: 1 }).toInt(),
    query('limit').optional().isInt({ min: 1, max: 100 }).toInt(),
    query('difficulty').optional().isIn(['beginner', 'intermediate', 'advanced']),
    query('search').optional().isLength({ max: 100 }).trim().escape(),
  ]
};

// GET /api/courses - List courses with filtering and pagination
router.get('/', 
  courseValidation.listQuery,
  validateRequest,
  asyncHandler(async (req, res) => {
    const { page = 1, limit = 20, difficulty, search } = req.query;
    const skip = (page - 1) * limit;

    // Build dynamic where clause
    const where = {};
    if (difficulty) where.difficulty = difficulty;
    if (search) {
      where.OR = [
        { title: { contains: search, mode: 'insensitive' } },
        { description: { contains: search, mode: 'insensitive' } }
      ];
    }

    // Execute queries in parallel
    const [courses, totalCount] = await Promise.all([
      prisma.course.findMany({
        where,
        skip,
        take: limit,
        include: {
          lessons: {
            select: { id: true, title: true, sections: { select: { id: true } } }
          },
          _count: { select: { enrollments: true } }
        },
        orderBy: { createdAt: 'desc' }
      }),
      prisma.course.count({ where })
    ]);

    // Transform data for frontend consumption
    const transformedCourses = courses.map(course => ({
      id: course.id,
      title: course.title,
      description: course.description,
      difficulty: course.difficulty,
      imageUrl: course.imageUrl,
      lessonCount: course.lessons.length,
      sectionCount: course.lessons.reduce((total, lesson) => total + lesson.sections.length, 0),
      enrollmentCount: course._count.enrollments,
      estimatedHours: course.estimatedHours,
      tags: course.tags,
      createdAt: course.createdAt,
      updatedAt: course.updatedAt
    }));

    res.json({
      courses: transformedCourses,
      pagination: {
        page,
        limit,
        totalCount,
        totalPages: Math.ceil(totalCount / limit),
        hasNextPage: page * limit < totalCount,
        hasPreviousPage: page > 1
      },
      filters: { difficulty, search }
    });
  })
);

// GET /api/courses/:courseId - Get detailed course information
router.get('/:courseId',
  courseValidation.courseId,
  validateRequest,
  asyncHandler(async (req, res) => {
    const { courseId } = req.params;
    const { includeProgress } = req.query;
    const userId = req.user?.id; // From auth middleware

    const course = await prisma.course.findUnique({
      where: { id: courseId },
      include: {
        lessons: {
          include: {
            sections: {
              select: {
                id: true,
                title: true,
                type: true,
                order: true,
                estimatedMinutes: true
              },
              orderBy: { order: 'asc' }
            }
          },
          orderBy: { order: 'asc' }
        },
        instructor: {
          select: { id: true, name: true, email: true, bio: true }
        }
      }
    });

    if (!course) {
      return res.status(404).json({ 
        error: 'Course not found',
        code: 'COURSE_NOT_FOUND' 
      });
    }

    // Get user progress if requested and user is authenticated
    let userProgress = null;
    if (includeProgress && userId) {
      userProgress = await prisma.userProgress.findUnique({
        where: {
          userId_courseId: { userId, courseId }
        },
        include: {
          lessonProgress: {
            include: {
              sectionProgress: true
            }
          }
        }
      });
    }

    // Transform course data for frontend
    const transformedCourse = {
      id: course.id,
      title: course.title,
      description: course.description,
      difficulty: course.difficulty,
      imageUrl: course.imageUrl,
      estimatedHours: course.estimatedHours,
      tags: course.tags,
      instructor: course.instructor,
      createdAt: course.createdAt,
      updatedAt: course.updatedAt,
      lessons: course.lessons.map(lesson => ({
        id: lesson.id,
        title: lesson.title,
        description: lesson.description,
        order: lesson.order,
        estimatedMinutes: lesson.estimatedMinutes,
        sections: lesson.sections.map(section => ({
          id: section.id,
          title: section.title,
          type: section.type,
          order: section.order,
          estimatedMinutes: section.estimatedMinutes
        }))
      })),
      progress: userProgress ? transformProgressData(userProgress) : null
    };

    // Cache response for performance
    res.set('Cache-Control', 'public, max-age=300'); // 5 minutes
    res.json(transformedCourse);
  })
);

// POST /api/courses/generate-stream - AI-powered course generation with SSE
router.post('/generate-stream',
  courseValidation.create,
  validateRequest,
  asyncHandler(async (req, res) => {
    const { title, description, difficulty = 'intermediate', tags = [] } = req.body;

    // Set up Server-Sent Events
    res.writeHead(200, {
      'Content-Type': 'text/event-stream',
      'Connection': 'keep-alive',
      'Cache-Control': 'no-cache',
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Headers': 'Cache-Control'
    });

    const sendEvent = (event, data) => {
      res.write(`event: ${event}\n`);
      res.write(`data: ${JSON.stringify(data)}\n\n`);
    };

    try {
      sendEvent('progress', { stage: 'starting', message: 'Initializing course generation...' });

      // Generate course using AI service with progress callbacks
      const course = await generateCourseWithAI({
        title,
        description,
        difficulty,
        tags
      }, {
        onProgress: (stage, progress, message) => {
          sendEvent('progress', { stage, progress, message });
        }
      });

      sendEvent('complete', { 
        message: 'Course generated successfully!',
        courseId: course.id,
        course: {
          id: course.id,
          title: course.title,
          lessonCount: course.lessons?.length || 0
        }
      });

    } catch (error) {
      console.error('Course generation error:', error);
      sendEvent('error', { 
        message: 'Failed to generate course',
        error: process.env.NODE_ENV === 'production' ? 'Internal server error' : error.message
      });
    } finally {
      res.end();
    }
  })
);

// PUT /api/courses/:courseId - Update course
router.put('/:courseId',
  courseValidation.courseId,
  courseValidation.create,
  validateRequest,
  asyncHandler(async (req, res) => {
    const { courseId } = req.params;
    const { title, description, difficulty, tags, imageUrl } = req.body;

    const updatedCourse = await prisma.course.update({
      where: { id: courseId },
      data: {
        title,
        description,
        difficulty,
        tags,
        imageUrl,
        updatedAt: new Date()
      },
      include: {
        lessons: {
          select: { id: true }
        }
      }
    });

    res.json({
      message: 'Course updated successfully',
      course: updatedCourse
    });
  })
);

// DELETE /api/courses/:courseId - Delete course (soft delete)
router.delete('/:courseId',
  courseValidation.courseId,
  validateRequest,
  asyncHandler(async (req, res) => {
    const { courseId } = req.params;

    // Soft delete by updating status
    await prisma.course.update({
      where: { id: courseId },
      data: { 
        status: 'deleted',
        deletedAt: new Date()
      }
    });

    res.json({ 
      message: 'Course deleted successfully',
      courseId 
    });
  })
);

// Helper function to transform progress data
const transformProgressData = (userProgress) => {
  const completedLessons = userProgress.lessonProgress.filter(lp => 
    lp.sectionProgress.every(sp => sp.completed)
  ).length;

  const totalSections = userProgress.lessonProgress.reduce((total, lp) => 
    total + lp.sectionProgress.length, 0
  );

  const completedSections = userProgress.lessonProgress.reduce((total, lp) => 
    total + lp.sectionProgress.filter(sp => sp.completed).length, 0
  );

  return {
    completedLessons,
    totalLessons: userProgress.lessonProgress.length,
    completedSections,
    totalSections,
    completionPercentage: totalSections > 0 ? Math.round((completedSections / totalSections) * 100) : 0,
    lastAccessedAt: userProgress.lastAccessedAt,
    totalTimeSpent: userProgress.totalTimeSpent
  };
};

export default router;
```

## Progress Tracking API Patterns

### Advanced Progress Management
✅ **Good Example: Comprehensive Progress Tracking**
```javascript
// routes/progressRoutes.js - Learning progress management
import express from 'express';
import { body, param, query } from 'express-validator';
import { prisma } from '../app.js';
import { validateRequest } from '../middleware/validation.js';
import { asyncHandler } from '../utils/asyncHandler.js';
import { calculateLearningAnalytics } from '../services/analyticsService.js';

const router = express.Router();

// Progress validation schemas
const progressValidation = {
  updateSection: [
    body('courseId').isUUID(),
    body('lessonId').isUUID(),
    body('sectionId').isUUID(),
    body('completed').isBoolean(),
    body('timeSpent').isInt({ min: 0 }),
    body('score').optional().isFloat({ min: 0, max: 100 })
  ],

  getUserProgress: [
    param('userId').optional().isUUID(),
    query('courseId').optional().isUUID()
  ]
};

// POST /api/progress/section - Update section progress
router.post('/section',
  progressValidation.updateSection,
  validateRequest,
  asyncHandler(async (req, res) => {
    const { courseId, lessonId, sectionId, completed, timeSpent, score } = req.body;
    const userId = req.user.id; // From auth middleware

    // Use transaction for data consistency
    const result = await prisma.$transaction(async (tx) => {
      // Upsert user course progress
      const userProgress = await tx.userProgress.upsert({
        where: {
          userId_courseId: { userId, courseId }
        },
        create: {
          userId,
          courseId,
          totalTimeSpent: timeSpent,
          lastAccessedAt: new Date()
        },
        update: {
          totalTimeSpent: { increment: timeSpent },
          lastAccessedAt: new Date()
        }
      });

      // Upsert lesson progress
      const lessonProgress = await tx.lessonProgress.upsert({
        where: {
          userProgressId_lessonId: {
            userProgressId: userProgress.id,
            lessonId
          }
        },
        create: {
          userProgressId: userProgress.id,
          lessonId,
          timeSpent: timeSpent,
          lastAccessedAt: new Date()
        },
        update: {
          timeSpent: { increment: timeSpent },
          lastAccessedAt: new Date()
        }
      });

      // Upsert section progress
      const sectionProgress = await tx.sectionProgress.upsert({
        where: {
          lessonProgressId_sectionId: {
            lessonProgressId: lessonProgress.id,
            sectionId
          }
        },
        create: {
          lessonProgressId: lessonProgress.id,
          sectionId,
          completed,
          timeSpent,
          score,
          completedAt: completed ? new Date() : null
        },
        update: {
          completed,
          timeSpent: { increment: timeSpent },
          score: score !== undefined ? score : undefined,
          completedAt: completed ? new Date() : null
        }
      });

      // Calculate updated lesson and course completion status
      const allSectionProgress = await tx.sectionProgress.findMany({
        where: { lessonProgressId: lessonProgress.id }
      });

      const lessonCompleted = allSectionProgress.every(sp => sp.completed);
      
      if (lessonCompleted) {
        await tx.lessonProgress.update({
          where: { id: lessonProgress.id },
          data: { 
            completed: true,
            completedAt: new Date()
          }
        });
      }

      // Check if course is completed
      const allLessonProgress = await tx.lessonProgress.findMany({
        where: { userProgressId: userProgress.id }
      });

      const courseCompleted = allLessonProgress.every(lp => lp.completed);
      
      if (courseCompleted) {
        await tx.userProgress.update({
          where: { id: userProgress.id },
          data: { 
            completed: true,
            completedAt: new Date()
          }
        });
      }

      return { userProgress, lessonProgress, sectionProgress, courseCompleted, lessonCompleted };
    });

    // Generate analytics event
    await calculateLearningAnalytics(userId, courseId, {
      eventType: completed ? 'section_completed' : 'section_progress',
      lessonId,
      sectionId,
      timeSpent,
      score
    });

    res.json({
      success: true,
      message: 'Progress updated successfully',
      progress: {
        sectionCompleted: completed,
        lessonCompleted: result.lessonCompleted,
        courseCompleted: result.courseCompleted,
        timeSpent: result.sectionProgress.timeSpent,
        totalTimeSpent: result.userProgress.totalTimeSpent
      }
    });
  })
);

// GET /api/progress/:userId? - Get user progress (current user if no userId)
router.get('/:userId?',
  progressValidation.getUserProgress,
  validateRequest,
  asyncHandler(async (req, res) => {
    const targetUserId = req.params.userId || req.user.id;
    const { courseId } = req.query;

    // Build where clause
    const where = { userId: targetUserId };
    if (courseId) where.courseId = courseId;

    const userProgress = await prisma.userProgress.findMany({
      where,
      include: {
        course: {
          select: {
            id: true,
            title: true,
            difficulty: true,
            imageUrl: true
          }
        },
        lessonProgress: {
          include: {
            lesson: {
              select: {
                id: true,
                title: true,
                order: true
              }
            },
            sectionProgress: {
              include: {
                section: {
                  select: {
                    id: true,
                    title: true,
                    type: true,
                    order: true
                  }
                }
              },
              orderBy: { section: { order: 'asc' } }
            }
          },
          orderBy: { lesson: { order: 'asc' } }
        }
      },
      orderBy: { lastAccessedAt: 'desc' }
    });

    // Transform data for frontend consumption
    const transformedProgress = userProgress.map(up => ({
      courseId: up.courseId,
      course: up.course,
      completed: up.completed,
      completedAt: up.completedAt,
      totalTimeSpent: up.totalTimeSpent,
      lastAccessedAt: up.lastAccessedAt,
      lessons: up.lessonProgress.map(lp => ({
        lessonId: lp.lessonId,
        lesson: lp.lesson,
        completed: lp.completed,
        completedAt: lp.completedAt,
        timeSpent: lp.timeSpent,
        sections: lp.sectionProgress.map(sp => ({
          sectionId: sp.sectionId,
          section: sp.section,
          completed: sp.completed,
          completedAt: sp.completedAt,
          timeSpent: sp.timeSpent,
          score: sp.score
        }))
      })),
      statistics: {
        totalLessons: up.lessonProgress.length,
        completedLessons: up.lessonProgress.filter(lp => lp.completed).length,
        totalSections: up.lessonProgress.reduce((total, lp) => total + lp.sectionProgress.length, 0),
        completedSections: up.lessonProgress.reduce((total, lp) => 
          total + lp.sectionProgress.filter(sp => sp.completed).length, 0
        ),
        averageScore: calculateAverageScore(up.lessonProgress)
      }
    }));

    res.json({
      progress: courseId ? transformedProgress[0] || null : transformedProgress,
      summary: generateProgressSummary(transformedProgress)
    });
  })
);

// Helper functions
const calculateAverageScore = (lessonProgress) => {
  const scores = lessonProgress
    .flatMap(lp => lp.sectionProgress)
    .map(sp => sp.score)
    .filter(score => score !== null);
  
  return scores.length > 0 
    ? scores.reduce((sum, score) => sum + score, 0) / scores.length 
    : null;
};

const generateProgressSummary = (progress) => {
  if (progress.length === 0) {
    return {
      totalCourses: 0,
      completedCourses: 0,
      inProgressCourses: 0,
      totalTimeSpent: 0,
      averageCompletion: 0
    };
  }

  const totalCourses = progress.length;
  const completedCourses = progress.filter(p => p.completed).length;
  const inProgressCourses = totalCourses - completedCourses;
  const totalTimeSpent = progress.reduce((total, p) => total + p.totalTimeSpent, 0);
  
  const completionRates = progress.map(p => 
    p.statistics.totalSections > 0 
      ? (p.statistics.completedSections / p.statistics.totalSections) * 100 
      : 0
  );
  const averageCompletion = completionRates.reduce((sum, rate) => sum + rate, 0) / totalCourses;

  return {
    totalCourses,
    completedCourses,
    inProgressCourses,
    totalTimeSpent,
    averageCompletion: Math.round(averageCompletion)
  };
};

export default router;
```

## AI Service Integration Patterns

### Claude and DALL-E Integration Architecture
✅ **Good Example: Production-Ready AI Service**
```javascript
// services/aiService.js - AI integration for course generation
import { Anthropic } from '@anthropic-ai/sdk';
import OpenAI from 'openai';
import { prisma } from '../app.js';
import { z } from 'zod';

// Initialize AI clients
const anthropic = new Anthropic({
  apiKey: process.env.CLAUDE_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

// Schema validation for AI responses
const CourseOutlineSchema = z.object({
  title: z.string().min(1),
  description: z.string().min(10),
  lessons: z.array(z.object({
    title: z.string(),
    description: z.string(),
    sections: z.array(z.object({
      title: z.string(),
      type: z.enum(['text', 'video', 'quiz', 'exercise'])
    }))
  }))
});

export const generateCourseWithAI = async (courseParams, callbacks = {}) => {
  const { title, description, difficulty, tags } = courseParams;
  const { onProgress } = callbacks;

  try {
    // Stage 1: Generate course outline
    onProgress?.('outline', 10, 'Generating course structure...');
    
    const outline = await generateCourseOutline({ title, description, difficulty, tags });
    onProgress?.('outline', 30, 'Course structure created');

    // Stage 2: Generate detailed content
    onProgress?.('content', 40, 'Creating lesson content...');
    
    const detailedCourse = await generateLessonContent(outline);
    onProgress?.('content', 70, 'Lesson content generated');

    // Stage 3: Generate course image
    onProgress?.('image', 80, 'Creating course image...');
    
    const imageUrl = await generateCourseImage(title, description);
    onProgress?.('image', 90, 'Course image created');

    // Stage 4: Save to database
    onProgress?.('saving', 95, 'Saving course to database...');
    
    const course = await saveCourseToDatabase({
      ...detailedCourse,
      imageUrl,
      difficulty,
      tags
    });

    onProgress?.('complete', 100, 'Course generation completed!');
    
    return course;
    
  } catch (error) {
    console.error('AI course generation error:', error);
    throw new Error(`Course generation failed: ${error.message}`);
  }
};

const generateCourseOutline = async ({ title, description, difficulty, tags }) => {
  const prompt = `
Generate a comprehensive course outline for an educational platform with the following specifications:

Title: ${title}
Description: ${description}
Difficulty Level: ${difficulty}
Tags: ${tags.join(', ')}

Create a course with exactly 6 lessons, each containing exactly 2 sections. Each section should be educational and substantial.

Return your response as a JSON object with this exact structure:
{
  "title": "Course Title",
  "description": "Detailed course description",
  "estimatedHours": number,
  "lessons": [
    {
      "title": "Lesson Title",
      "description": "Lesson description",
      "order": 1,
      "estimatedMinutes": number,
      "sections": [
        {
          "title": "Section Title",
          "type": "text",
          "order": 1,
          "estimatedMinutes": number
        }
      ]
    }
  ]
}

Ensure variety in section types: text, quiz, exercise. Make content progressive and educational.
`;

  const response = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    temperature: 0.7,
    messages: [{ role: 'user', content: prompt }]
  });

  try {
    const courseData = JSON.parse(response.content[0].text);
    
    // Validate the response structure
    const validatedCourse = CourseOutlineSchema.parse(courseData);
    
    return validatedCourse;
  } catch (error) {
    console.error('Failed to parse AI response:', error);
    throw new Error('Invalid course structure generated by AI');
  }
};

const generateLessonContent = async (courseOutline) => {
  const lessonsWithContent = await Promise.all(
    courseOutline.lessons.map(async (lesson, lessonIndex) => {
      const sectionsWithContent = await Promise.all(
        lesson.sections.map(async (section, sectionIndex) => {
          const content = await generateSectionContent(
            courseOutline.title,
            lesson.title,
            section,
            lessonIndex + 1,
            sectionIndex + 1
          );
          
          return {
            ...section,
            content,
            estimatedMinutes: section.estimatedMinutes || 5
          };
        })
      );

      return {
        ...lesson,
        sections: sectionsWithContent
      };
    })
  );

  return {
    ...courseOutline,
    lessons: lessonsWithContent
  };
};

const generateSectionContent = async (courseTitle, lessonTitle, section, lessonNumber, sectionNumber) => {
  const contentPrompts = {
    text: `Create educational text content for:
Course: ${courseTitle}
Lesson ${lessonNumber}: ${lessonTitle}
Section ${sectionNumber}: ${section.title}

Provide comprehensive, educational content that teaches the topic clearly and thoroughly. Include examples and practical applications where relevant.`,

    quiz: `Create a quiz for:
Course: ${courseTitle}
Section: ${section.title}

Generate 3-5 multiple choice questions with explanations. Format as JSON:
{
  "questions": [
    {
      "question": "Question text",
      "options": ["A", "B", "C", "D"],
      "correctAnswer": 0,
      "explanation": "Why this is correct"
    }
  ]
}`,

    exercise: `Create a practical exercise for:
Course: ${courseTitle}
Section: ${section.title}

Include:
- Clear instructions
- Expected outcomes
- Step-by-step guidance
- Success criteria`
  };

  const prompt = contentPrompts[section.type] || contentPrompts.text;

  const response = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 2000,
    temperature: 0.6,
    messages: [{ role: 'user', content: prompt }]
  });

  return response.content[0].text;
};

const generateCourseImage = async (title, description) => {
  try {
    const imagePrompt = `Create a professional, educational course thumbnail for "${title}". 
    Style: Modern, clean, academic, with subtle technology elements. 
    Color scheme: Professional blues and whites. 
    No text overlay needed. 
    Focus on representing the subject matter visually.`;

    const response = await openai.images.generate({
      model: "dall-e-3",
      prompt: imagePrompt,
      size: "1024x1024",
      quality: "standard",
      n: 1,
    });

    return response.data[0].url;
  } catch (error) {
    console.error('Image generation error:', error);
    // Return a default placeholder if image generation fails
    return `https://via.placeholder.com/400x300/4f46e5/ffffff?text=${encodeURIComponent(title)}`;
  }
};

const saveCourseToDatabase = async (courseData) => {
  return await prisma.$transaction(async (tx) => {
    // Create the course
    const course = await tx.course.create({
      data: {
        title: courseData.title,
        description: courseData.description,
        difficulty: courseData.difficulty,
        imageUrl: courseData.imageUrl,
        estimatedHours: courseData.estimatedHours,
        tags: courseData.tags,
        status: 'published'
      }
    });

    // Create lessons and sections
    for (const lessonData of courseData.lessons) {
      const lesson = await tx.lesson.create({
        data: {
          title: lessonData.title,
          description: lessonData.description,
          order: lessonData.order,
          estimatedMinutes: lessonData.estimatedMinutes,
          courseId: course.id
        }
      });

      // Create sections for this lesson
      for (const sectionData of lessonData.sections) {
        await tx.section.create({
          data: {
            title: sectionData.title,
            content: sectionData.content,
            type: sectionData.type,
            order: sectionData.order,
            estimatedMinutes: sectionData.estimatedMinutes,
            lessonId: lesson.id
          }
        });
      }
    }

    return await tx.course.findUnique({
      where: { id: course.id },
      include: {
        lessons: {
          include: {
            sections: true
          },
          orderBy: { order: 'asc' }
        }
      }
    });
  });
};

// Retry mechanism for AI API calls
export const withRetry = async (operation, maxRetries = 3, delay = 1000) => {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await operation();
    } catch (error) {
      if (attempt === maxRetries) throw error;
      
      console.warn(`Attempt ${attempt} failed, retrying in ${delay}ms...`, error.message);
      await new Promise(resolve => setTimeout(resolve, delay));
      delay *= 2; // Exponential backoff
    }
  }
};

export default { generateCourseWithAI, withRetry };
```

## Key Node.js/Express Patterns Takeaways

### Critical Backend Development Patterns ✅

1. **Express Application Architecture**
   - Implement proper middleware ordering for security and functionality
   - Use environment-specific configurations and graceful shutdown handling
   - Structure routes with proper validation and error handling
   - Implement comprehensive logging and monitoring

2. **API Design Patterns**
   - Use RESTful conventions with proper HTTP status codes
   - Implement request validation using express-validator
   - Provide consistent error responses and proper data transformation
   - Use pagination, filtering, and search capabilities for list endpoints

3. **Database Integration**
   - Utilize Prisma ORM for type-safe database operations
   - Implement proper transaction handling for data consistency
   - Use database relationships effectively with proper includes and selects
   - Implement soft deletes and audit trails for data integrity

4. **AI Service Integration**
   - Implement proper error handling and retry mechanisms for external APIs
   - Use schema validation for AI-generated content
   - Implement progress tracking for long-running operations
   - Handle rate limiting and API quotas effectively

5. **Progress Tracking Architecture**
   - Design hierarchical progress tracking (course → lesson → section)
   - Use database transactions for progress consistency
   - Implement real-time progress updates and analytics
   - Provide comprehensive progress reporting and statistics

### Node.js/Express Anti-Patterns to Avoid ❌

1. **Blocking Operations** - Using synchronous operations that block the event loop
2. **Missing Error Handling** - Not implementing proper error handling middleware
3. **Unvalidated Input** - Accepting user input without proper validation and sanitization
4. **Database Connection Leaks** - Not properly closing database connections
5. **Missing Security Headers** - Not implementing proper security middleware and headers

Node.js and Express patterns are essential for building scalable, secure, and maintainable LMS backend systems that can handle educational content management, user progress tracking, and AI integration effectively. 