# Comprehensive Node.js LMS Backend Architecture Guide

## Executive Summary

This guide provides a production-ready architecture framework for building scalable Learning Management System (LMS) backends using Node.js. Based on real-world implementations and performance benchmarks, it covers everything from basic setup to enterprise-grade scaling patterns.

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Core Technology Stack](#core-technology-stack)
3. [Project Structure](#project-structure)
4. [Database Design](#database-design)
5. [Authentication & Authorization](#authentication--authorization)
6. [API Design Patterns](#api-design-patterns)
7. [Real-time Features](#real-time-features)
8. [File Management](#file-management)
9. [Scaling Strategies](#scaling-strategies)
10. [Performance Optimization](#performance-optimization)
11. [Security Best Practices](#security-best-practices)
12. [Monitoring & Logging](#monitoring--logging)
13. [Deployment & DevOps](#deployment--devops)

## Architecture Overview

### Microservices vs Monolithic Approach

**For Most LMS Projects: Start Monolithic, Scale to Microservices**

```javascript
// Monolithic structure (recommended for small-medium teams)
├── auth/           // Authentication & user management
├── courses/        // Course management
├── content/        // Content delivery
├── analytics/      // Learning analytics
├── notifications/  // Email/push notifications
└── common/         // Shared utilities
```

**When to Consider Microservices:**
- Team size > 15 developers
- Clear domain boundaries
- Independent scaling requirements
- Complex deployment needs

### Event-Driven Architecture

```javascript
// Event-driven pattern for LMS
const EventEmitter = require('events');

class LMSEventBus extends EventEmitter {
  // Course events
  courseCreated(courseData) {
    this.emit('course.created', courseData);
  }
  
  courseCompleted(userId, courseId) {
    this.emit('course.completed', { userId, courseId });
  }
  
  // User events
  userRegistered(userData) {
    this.emit('user.registered', userData);
  }
  
  // Progress events
  lessonCompleted(progressData) {
    this.emit('lesson.completed', progressData);
  }
}

const eventBus = new LMSEventBus();

// Listeners for analytics
eventBus.on('course.completed', async (data) => {
  await analyticsService.recordCompletion(data);
  await certificateService.generate(data.userId, data.courseId);
  await notificationService.sendCompletionEmail(data);
});
```

## Core Technology Stack

### Recommended Stack for 2025

```json
{
  "runtime": "Node.js 20+ LTS",
  "framework": "Express.js 4.x",
  "database": {
    "primary": "PostgreSQL 15+",
    "cache": "Redis 7+",
    "search": "Elasticsearch 8+"
  },
  "authentication": "JWT + Refresh Tokens",
  "fileStorage": "AWS S3 / MinIO",
  "realTime": "Socket.io",
  "validation": "Joi / Zod",
  "testing": "Jest + Supertest",
  "documentation": "OpenAPI 3.0"
}
```

### Package.json Foundation

```json
{
  "dependencies": {
    "express": "^4.18.2",
    "express-rate-limit": "^6.7.0",
    "express-validator": "^6.15.0",
    "helmet": "^6.1.5",
    "cors": "^2.8.5",
    "compression": "^1.7.4",
    "morgan": "^1.10.0",
    "winston": "^3.8.2",
    "jsonwebtoken": "^9.0.0",
    "bcryptjs": "^2.4.3",
    "multer": "^1.4.5",
    "sharp": "^0.32.1",
    "socket.io": "^4.6.2",
    "joi": "^17.9.2",
    "pg": "^8.11.0",
    "redis": "^4.6.7",
    "bull": "^4.10.4",
    "nodemailer": "^6.9.3",
    "aws-sdk": "^2.1383.0"
  },
  "devDependencies": {
    "jest": "^29.5.0",
    "supertest": "^6.3.3",
    "nodemon": "^2.0.22",
    "eslint": "^8.42.0",
    "prettier": "^2.8.8"
  }
}
```

## Project Structure

### Enterprise-Grade Structure

```
src/
├── config/                 # Configuration files
│   ├── database.js
│   ├── redis.js
│   ├── aws.js
│   └── environment.js
├── middleware/             # Custom middleware
│   ├── auth.js
│   ├── validation.js
│   ├── rateLimiting.js
│   └── errorHandler.js
├── models/                 # Database models
│   ├── User.js
│   ├── Course.js
│   ├── Lesson.js
│   └── Progress.js
├── controllers/            # Route controllers
│   ├── authController.js
│   ├── courseController.js
│   ├── userController.js
│   └── analyticsController.js
├── services/              # Business logic
│   ├── authService.js
│   ├── courseService.js
│   ├── emailService.js
│   └── analyticsService.js
├── routes/                # Route definitions
│   ├── auth.js
│   ├── courses.js
│   ├── users.js
│   └── analytics.js
├── utils/                 # Utility functions
│   ├── logger.js
│   ├── validators.js
│   ├── helpers.js
│   └── constants.js
├── jobs/                  # Background jobs
│   ├── emailJobs.js
│   ├── analyticsJobs.js
│   └── cleanupJobs.js
├── tests/                 # Test files
│   ├── unit/
│   ├── integration/
│   └── helpers/
└── app.js                 # Application entry point
```

## Database Design

### PostgreSQL Schema for LMS

```sql
-- Users table
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100) NOT NULL,
  role VARCHAR(20) DEFAULT 'student' CHECK (role IN ('admin', 'instructor', 'student')),
  is_active BOOLEAN DEFAULT true,
  profile_image_url TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Courses table
CREATE TABLE courses (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title VARCHAR(255) NOT NULL,
  description TEXT,
  instructor_id UUID REFERENCES users(id),
  category VARCHAR(100),
  difficulty_level VARCHAR(20) CHECK (difficulty_level IN ('beginner', 'intermediate', 'advanced')),
  estimated_duration INTEGER, -- in minutes
  price DECIMAL(10,2) DEFAULT 0.00,
  is_published BOOLEAN DEFAULT false,
  thumbnail_url TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Lessons table
CREATE TABLE lessons (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  course_id UUID REFERENCES courses(id) ON DELETE CASCADE,
  title VARCHAR(255) NOT NULL,
  content TEXT,
  video_url TEXT,
  duration INTEGER, -- in seconds
  order_index INTEGER NOT NULL,
  lesson_type VARCHAR(20) DEFAULT 'video' CHECK (lesson_type IN ('video', 'text', 'quiz', 'assignment')),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- User course enrollments
CREATE TABLE enrollments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  course_id UUID REFERENCES courses(id) ON DELETE CASCADE,
  enrolled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  completed_at TIMESTAMP,
  progress_percentage DECIMAL(5,2) DEFAULT 0.00,
  UNIQUE(user_id, course_id)
);

-- Lesson progress tracking
CREATE TABLE lesson_progress (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  lesson_id UUID REFERENCES lessons(id) ON DELETE CASCADE,
  enrollment_id UUID REFERENCES enrollments(id) ON DELETE CASCADE,
  completed BOOLEAN DEFAULT false,
  time_spent INTEGER DEFAULT 0, -- in seconds
  last_accessed TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  completion_date TIMESTAMP,
  UNIQUE(user_id, lesson_id)
);

-- Quizzes and assessments
CREATE TABLE quizzes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  lesson_id UUID REFERENCES lessons(id) ON DELETE CASCADE,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  passing_score INTEGER DEFAULT 70,
  time_limit INTEGER, -- in minutes
  attempts_allowed INTEGER DEFAULT 3,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Quiz questions
CREATE TABLE quiz_questions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  quiz_id UUID REFERENCES quizzes(id) ON DELETE CASCADE,
  question_text TEXT NOT NULL,
  question_type VARCHAR(20) DEFAULT 'multiple_choice' CHECK (question_type IN ('multiple_choice', 'true_false', 'short_answer')),
  options JSONB, -- For multiple choice options
  correct_answer TEXT NOT NULL,
  points INTEGER DEFAULT 1,
  order_index INTEGER NOT NULL
);

-- Quiz attempts
CREATE TABLE quiz_attempts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  quiz_id UUID REFERENCES quizzes(id) ON DELETE CASCADE,
  score INTEGER NOT NULL,
  max_score INTEGER NOT NULL,
  answers JSONB NOT NULL,
  started_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  completed_at TIMESTAMP,
  passed BOOLEAN DEFAULT false
);

-- Indexes for performance
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_courses_instructor ON courses(instructor_id);
CREATE INDEX idx_lessons_course ON lessons(course_id);
CREATE INDEX idx_enrollments_user ON enrollments(user_id);
CREATE INDEX idx_enrollments_course ON enrollments(course_id);
CREATE INDEX idx_lesson_progress_user ON lesson_progress(user_id);
CREATE INDEX idx_lesson_progress_lesson ON lesson_progress(lesson_id);
```

### Database Models with Sequelize

```javascript
// models/User.js
const { DataTypes } = require('sequelize');
const bcrypt = require('bcryptjs');

module.exports = (sequelize) => {
  const User = sequelize.define('User', {
    id: {
      type: DataTypes.UUID,
      defaultValue: DataTypes.UUIDV4,
      primaryKey: true
    },
    email: {
      type: DataTypes.STRING,
      allowNull: false,
      unique: true,
      validate: {
        isEmail: true
      }
    },
    passwordHash: {
      type: DataTypes.STRING,
      allowNull: false,
      field: 'password_hash'
    },
    firstName: {
      type: DataTypes.STRING,
      allowNull: false,
      field: 'first_name'
    },
    lastName: {
      type: DataTypes.STRING,
      allowNull: false,
      field: 'last_name'
    },
    role: {
      type: DataTypes.ENUM('admin', 'instructor', 'student'),
      defaultValue: 'student'
    },
    isActive: {
      type: DataTypes.BOOLEAN,
      defaultValue: true,
      field: 'is_active'
    },
    profileImageUrl: {
      type: DataTypes.TEXT,
      field: 'profile_image_url'
    }
  }, {
    tableName: 'users',
    underscored: true,
    hooks: {
      beforeCreate: async (user) => {
        if (user.passwordHash) {
          user.passwordHash = await bcrypt.hash(user.passwordHash, 12);
        }
      },
      beforeUpdate: async (user) => {
        if (user.changed('passwordHash')) {
          user.passwordHash = await bcrypt.hash(user.passwordHash, 12);
        }
      }
    }
  });

  User.prototype.comparePassword = async function(password) {
    return bcrypt.compare(password, this.passwordHash);
  };

  User.prototype.toJSON = function() {
    const values = Object.assign({}, this.get());
    delete values.passwordHash;
    return values;
  };

  return User;
};
```

## Authentication & Authorization

### JWT Implementation with Refresh Tokens

```javascript
// services/authService.js
const jwt = require('jsonwebtoken');
const crypto = require('crypto');
const { User } = require('../models');
const redis = require('../config/redis');

class AuthService {
  generateTokens(user) {
    const payload = {
      userId: user.id,
      email: user.email,
      role: user.role
    };

    const accessToken = jwt.sign(payload, process.env.JWT_SECRET, {
      expiresIn: process.env.JWT_EXPIRES_IN || '15m'
    });

    const refreshToken = jwt.sign(payload, process.env.JWT_REFRESH_SECRET, {
      expiresIn: process.env.JWT_REFRESH_EXPIRES_IN || '7d'
    });

    return { accessToken, refreshToken };
  }

  async storeRefreshToken(userId, refreshToken) {
    const key = `refresh_token:${userId}`;
    await redis.setex(key, 7 * 24 * 60 * 60, refreshToken); // 7 days
  }

  async validateRefreshToken(refreshToken) {
    try {
      const decoded = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET);
      const storedToken = await redis.get(`refresh_token:${decoded.userId}`);
      
      if (storedToken !== refreshToken) {
        throw new Error('Invalid refresh token');
      }

      return decoded;
    } catch (error) {
      throw new Error('Invalid refresh token');
    }
  }

  async revokeRefreshToken(userId) {
    await redis.del(`refresh_token:${userId}`);
  }

  async login(email, password) {
    const user = await User.findOne({ where: { email, isActive: true } });
    
    if (!user || !(await user.comparePassword(password))) {
      throw new Error('Invalid credentials');
    }

    const tokens = this.generateTokens(user);
    await this.storeRefreshToken(user.id, tokens.refreshToken);

    return {
      user: user.toJSON(),
      ...tokens
    };
  }

  async refreshTokens(refreshToken) {
    const decoded = await this.validateRefreshToken(refreshToken);
    const user = await User.findByPk(decoded.userId);

    if (!user || !user.isActive) {
      throw new Error('User not found or inactive');
    }

    const tokens = this.generateTokens(user);
    await this.storeRefreshToken(user.id, tokens.refreshToken);

    return tokens;
  }
}

module.exports = new AuthService();
```

### Middleware for Authentication

```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');
const { User } = require('../models');

const authenticateToken = async (req, res, next) => {
  try {
    const authHeader = req.headers.authorization;
    const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN

    if (!token) {
      return res.status(401).json({ error: 'Access token required' });
    }

    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    const user = await User.findByPk(decoded.userId);

    if (!user || !user.isActive) {
      return res.status(401).json({ error: 'Invalid token' });
    }

    req.user = user;
    next();
  } catch (error) {
    return res.status(403).json({ error: 'Invalid token' });
  }
};

const authorize = (roles = []) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }

    if (roles.length && !roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }

    next();
  };
};

module.exports = { authenticateToken, authorize };
```

## API Design Patterns

### RESTful API Structure

```javascript
// controllers/courseController.js
const { Course, Lesson, User, Enrollment } = require('../models');
const { validationResult } = require('express-validator');
const courseService = require('../services/courseService');

class CourseController {
  // GET /api/courses
  async getAllCourses(req, res) {
    try {
      const { page = 1, limit = 10, category, difficulty, search } = req.query;
      
      const result = await courseService.getCourses({
        page: parseInt(page),
        limit: parseInt(limit),
        category,
        difficulty,
        search
      });

      res.json({
        success: true,
        data: result.courses,
        pagination: {
          currentPage: parseInt(page),
          totalPages: Math.ceil(result.total / limit),
          totalItems: result.total,
          itemsPerPage: parseInt(limit)
        }
      });
    } catch (error) {
      res.status(500).json({
        success: false,
        error: error.message
      });
    }
  }

  // GET /api/courses/:id
  async getCourseById(req, res) {
    try {
      const { id } = req.params;
      const course = await courseService.getCourseById(id, req.user);

      if (!course) {
        return res.status(404).json({
          success: false,
          error: 'Course not found'
        });
      }

      res.json({
        success: true,
        data: course
      });
    } catch (error) {
      res.status(500).json({
        success: false,
        error: error.message
      });
    }
  }

  // POST /api/courses
  async createCourse(req, res) {
    try {
      const errors = validationResult(req);
      if (!errors.isEmpty()) {
        return res.status(400).json({
          success: false,
          errors: errors.array()
        });
      }

      const courseData = {
        ...req.body,
        instructorId: req.user.id
      };

      const course = await courseService.createCourse(courseData);

      res.status(201).json({
        success: true,
        data: course,
        message: 'Course created successfully'
      });
    } catch (error) {
      res.status(500).json({
        success: false,
        error: error.message
      });
    }
  }

  // POST /api/courses/:id/enroll
  async enrollInCourse(req, res) {
    try {
      const { id } = req.params;
      const enrollment = await courseService.enrollUser(req.user.id, id);

      res.status(201).json({
        success: true,
        data: enrollment,
        message: 'Successfully enrolled in course'
      });
    } catch (error) {
      if (error.message === 'Already enrolled') {
        return res.status(409).json({
          success: false,
          error: error.message
        });
      }

      res.status(500).json({
        success: false,
        error: error.message
      });
    }
  }

  // PUT /api/courses/:id/progress
  async updateProgress(req, res) {
    try {
      const { id } = req.params;
      const { lessonId, completed, timeSpent } = req.body;

      const progress = await courseService.updateLessonProgress(
        req.user.id,
        id,
        lessonId,
        { completed, timeSpent }
      );

      res.json({
        success: true,
        data: progress
      });
    } catch (error) {
      res.status(500).json({
        success: false,
        error: error.message
      });
    }
  }
}

module.exports = new CourseController();
```

### Input Validation

```javascript
// utils/validators.js
const { body, param, query } = require('express-validator');

const courseValidation = {
  create: [
    body('title')
      .trim()
      .isLength({ min: 5, max: 255 })
      .withMessage('Title must be between 5 and 255 characters'),
    
    body('description')
      .trim()
      .isLength({ min: 10, max: 5000 })
      .withMessage('Description must be between 10 and 5000 characters'),
    
    body('category')
      .trim()
      .isLength({ min: 2, max: 100 })
      .withMessage('Category is required'),
    
    body('difficultyLevel')
      .isIn(['beginner', 'intermediate', 'advanced'])
      .withMessage('Invalid difficulty level'),
    
    body('estimatedDuration')
      .optional()
      .isInt({ min: 1 })
      .withMessage('Estimated duration must be a positive integer'),
    
    body('price')
      .optional()
      .isFloat({ min: 0 })
      .withMessage('Price must be a positive number')
  ],

  update: [
    param('id').isUUID().withMessage('Invalid course ID'),
    body('title')
      .optional()
      .trim()
      .isLength({ min: 5, max: 255 })
      .withMessage('Title must be between 5 and 255 characters'),
    
    body('description')
      .optional()
      .trim()
      .isLength({ min: 10, max: 5000 })
      .withMessage('Description must be between 10 and 5000 characters')
  ],

  getCourses: [
    query('page')
      .optional()
      .isInt({ min: 1 })
      .withMessage('Page must be a positive integer'),
    
    query('limit')
      .optional()
      .isInt({ min: 1, max: 100 })
      .withMessage('Limit must be between 1 and 100'),
    
    query('difficulty')
      .optional()
      .isIn(['beginner', 'intermediate', 'advanced'])
      .withMessage('Invalid difficulty level')
  ]
};

module.exports = { courseValidation };
```

## Real-time Features

### Socket.io Implementation

```javascript
// services/socketService.js
const socketIo = require('socket.io');
const jwt = require('jsonwebtoken');
const { User } = require('../models');

class SocketService {
  constructor() {
    this.io = null;
    this.connectedUsers = new Map();
  }

  initialize(server) {
    this.io = socketIo(server, {
      cors: {
        origin: process.env.CLIENT_URL,
        credentials: true
      }
    });

    this.io.use(this.authenticateSocket.bind(this));
    this.io.on('connection', this.handleConnection.bind(this));
  }

  async authenticateSocket(socket, next) {
    try {
      const token = socket.handshake.auth.token;
      const decoded = jwt.verify(token, process.env.JWT_SECRET);
      const user = await User.findByPk(decoded.userId);

      if (!user || !user.isActive) {
        return next(new Error('Authentication failed'));
      }

      socket.userId = user.id;
      socket.user = user;
      next();
    } catch (error) {
      next(new Error('Authentication failed'));
    }
  }

  handleConnection(socket) {
    console.log(`User ${socket.user.email} connected`);
    
    this.connectedUsers.set(socket.userId, socket.id);

    // Join course rooms based on user enrollments
    this.joinCourseRooms(socket);

    // Handle lesson progress updates
    socket.on('lesson:progress', this.handleLessonProgress.bind(this, socket));
    
    // Handle live chat in courses
    socket.on('course:message', this.handleCourseMessage.bind(this, socket));
    
    // Handle quiz participation
    socket.on('quiz:start', this.handleQuizStart.bind(this, socket));
    socket.on('quiz:submit', this.handleQuizSubmit.bind(this, socket));

    socket.on('disconnect', this.handleDisconnection.bind(this, socket));
  }

  async joinCourseRooms(socket) {
    try {
      const enrollments = await socket.user.getEnrollments({
        include: ['Course']
      });

      enrollments.forEach(enrollment => {
        socket.join(`course:${enrollment.courseId}`);
      });
    } catch (error) {
      console.error('Error joining course rooms:', error);
    }
  }

  handleLessonProgress(socket, data) {
    const { courseId, lessonId, progress } = data;
    
    // Broadcast progress to instructors in the course
    socket.to(`course:${courseId}`).emit('student:progress', {
      userId: socket.userId,
      lessonId,
      progress,
      timestamp: new Date()
    });
  }

  handleCourseMessage(socket, data) {
    const { courseId, message } = data;
    
    // Broadcast message to all participants in the course
    this.io.to(`course:${courseId}`).emit('course:message', {
      userId: socket.userId,
      userName: `${socket.user.firstName} ${socket.user.lastName}`,
      message,
      timestamp: new Date()
    });
  }

  // Notify user about course updates
  notifyUser(userId, event, data) {
    const socketId = this.connectedUsers.get(userId);
    if (socketId) {
      this.io.to(socketId).emit(event, data);
    }
  }

  // Broadcast to all users in a course
  notifyCourse(courseId, event, data) {
    this.io.to(`course:${courseId}`).emit(event, data);
  }

  handleDisconnection(socket) {
    console.log(`User ${socket.user.email} disconnected`);
    this.connectedUsers.delete(socket.userId);
  }
}

module.exports = new SocketService();
```

## Performance Optimization

### Caching Strategies

```javascript
// services/cacheService.js
const redis = require('../config/redis');

class CacheService {
  // Cache course data
  async getCourse(courseId) {
    const key = `course:${courseId}`;
    const cached = await redis.get(key);
    
    if (cached) {
      return JSON.parse(cached);
    }
    
    return null;
  }

  async setCourse(courseId, courseData, ttl = 3600) {
    const key = `course:${courseId}`;
    await redis.setex(key, ttl, JSON.stringify(courseData));
  }

  // Cache user progress
  async getUserProgress(userId, courseId) {
    const key = `progress:${userId}:${courseId}`;
    const cached = await redis.get(key);
    
    if (cached) {
      return JSON.parse(cached);
    }
    
    return null;
  }

  async setUserProgress(userId, courseId, progressData, ttl = 1800) {
    const key = `progress:${userId}:${courseId}`;
    await redis.setex(key, ttl, JSON.stringify(progressData));
  }

  // Invalidate cache
  async invalidateCourse(courseId) {
    const pattern = `course:${courseId}*`;
    const keys = await redis.keys(pattern);
    
    if (keys.length > 0) {
      await redis.del(...keys);
    }
  }

  // Cache popular courses
  async getPopularCourses() {
    const key = 'courses:popular';
    const cached = await redis.get(key);
    
    if (cached) {
      return JSON.parse(cached);
    }
    
    return null;
  }

  async setPopularCourses(courses, ttl = 7200) {
    const key = 'courses:popular';
    await redis.setex(key, ttl, JSON.stringify(courses));
  }
}

module.exports = new CacheService();
```

### Database Query Optimization

```javascript
// services/courseService.js
const { Course, Lesson, User, Enrollment, sequelize } = require('../models');
const cacheService = require('./cacheService');

class CourseService {
  async getCourses(options = {}) {
    const { page = 1, limit = 10, category, difficulty, search } = options;
    const offset = (page - 1) * limit;

    // Build where clause
    const where = {};
    if (category) where.category = category;
    if (difficulty) where.difficultyLevel = difficulty;
    if (search) {
      where[Op.or] = [
        { title: { [Op.iLike]: `%${search}%` } },
        { description: { [Op.iLike]: `%${search}%` } }
      ];
    }

    // Use include for eager loading
    const { count, rows } = await Course.findAndCountAll({
      where,
      include: [
        {
          model: User,
          as: 'instructor',
          attributes: ['id', 'firstName', 'lastName', 'profileImageUrl']
        },
        {
          model: Lesson,
          attributes: ['id'],
          required: false
        }
      ],
      limit,
      offset,
      order: [['createdAt', 'DESC']],
      distinct: true
    });

    // Add lesson count to each course
    const courses = rows.map(course => {
      const courseData = course.toJSON();
      courseData.lessonCount = course.Lessons ? course.Lessons.length : 0;
      delete courseData.Lessons;
      return courseData;
    });

    return {
      courses,
      total: count
    };
  }

  async getCourseById(courseId, user) {
    // Try cache first
    let course = await cacheService.getCourse(courseId);
    
    if (!course) {
      // Fetch from database with optimized query
      course = await Course.findByPk(courseId, {
        include: [
          {
            model: User,
            as: 'instructor',
            attributes: ['id', 'firstName', 'lastName', 'profileImageUrl']
          },
          {
            model: Lesson,
            order: [['orderIndex', 'ASC']],
            attributes: ['id', 'title', 'duration', 'lessonType', 'orderIndex']
          }
        ]
      });

      if (course) {
        // Cache for 1 hour
        await cacheService.setCourse(courseId, course.toJSON(), 3600);
      }
    }

    if (!course) {
      throw new Error('Course not found');
    }

    // If user is provided, check enrollment and progress
    if (user) {
      const enrollment = await Enrollment.findOne({
        where: { userId: user.id, courseId },
        include: [
          {
            model: LessonProgress,
            attributes: ['lessonId', 'completed', 'timeSpent']
          }
        ]
      });

      course.enrollment = enrollment;
    }

    return course;
  }

  // Bulk operations for better performance
  async bulkUpdateProgress(progressUpdates) {
    return sequelize.transaction(async (transaction) => {
      const promises = progressUpdates.map(update => 
        LessonProgress.upsert(update, { transaction })
      );
      
      return Promise.all(promises);
    });
  }
}

module.exports = new CourseService();
```

## Security Best Practices

### Security Middleware Setup

```javascript
// middleware/security.js
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const slowDown = require('express-slow-down');

// Rate limiting configuration
const createRateLimiter = (windowMs, max, message) => {
  return rateLimit({
    windowMs,
    max,
    message: { error: message },
    standardHeaders: true,
    legacyHeaders: false,
  });
};

// Different rate limits for different endpoints
const authLimiter = createRateLimiter(
  15 * 60 * 1000, // 15 minutes
  5, // 5 attempts
  'Too many authentication attempts, please try again later'
);

const apiLimiter = createRateLimiter(
  15 * 60 * 1000, // 15 minutes
  100, // 100 requests
  'Too many requests, please try again later'
);

// Slow down repeated requests
const speedLimiter = slowDown({
  windowMs: 15 * 60 * 1000, // 15 minutes
  delayAfter: 50, // allow 50 requests per windowMs without delay
  delayMs: 500 // add 500ms delay per request after delayAfter
});

// Content Security Policy
const cspOptions = {
  directives: {
    defaultSrc: ["'self'"],
    styleSrc: ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"],
    fontSrc: ["'self'", "https://fonts.gstatic.com"],
    imgSrc: ["'self'", "data:", "https:"],
    scriptSrc: ["'self'"],
    connectSrc: ["'self'"],
    mediaSrc: ["'self'", "https:"],
  },
};

module.exports = {
  helmet: helmet({
    contentSecurityPolicy: cspOptions,
    crossOriginEmbedderPolicy: false // Needed for video content
  }),
  authLimiter,
  apiLimiter,
  speedLimiter
};
```

### Input Sanitization

```javascript
// utils/sanitizer.js
const validator = require('validator');
const xss = require('xss');

class Sanitizer {
  static sanitizeString(input, options = {}) {
    if (typeof input !== 'string') return input;
    
    let sanitized = input.trim();
    
    // Remove XSS attempts
    sanitized = xss(sanitized, {
      whiteList: options.allowHTML ? {
        b: [],
        i: [],
        u: [],
        br: [],
        p: [],
        strong: [],
        em: []
      } : {}
    });
    
    // Escape HTML if not allowing HTML
    if (!options.allowHTML) {
      sanitized = validator.escape(sanitized);
    }
    
    return sanitized;
  }

  static sanitizeEmail(email) {
    if (typeof email !== 'string') return '';
    
    const sanitized = email.toLowerCase().trim();
    return validator.isEmail(sanitized) ? sanitized : '';
  }

  static sanitizeObject(obj, schema) {
    const sanitized = {};
    
    for (const [key, rules] of Object.entries(schema)) {
      if (obj.hasOwnProperty(key)) {
        const value = obj[key];
        
        switch (rules.type) {
          case 'string':
            sanitized[key] = this.sanitizeString(value, rules.options || {});
            break;
          case 'email':
            sanitized[key] = this.sanitizeEmail(value);
            break;
          case 'number':
            sanitized[key] = typeof value === 'number' ? value : parseInt(value, 10);
            break;
          case 'boolean':
            sanitized[key] = Boolean(value);
            break;
          default:
            sanitized[key] = value;
        }
      }
    }
    
    return sanitized;
  }
}

module.exports = Sanitizer;
```

## Monitoring & Logging

### Winston Logger Configuration

```javascript
// utils/logger.js
const winston = require('winston');
const path = require('path');

// Custom format for logs
const logFormat = winston.format.combine(
  winston.format.timestamp(),
  winston.format.errors({ stack: true }),
  winston.format.json()
);

// Create logger instance
const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: logFormat,
  defaultMeta: { service: 'lms-backend' },
  transports: [
    // Write all logs to console in development
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
      )
    }),
    
    // Write error logs to file
    new winston.transports.File({
      filename: path.join(__dirname, '../logs/error.log'),
      level: 'error',
      maxsize: 5242880, // 5MB
      maxFiles: 5
    }),
    
    // Write all logs to file
    new winston.transports.File({
      filename: path.join(__dirname, '../logs/combined.log'),
      maxsize: 5242880, // 5MB
      maxFiles: 5
    })
  ]
});

// Add request logging middleware
const requestLogger = (req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    
    logger.info('HTTP Request', {
      method: req.method,
      url: req.url,
      statusCode: res.statusCode,
      duration: `${duration}ms`,
      userAgent: req.get('User-Agent'),
      ip: req.ip,
      userId: req.user?.id
    });
  });
  
  next();
};

module.exports = { logger, requestLogger };
```

### Performance Monitoring

```javascript
// utils/metrics.js
const client = require('prom-client');

// Create a Registry to register metrics
const register = new client.Registry();

// Add default metrics
client.collectDefaultMetrics({ register });

// Custom metrics for LMS
const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  registers: [register]
});

const activeUsers = new client.Gauge({
  name: 'lms_active_users_total',
  help: 'Number of currently active users',
  registers: [register]
});

const courseEnrollments = new client.Counter({
  name: 'lms_course_enrollments_total',
  help: 'Total number of course enrollments',
  labelNames: ['course_id'],
  registers: [register]
});

const lessonCompletions = new client.Counter({
  name: 'lms_lesson_completions_total',
  help: 'Total number of lesson completions',
  labelNames: ['course_id', 'lesson_id'],
  registers: [register]
});

// Middleware to collect HTTP metrics
const metricsMiddleware = (req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    
    httpRequestDuration
      .labels(req.method, req.route?.path || req.path, res.statusCode)
      .observe(duration);
  });
  
  next();
};

module.exports = {
  register,
  metrics: {
    httpRequestDuration,
    activeUsers,
    courseEnrollments,
    lessonCompletions
  },
  metricsMiddleware
};
```

## Deployment & DevOps

### Docker Configuration

```dockerfile
# Dockerfile
FROM node:20-alpine AS base

# Install dependencies only when needed
FROM base AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Development image
FROM base AS dev
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev"]

# Production image
FROM base AS prod
WORKDIR /app

# Create app user
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nodejs

# Copy dependencies
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Set correct permissions
USER nodejs

EXPOSE 3000

ENV NODE_ENV=production

CMD ["npm", "start"]
```

### Docker Compose for Development

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build:
      context: .
      target: dev
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgres://postgres:password@postgres:5432/lms_dev
      - REDIS_URL=redis://redis:6379
    volumes:
      - .:/app
      - /app/node_modules
    depends_on:
      - postgres
      - redis
    restart: unless-stopped

  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=lms_dev
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - app
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
```

### Kubernetes Deployment

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lms-backend
  labels:
    app: lms-backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: lms-backend
  template:
    metadata:
      labels:
        app: lms-backend
    spec:
      containers:
      - name: lms-backend
        image: your-registry/lms-backend:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: lms-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: lms-secrets
              key: redis-url
        resources:
          limits:
            cpu: "1"
            memory: "512Mi"
          requests:
            cpu: "0.5"
            memory: "256Mi"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: lms-backend-service
spec:
  selector:
    app: lms-backend
  ports:
  - port: 80
    targetPort: 3000
  type: ClusterIP

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: lms-backend-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - api.yourlms.com
    secretName: lms-backend-tls
  rules:
  - host: api.yourlms.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: lms-backend-service
            port:
              number: 80
```

## Conclusion

This comprehensive guide provides a solid foundation for building scalable LMS backends with Node.js. Key takeaways:

1. **Start Simple**: Begin with a monolithic architecture and scale to microservices as needed
2. **Performance First**: Implement caching, optimize queries, and monitor metrics from day one
3. **Security by Design**: Use proper authentication, input validation, and rate limiting
4. **Real-time Features**: Leverage Socket.io for live updates and engagement
5. **Observability**: Implement comprehensive logging and monitoring
6. **Scalable Infrastructure**: Use Docker and Kubernetes for deployment flexibility

Remember to adapt these patterns to your specific requirements and always test thoroughly before deploying to production. 