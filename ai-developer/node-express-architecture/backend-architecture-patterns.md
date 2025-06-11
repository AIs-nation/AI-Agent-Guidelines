# Node.js/Express Backend Architecture for LMS Systems

## Project Structure and Organization

### Modular Architecture Pattern
**Organize code by feature/domain rather than by type**

✅ **Good Example: Feature-Based Structure**
```
src/
├── courses/
│   ├── controllers/
│   │   ├── courseController.js
│   │   └── lessonController.js
│   ├── services/
│   │   ├── courseService.js
│   │   └── aiCourseGenerator.js
│   ├── models/
│   │   ├── Course.js
│   │   └── Lesson.js
│   ├── routes/
│   │   └── courseRoutes.js
│   └── validators/
│       └── courseValidators.js
├── progress/
│   ├── controllers/
│   │   └── progressController.js
│   ├── services/
│   │   └── progressService.js
│   ├── models/
│   │   └── Progress.js
│   └── routes/
│       └── progressRoutes.js
├── shared/
│   ├── middleware/
│   ├── utils/
│   └── config/
└── app.js
```

❌ **Bad Example: Type-Based Structure (Flat)**
```
src/
├── controllers/
│   ├── courseController.js
│   ├── lessonController.js
│   ├── progressController.js
│   └── userController.js
├── models/
│   ├── Course.js
│   ├── Lesson.js
│   ├── Progress.js
│   └── User.js
├── routes/
│   └── index.js
└── app.js
```

## MVC with Service Layer Pattern

### Controllers - Request/Response Handling
✅ **Good Example: Thin Controllers**
```javascript
// courseController.js
class CourseController {
  static async createCourse(req, res, next) {
    try {
      const { prompt } = req.body;
      
      // Validation happens in middleware
      const course = await CourseService.generateCourse(prompt);
      
      res.status(201).json({
        success: true,
        data: course,
        message: 'Course created successfully'
      });
    } catch (error) {
      next(error);
    }
  }

  static async getCourse(req, res, next) {
    try {
      const { id } = req.params;
      const course = await CourseService.getCourseById(id);
      
      if (!course) {
        return res.status(404).json({
          success: false,
          message: 'Course not found'
        });
      }

      res.json({
        success: true,
        data: course
      });
    } catch (error) {
      next(error);
    }
  }
}

module.exports = CourseController;
```

❌ **Bad Example: Fat Controllers with Business Logic**
```javascript
// courseController.js - BAD
class CourseController {
  static async createCourse(req, res) {
    try {
      const { prompt } = req.body;
      
      // Validation logic mixed in controller
      if (!prompt || prompt.length < 10) {
        return res.status(400).json({ error: 'Invalid prompt' });
      }

      // Business logic in controller
      const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });
      const completion = await openai.chat.completions.create({
        model: "gpt-3.5-turbo",
        messages: [{ role: "user", content: prompt }]
      });

      // Database operations in controller
      const course = await prisma.course.create({
        data: {
          title: completion.choices[0].message.content,
          description: 'Generated course'
        }
      });

      res.json(course);
    } catch (error) {
      console.error(error);
      res.status(500).json({ error: 'Server error' });
    }
  }
}
```

### Services - Business Logic Layer
✅ **Good Example: Service Layer with Clear Responsibilities**
```javascript
// courseService.js
const { PrismaClient } = require('@prisma/client');
const AIService = require('../shared/services/aiService');
const ValidationService = require('../shared/services/validationService');

class CourseService {
  constructor() {
    this.prisma = new PrismaClient();
  }

  async generateCourse(prompt) {
    // Input validation
    ValidationService.validateCoursePrompt(prompt);

    try {
      // Generate course outline using AI
      const courseOutline = await AIService.generateCourseOutline(prompt);
      
      // Generate detailed content
      const courseContent = await AIService.generateCourseContent(courseOutline);
      
      // Save to database
      const course = await this.createCourseInDatabase(courseContent);
      
      // Create default progress tracking
      await this.initializeProgressTracking(course.id);
      
      return course;
    } catch (error) {
      throw new Error(`Course generation failed: ${error.message}`);
    }
  }

  async getCourseById(courseId) {
    const course = await this.prisma.course.findUnique({
      where: { id: courseId },
      include: {
        lessons: {
          include: {
            sections: true
          }
        }
      }
    });

    if (!course) {
      throw new Error('Course not found');
    }

    return this.formatCourseResponse(course);
  }

  async createCourseInDatabase(courseData) {
    return await this.prisma.course.create({
      data: {
        title: courseData.title,
        description: courseData.description,
        lessons: {
          create: courseData.lessons.map(lesson => ({
            title: lesson.title,
            order: lesson.order,
            sections: {
              create: lesson.sections.map(section => ({
                title: section.title,
                content: section.content,
                type: section.type,
                order: section.order
              }))
            }
          }))
        }
      },
      include: {
        lessons: {
          include: {
            sections: true
          }
        }
      }
    });
  }

  formatCourseResponse(course) {
    return {
      id: course.id,
      title: course.title,
      description: course.description,
      lessons: course.lessons.map(lesson => ({
        id: lesson.id,
        title: lesson.title,
        order: lesson.order,
        sectionsCount: lesson.sections.length,
        sections: lesson.sections.map(section => ({
          id: section.id,
          title: section.title,
          type: section.type,
          order: section.order
        }))
      }))
    };
  }
}

module.exports = new CourseService();
```

## API Design Best Practices

### RESTful API Structure
✅ **Good Example: Consistent API Design**
```javascript
// courseRoutes.js
const express = require('express');
const CourseController = require('../controllers/courseController');
const { validateCourseCreation, validateCourseId } = require('../validators/courseValidators');
const rateLimit = require('express-rate-limit');

const router = express.Router();

// Rate limiting for course generation
const courseGenerationLimit = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // Limit each IP to 5 course generations per windowMs
  message: 'Too many course generation requests, please try again later'
});

// RESTful routes
router.get('/', CourseController.getAllCourses);
router.get('/:id', validateCourseId, CourseController.getCourse);
router.post('/', courseGenerationLimit, validateCourseCreation, CourseController.createCourse);
router.put('/:id', validateCourseId, CourseController.updateCourse);
router.delete('/:id', validateCourseId, CourseController.deleteCourse);

// Nested resources
router.get('/:id/lessons', validateCourseId, CourseController.getCourseLessons);
router.get('/:id/progress', validateCourseId, CourseController.getCourseProgress);

// Specific actions
router.post('/:id/enroll', validateCourseId, CourseController.enrollInCourse);
router.post('/generate-stream', courseGenerationLimit, CourseController.generateCourseStream);

module.exports = router;
```

### Request/Response Formatting
✅ **Good Example: Consistent Response Format**
```javascript
// responseFormatter.js
class ResponseFormatter {
  static success(data, message = 'Success', statusCode = 200) {
    return {
      success: true,
      data,
      message,
      timestamp: new Date().toISOString()
    };
  }

  static error(message, statusCode = 500, errors = null) {
    return {
      success: false,
      message,
      errors,
      timestamp: new Date().toISOString()
    };
  }

  static paginated(data, pagination) {
    return {
      success: true,
      data,
      pagination: {
        currentPage: pagination.page,
        totalPages: Math.ceil(pagination.total / pagination.limit),
        totalItems: pagination.total,
        itemsPerPage: pagination.limit,
        hasNextPage: pagination.page < Math.ceil(pagination.total / pagination.limit),
        hasPreviousPage: pagination.page > 1
      },
      timestamp: new Date().toISOString()
    };
  }
}

module.exports = ResponseFormatter;
```

## Database Patterns with Prisma

### Schema Design for LMS
✅ **Good Example: Properly Structured Prisma Schema**
```prisma
// schema.prisma
model Course {
  id          String   @id @default(cuid())
  title       String
  description String?
  imageUrl    String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  lessons     Lesson[]
  enrollments Enrollment[]
  
  @@map("courses")
}

model Lesson {
  id       String @id @default(cuid())
  title    String
  order    Int
  courseId String
  
  course   Course    @relation(fields: [courseId], references: [id], onDelete: Cascade)
  sections Section[]
  
  @@map("lessons")
  @@index([courseId, order])
}

model Section {
  id       String      @id @default(cuid())
  title    String
  content  String
  type     SectionType @default(TEXT)
  order    Int
  lessonId String
  
  lesson   Lesson @relation(fields: [lessonId], references: [id], onDelete: Cascade)
  
  @@map("sections")
  @@index([lessonId, order])
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  enrollments Enrollment[]
  progress    Progress[]
  
  @@map("users")
}

model Enrollment {
  id         String           @id @default(cuid())
  userId     String
  courseId   String
  status     EnrollmentStatus @default(ACTIVE)
  enrolledAt DateTime         @default(now())
  
  user   User   @relation(fields: [userId], references: [id])
  course Course @relation(fields: [courseId], references: [id])
  
  @@unique([userId, courseId])
  @@map("enrollments")
}

model Progress {
  id            String   @id @default(cuid())
  userId        String
  courseId      String
  lessonId      String
  sectionId     String
  completedAt   DateTime @default(now())
  timeSpent     Int      @default(0) // in seconds
  
  user User @relation(fields: [userId], references: [id])
  
  @@unique([userId, sectionId])
  @@map("progress")
  @@index([userId, courseId])
}

enum SectionType {
  TEXT
  VIDEO
  QUIZ
  ASSIGNMENT
  INTERACTIVE
}

enum EnrollmentStatus {
  ACTIVE
  COMPLETED
  DROPPED
}
```

### Database Service Pattern
✅ **Good Example: Repository Pattern with Prisma**
```javascript
// repositories/courseRepository.js
class CourseRepository {
  constructor(prisma) {
    this.prisma = prisma;
  }

  async findAllWithPagination(page = 1, limit = 10, filters = {}) {
    const skip = (page - 1) * limit;
    
    const where = {};
    if (filters.search) {
      where.OR = [
        { title: { contains: filters.search, mode: 'insensitive' } },
        { description: { contains: filters.search, mode: 'insensitive' } }
      ];
    }

    const [courses, total] = await Promise.all([
      this.prisma.course.findMany({
        where,
        include: {
          lessons: {
            include: {
              sections: true
            }
          },
          _count: {
            select: {
              enrollments: true
            }
          }
        },
        skip,
        take: limit,
        orderBy: { createdAt: 'desc' }
      }),
      this.prisma.course.count({ where })
    ]);

    return { courses, total };
  }

  async findByIdWithProgress(courseId, userId = null) {
    const course = await this.prisma.course.findUnique({
      where: { id: courseId },
      include: {
        lessons: {
          include: {
            sections: true
          },
          orderBy: { order: 'asc' }
        }
      }
    });

    if (!course) return null;

    // Get user progress if userId provided
    let userProgress = null;
    if (userId) {
      userProgress = await this.getUserProgress(courseId, userId);
    }

    return { course, userProgress };
  }

  async getUserProgress(courseId, userId) {
    const progress = await this.prisma.progress.findMany({
      where: {
        userId,
        courseId
      },
      select: {
        sectionId: true,
        completedAt: true,
        timeSpent: true
      }
    });

    return progress.reduce((acc, p) => {
      acc[p.sectionId] = {
        completed: true,
        completedAt: p.completedAt,
        timeSpent: p.timeSpent
      };
      return acc;
    }, {});
  }

  async createWithRelations(courseData) {
    return await this.prisma.course.create({
      data: {
        title: courseData.title,
        description: courseData.description,
        imageUrl: courseData.imageUrl,
        lessons: {
          create: courseData.lessons.map(lesson => ({
            title: lesson.title,
            order: lesson.order,
            sections: {
              create: lesson.sections.map(section => ({
                title: section.title,
                content: section.content,
                type: section.type,
                order: section.order
              }))
            }
          }))
        }
      },
      include: {
        lessons: {
          include: {
            sections: true
          }
        }
      }
    });
  }
}

module.exports = CourseRepository;
```

## Security Best Practices

### Input Validation and Sanitization
✅ **Good Example: Comprehensive Validation**
```javascript
// validators/courseValidators.js
const Joi = require('joi');
const validator = require('validator');

const courseCreationSchema = Joi.object({
  prompt: Joi.string()
    .min(10)
    .max(1000)
    .required()
    .custom((value, helpers) => {
      // Remove potentially harmful content
      const sanitized = validator.escape(value);
      if (sanitized !== value) {
        return helpers.error('string.unsafe');
      }
      return value;
    }),
  difficulty: Joi.string()
    .valid('beginner', 'intermediate', 'advanced')
    .default('beginner'),
  duration: Joi.number()
    .integer()
    .min(1)
    .max(100)
    .default(6)
});

const validateCourseCreation = (req, res, next) => {
  const { error, value } = courseCreationSchema.validate(req.body);
  
  if (error) {
    return res.status(400).json({
      success: false,
      message: 'Validation failed',
      errors: error.details.map(detail => ({
        field: detail.path.join('.'),
        message: detail.message
      }))
    });
  }
  
  req.body = value;
  next();
};

const validateCourseId = (req, res, next) => {
  const { id } = req.params;
  
  if (!id || !validator.isAlphanumeric(id)) {
    return res.status(400).json({
      success: false,
      message: 'Invalid course ID format'
    });
  }
  
  next();
};

module.exports = {
  validateCourseCreation,
  validateCourseId
};
```

### Security Middleware
✅ **Good Example: Security Middleware Stack**
```javascript
// middleware/security.js
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const mongoSanitize = require('express-mongo-sanitize');
const xss = require('xss-clean');
const hpp = require('hpp');
const cors = require('cors');

const securityMiddleware = (app) => {
  // Basic security headers
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

  // CORS configuration
  app.use(cors({
    origin: process.env.ALLOWED_ORIGINS?.split(',') || 'http://localhost:3000',
    credentials: true,
    optionsSuccessStatus: 200
  }));

  // Rate limiting
  const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // Limit each IP to 100 requests per windowMs
    message: 'Too many requests from this IP, please try again later',
    standardHeaders: true,
    legacyHeaders: false,
  });
  app.use('/api/', limiter);

  // Data sanitization
  app.use(mongoSanitize()); // Prevent NoSQL injection
  app.use(xss()); // Clean user input from malicious HTML
  app.use(hpp()); // Prevent HTTP Parameter Pollution

  // Body size limiting
  app.use(express.json({ limit: '10mb' }));
  app.use(express.urlencoded({ extended: true, limit: '10mb' }));
};

module.exports = securityMiddleware;
```

## Error Handling and Logging

### Global Error Handler
✅ **Good Example: Centralized Error Handling**
```javascript
// middleware/errorHandler.js
const logger = require('../utils/logger');

class AppError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.statusCode = statusCode;
    this.status = `${statusCode}`.startsWith('4') ? 'fail' : 'error';
    this.isOperational = true;

    Error.captureStackTrace(this, this.constructor);
  }
}

const handleCastErrorDB = (err) => {
  const message = `Invalid ${err.path}: ${err.value}`;
  return new AppError(message, 400);
};

const handleDuplicateFieldsDB = (err) => {
  const value = err.errmsg.match(/(["'])(\\?.)*?\1/)[0];
  const message = `Duplicate field value: ${value}. Please use another value!`;
  return new AppError(message, 400);
};

const handleValidationErrorDB = (err) => {
  const errors = Object.values(err.errors).map(el => el.message);
  const message = `Invalid input data. ${errors.join('. ')}`;
  return new AppError(message, 400);
};

const sendErrorDev = (err, req, res) => {
  logger.error('ERROR 💥', err);
  
  return res.status(err.statusCode).json({
    success: false,
    error: err,
    message: err.message,
    stack: err.stack
  });
};

const sendErrorProd = (err, req, res) => {
  // Operational, trusted error: send message to client
  if (err.isOperational) {
    return res.status(err.statusCode).json({
      success: false,
      message: err.message
    });
  }

  // Programming or other unknown error: don't leak error details
  logger.error('ERROR 💥', err);
  
  return res.status(500).json({
    success: false,
    message: 'Something went wrong!'
  });
};

const globalErrorHandler = (err, req, res, next) => {
  err.statusCode = err.statusCode || 500;
  err.status = err.status || 'error';

  if (process.env.NODE_ENV === 'development') {
    sendErrorDev(err, req, res);
  } else {
    let error = { ...err };
    error.message = err.message;

    if (error.name === 'CastError') error = handleCastErrorDB(error);
    if (error.code === 11000) error = handleDuplicateFieldsDB(error);
    if (error.name === 'ValidationError') error = handleValidationErrorDB(error);

    sendErrorProd(error, req, res);
  }
};

module.exports = {
  AppError,
  globalErrorHandler
};
```

### Structured Logging
✅ **Good Example: Winston Logger Configuration**
```javascript
// utils/logger.js
const winston = require('winston');
const path = require('path');

const logFormat = winston.format.combine(
  winston.format.timestamp(),
  winston.format.errors({ stack: true }),
  winston.format.json()
);

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: logFormat,
  defaultMeta: { service: 'lms-backend' },
  transports: [
    new winston.transports.File({
      filename: path.join(__dirname, '../logs/error.log'),
      level: 'error'
    }),
    new winston.transports.File({
      filename: path.join(__dirname, '../logs/combined.log')
    })
  ]
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.combine(
      winston.format.colorize(),
      winston.format.simple()
    )
  }));
}

module.exports = logger;
```

## Performance Optimization

### Caching Strategy
✅ **Good Example: Redis Caching Implementation**
```javascript
// services/cacheService.js
const Redis = require('redis');
const logger = require('../utils/logger');

class CacheService {
  constructor() {
    this.client = Redis.createClient({
      host: process.env.REDIS_HOST || 'localhost',
      port: process.env.REDIS_PORT || 6379,
      password: process.env.REDIS_PASSWORD
    });

    this.client.on('error', (err) => {
      logger.error('Redis Client Error', err);
    });

    this.client.on('connect', () => {
      logger.info('Connected to Redis');
    });
  }

  async get(key) {
    try {
      const cached = await this.client.get(key);
      return cached ? JSON.parse(cached) : null;
    } catch (error) {
      logger.error('Cache get error:', error);
      return null;
    }
  }

  async set(key, value, expiration = 3600) {
    try {
      await this.client.setex(key, expiration, JSON.stringify(value));
      return true;
    } catch (error) {
      logger.error('Cache set error:', error);
      return false;
    }
  }

  async delete(key) {
    try {
      await this.client.del(key);
      return true;
    } catch (error) {
      logger.error('Cache delete error:', error);
      return false;
    }
  }

  generateKey(prefix, ...parts) {
    return `${prefix}:${parts.join(':')}`;
  }
}

module.exports = new CacheService();
```

### Database Query Optimization
✅ **Good Example: Optimized Queries with Caching**
```javascript
// services/optimizedCourseService.js
const CacheService = require('./cacheService');

class OptimizedCourseService {
  async getCourseWithCache(courseId) {
    const cacheKey = CacheService.generateKey('course', courseId);
    
    // Try to get from cache first
    let course = await CacheService.get(cacheKey);
    
    if (!course) {
      // If not in cache, get from database
      course = await this.prisma.course.findUnique({
        where: { id: courseId },
        include: {
          lessons: {
            include: {
              sections: {
                select: {
                  id: true,
                  title: true,
                  type: true,
                  order: true
                  // Don't include content to reduce payload
                }
              }
            },
            orderBy: { order: 'asc' }
          }
        }
      });

      // Cache for 1 hour
      if (course) {
        await CacheService.set(cacheKey, course, 3600);
      }
    }

    return course;
  }

  async getCoursesWithPagination(page, limit, filters) {
    const cacheKey = CacheService.generateKey('courses', page, limit, JSON.stringify(filters));
    
    let result = await CacheService.get(cacheKey);
    
    if (!result) {
      // Optimized query with selective fields
      const skip = (page - 1) * limit;
      
      const [courses, total] = await Promise.all([
        this.prisma.course.findMany({
          select: {
            id: true,
            title: true,
            description: true,
            imageUrl: true,
            createdAt: true,
            _count: {
              select: {
                lessons: true,
                enrollments: true
              }
            }
          },
          skip,
          take: limit,
          orderBy: { createdAt: 'desc' }
        }),
        this.prisma.course.count()
      ]);

      result = { courses, total };
      
      // Cache for 10 minutes
      await CacheService.set(cacheKey, result, 600);
    }

    return result;
  }
}
```

## Key Takeaways for LMS Backend Development

1. **Use feature-based architecture** - Organize by domain, not by technical layers
2. **Implement service layer pattern** - Keep controllers thin, business logic in services
3. **Design RESTful APIs consistently** - Standard HTTP methods and response formats
4. **Optimize database queries** - Use indexes, selective fields, and caching
5. **Implement comprehensive security** - Input validation, rate limiting, sanitization  
6. **Use structured error handling** - Centralized error handling with proper logging
7. **Cache strategically** - Cache expensive operations and frequently accessed data
8. **Monitor performance** - Use logging and metrics to identify bottlenecks

These patterns ensure scalable, secure, and maintainable backend architecture specifically designed for learning management system requirements. 