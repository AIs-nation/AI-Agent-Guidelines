# Express.js Backend Patterns for LMS Development

## Educational Platform API Architecture

### LMS Backend Structure for Educational Systems
✅ **Good Example: Production-Ready LMS Backend Architecture**
```
🎓 EXPRESS.JS LMS BACKEND ARCHITECTURE

project-root/
├── server.js                          # Main application entry point
├── config/
│   ├── database.js                    # Database connection & configuration
│   ├── auth.js                        # Authentication strategies & JWT setup
│   ├── ai-services.js                 # Claude/DALL-E API configuration
│   └── environment.js                 # Environment-based configuration
├── controllers/
│   ├── courseController.js            # Course CRUD operations & generation
│   ├── lessonController.js            # Lesson management & content delivery
│   ├── progressController.js          # Learning progress tracking APIs
│   ├── userController.js              # User management & authentication
│   └── analyticsController.js         # Educational analytics & reporting
├── middleware/
│   ├── auth.js                        # JWT verification & role-based access
│   ├── validation.js                  # Input validation for educational data
│   ├── errorHandler.js                # Educational platform error handling
│   ├── rateLimit.js                   # API rate limiting for course generation
│   └── logging.js                     # Educational activity logging
├── models/
│   ├── Course.js                      # Course schema with educational metadata
│   ├── Lesson.js                      # Lesson structure with sections
│   ├── User.js                        # User profiles with learning preferences
│   ├── Progress.js                    # Progress tracking with time analytics
│   └── Analytics.js                   # Educational metrics & performance data
├── routes/
│   ├── api/
│   │   ├── courses.js                 # Course management endpoints
│   │   ├── lessons.js                 # Lesson delivery & navigation
│   │   ├── progress.js                # Progress tracking & completion
│   │   ├── auth.js                    # Authentication & user management
│   │   └── analytics.js               # Educational reporting endpoints
│   └── index.js                       # Route aggregation & API versioning
├── services/
│   ├── aiContentService.js            # Claude API integration for course content
│   ├── imageGenerationService.js      # DALL-E integration for thumbnails
│   ├── progressTrackingService.js     # Learning analytics calculations
│   ├── emailService.js                # Educational notifications
│   └── cacheService.js                # Course content caching
└── utils/
    ├── courseValidation.js            # Educational content validation
    ├── progressCalculations.js        # Learning metrics calculations
    ├── contentSanitization.js         # Educational content safety
    └── learningAnalytics.js           # Advanced educational analytics
```

### Express.js Application Setup for Educational Platforms
✅ **Good Example: LMS Express Configuration**
```javascript
// server.js - Production-ready LMS backend setup
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const compression = require('compression');
const morgan = require('morgan');

const { connectDatabase } = require('./config/database');
const { setupAuth } = require('./config/auth');
const { initializeAIServices } = require('./config/ai-services');
const routes = require('./routes');
const { errorHandler, notFound } = require('./middleware/errorHandler');
const { setupLogging } = require('./middleware/logging');

const app = express();
const PORT = process.env.PORT || 5000;

// Educational Platform Security Configuration
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'", "fonts.googleapis.com"],
      fontSrc: ["'self'", "fonts.gstatic.com"],
      imgSrc: ["'self'", "data:", "https:"], // Allow AI-generated images
      scriptSrc: ["'self'"],
      connectSrc: ["'self'", process.env.FRONTEND_URL]
    }
  }
}));

// CORS Configuration for Educational Platform
app.use(cors({
  origin: process.env.NODE_ENV === 'production' 
    ? [process.env.FRONTEND_URL, process.env.ADMIN_URL]
    : ['http://localhost:3000', 'http://localhost:3001'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-User-Role']
}));

// Rate Limiting for Educational Platform APIs
const createLMSRateLimit = (windowMs, max, message) => rateLimit({
  windowMs,
  max,
  message: { error: message },
  standardHeaders: true,
  legacyHeaders: false,
  keyGenerator: (req) => {
    // Different rate limits based on user role
    const userRole = req.headers['x-user-role'] || 'anonymous';
    return `${req.ip}-${userRole}`;
  }
});

// Educational Platform Specific Rate Limits
app.use('/api/courses/generate', createLMSRateLimit(
  15 * 60 * 1000, // 15 minutes
  5, // 5 course generations per 15 minutes
  'Course generation rate limit exceeded. Educational content creation is limited to ensure quality.'
));

app.use('/api/progress', createLMSRateLimit(
  60 * 1000, // 1 minute
  100, // 100 progress updates per minute
  'Progress tracking rate limit exceeded. Please slow down learning activity updates.'
));

app.use('/api/', createLMSRateLimit(
  15 * 60 * 1000, // 15 minutes
  1000, // 1000 general API calls per 15 minutes
  'API rate limit exceeded. Please reduce request frequency.'
));

// Performance Optimization for Educational Content
app.use(compression({
  level: 6,
  threshold: 1024,
  filter: (req, res) => {
    if (req.headers['x-no-compression']) return false;
    return compression.filter(req, res);
  }
}));

// Request Processing Middleware
app.use(express.json({ 
  limit: '10mb', // Allow large course content uploads
  verify: (req, res, buf) => {
    req.rawBody = buf;
  }
}));
app.use(express.urlencoded({ extended: true, limit: '10mb' }));

// Educational Activity Logging
setupLogging(app);

// Health Check for Educational Platform
app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'healthy',
    timestamp: new Date().toISOString(),
    environment: process.env.NODE_ENV,
    services: {
      database: 'connected',
      aiServices: 'operational',
      cache: 'active'
    }
  });
});

// API Routes
app.use('/api', routes);

// Educational Platform Error Handling
app.use(notFound);
app.use(errorHandler);

// Graceful Shutdown for Educational Platform
const gracefulShutdown = (signal) => {
  console.log(`Received ${signal}. Graceful shutdown initiated...`);
  server.close(() => {
    console.log('HTTP server closed.');
    // Close database connections
    connectDatabase.close(() => {
      console.log('Database connection closed.');
      process.exit(0);
    });
  });
};

// Start Educational Platform Server
const startLMSServer = async () => {
  try {
    // Initialize core services
    await connectDatabase();
    await setupAuth();
    await initializeAIServices();
    
    const server = app.listen(PORT, () => {
      console.log(`🎓 LMS Backend Server running on port ${PORT}`);
      console.log(`📚 Educational Platform API: http://localhost:${PORT}/api`);
      console.log(`🔍 Health Check: http://localhost:${PORT}/health`);
    });

    // Graceful shutdown handlers
    process.on('SIGTERM', () => gracefulShutdown('SIGTERM'));
    process.on('SIGINT', () => gracefulShutdown('SIGINT'));

    return server;
  } catch (error) {
    console.error('Failed to start LMS server:', error);
    process.exit(1);
  }
};

module.exports = { app, startLMSServer };
```

❌ **Bad Example: Poor Express.js Setup for Educational Platform**
```javascript
// ❌ Poor LMS backend setup
const express = require('express');
const app = express();

// ❌ No security headers for educational data
// ❌ No CORS configuration
// ❌ No rate limiting (vulnerable to course generation abuse)
// ❌ No compression (poor performance for educational content)
// ❌ No error handling for educational workflows

app.use(express.json()); // ❌ No size limits (vulnerable to large uploads)

// ❌ No structured route organization
app.get('/courses', (req, res) => {
  // ❌ Direct database queries in routes
  // ❌ No authentication or authorization
  // ❌ No input validation
  res.json({ courses: [] });
});

app.listen(3000); // ❌ No graceful shutdown, no health checks
```

## Course Management API Patterns

### Course Generation with AI Integration
✅ **Good Example: Professional Course Generation Endpoint**
```javascript
// controllers/courseController.js - Production course generation
const { body, validationResult } = require('express-validator');
const aiContentService = require('../services/aiContentService');
const imageGenerationService = require('../services/imageGenerationService');
const Course = require('../models/Course');
const { v4: uuidv4 } = require('uuid');

// Course Generation with Real-time Progress Tracking
const generateCourse = async (req, res) => {
  try {
    // Input validation for educational content
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({
        error: 'Course generation validation failed',
        details: errors.array(),
        educationalContext: 'Please provide valid educational parameters'
      });
    }

    const { title, description, difficulty, objectives, estimatedDuration } = req.body;
    const userId = req.user.id;
    
    // Generate unique course ID for tracking
    const courseId = uuidv4();
    
    // Set up Server-Sent Events for real-time progress
    res.writeHead(200, {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive',
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Headers': 'Cache-Control'
    });

    const sendProgress = (progress, message, data = null) => {
      const eventData = JSON.stringify({
        courseId,
        progress,
        message,
        timestamp: new Date().toISOString(),
        data
      });
      res.write(`data: ${eventData}\n\n`);
    };

    // Progress tracking throughout generation
    sendProgress(10, 'Initializing course generation for educational content...');

    // Educational content structure planning
    const courseStructure = await aiContentService.planCourseStructure({
      title,
      description,
      difficulty,
      objectives,
      estimatedDuration
    });
    
    sendProgress(25, 'Course structure planned with educational objectives');

    // Generate course thumbnail with DALL-E
    sendProgress(30, 'Creating course thumbnail with AI image generation...');
    const thumbnailUrl = await imageGenerationService.generateCourseThumbnail({
      title,
      difficulty,
      subject: courseStructure.subject
    });

    sendProgress(40, 'Course thumbnail generated successfully');

    // Generate lessons with educational content
    const lessons = [];
    const totalLessons = courseStructure.lessons.length;

    for (let i = 0; i < courseStructure.lessons.length; i++) {
      const lessonPlan = courseStructure.lessons[i];
      
      sendProgress(
        40 + (i * 40 / totalLessons), 
        `Generating lesson ${i + 1}/${totalLessons}: ${lessonPlan.title}`
      );

      const lesson = await aiContentService.generateLesson({
        ...lessonPlan,
        courseContext: { title, difficulty, objectives },
        lessonIndex: i + 1,
        totalLessons
      });

      lessons.push({
        id: uuidv4(),
        ...lesson,
        order: i + 1,
        estimatedDuration: lessonPlan.estimatedDuration
      });
    }

    sendProgress(85, 'All lessons generated with educational content');

    // Create course database record
    const course = new Course({
      id: courseId,
      title,
      description,
      difficulty,
      objectives,
      thumbnailUrl,
      lessons,
      estimatedDuration,
      createdBy: userId,
      generatedAt: new Date(),
      status: 'active',
      metadata: {
        aiGenerated: true,
        aiModel: 'claude-3-sonnet',
        generationVersion: '1.0',
        educationalFramework: 'competency-based'
      }
    });

    await course.save();
    
    sendProgress(95, 'Course saved to database with educational metadata');

    // Final success message with course data
    sendProgress(100, 'Course generation completed successfully!', {
      courseId,
      title,
      lessonsCount: lessons.length,
      estimatedDuration,
      accessUrl: `/courses/${courseId}`
    });

    // End the SSE connection
    res.write('event: complete\n');
    res.write(`data: ${JSON.stringify({ courseId, status: 'completed' })}\n\n`);
    res.end();

  } catch (error) {
    console.error('Course generation error:', error);
    
    // Error handling with educational context
    const errorResponse = JSON.stringify({
      error: 'Course generation failed',
      message: error.message,
      educationalImpact: 'Course creation was interrupted. Please try again.',
      supportContact: 'educational-support@platform.com'
    });
    
    res.write(`event: error\n`);
    res.write(`data: ${errorResponse}\n\n`);
    res.end();
  }
};

// Course Generation Validation Middleware
const validateCourseGeneration = [
  body('title')
    .isLength({ min: 10, max: 100 })
    .withMessage('Course title must be between 10-100 characters')
    .matches(/^[a-zA-Z0-9\s\-:,().]+$/)
    .withMessage('Course title contains invalid characters for educational content'),
  
  body('description')
    .isLength({ min: 50, max: 500 })
    .withMessage('Course description must be between 50-500 characters for proper educational context'),
  
  body('difficulty')
    .isIn(['beginner', 'intermediate', 'advanced'])
    .withMessage('Difficulty must be appropriate for educational levels'),
  
  body('objectives')
    .isArray({ min: 3, max: 10 })
    .withMessage('Course must have 3-10 learning objectives')
    .custom((objectives) => {
      return objectives.every(obj => 
        typeof obj === 'string' && 
        obj.length >= 10 && 
        obj.length <= 200
      );
    })
    .withMessage('Each learning objective must be 10-200 characters'),
  
  body('estimatedDuration')
    .isInt({ min: 30, max: 480 })
    .withMessage('Course duration must be between 30 minutes and 8 hours')
];

// Get Course with Educational Analytics
const getCourse = async (req, res) => {
  try {
    const { courseId } = req.params;
    const userId = req.user?.id;

    const course = await Course.findById(courseId)
      .populate('lessons')
      .lean();

    if (!course) {
      return res.status(404).json({
        error: 'Course not found',
        educationalContext: 'The requested educational content is not available'
      });
    }

    // Add user progress if authenticated
    let userProgress = null;
    if (userId) {
      userProgress = await Progress.findOne({
        userId,
        courseId
      }).lean();
    }

    // Educational analytics calculation
    const analytics = {
      totalLessons: course.lessons.length,
      totalSections: course.lessons.reduce((sum, lesson) => sum + lesson.sections.length, 0),
      estimatedCompletionTime: course.estimatedDuration,
      difficultyLevel: course.difficulty,
      learningObjectives: course.objectives.length,
      contentReadiness: course.status === 'active' ? 100 : 0
    };

    res.json({
      course: {
        ...course,
        analytics,
        userProgress: userProgress ? {
          completedSections: userProgress.completedSections.length,
          completionPercentage: userProgress.completionPercentage,
          timeSpent: userProgress.timeSpent,
          lastAccessedAt: userProgress.lastAccessedAt
        } : null
      }
    });

  } catch (error) {
    console.error('Get course error:', error);
    res.status(500).json({
      error: 'Failed to retrieve educational content',
      educationalImpact: 'Course data temporarily unavailable'
    });
  }
};

module.exports = {
  generateCourse,
  validateCourseGeneration,
  getCourse
};
```

## Progress Tracking API Architecture

### Learning Analytics and Progress Management
✅ **Good Example: Comprehensive Progress Tracking System**
```javascript
// controllers/progressController.js - Educational progress tracking
const Progress = require('../models/Progress');
const Course = require('../models/Course');
const { calculateLearningAnalytics } = require('../utils/learningAnalytics');

// Section Completion with Time Tracking
const completeSection = async (req, res) => {
  try {
    const { courseId, lessonId, sectionId } = req.params;
    const { timeSpent, interactions, notes } = req.body;
    const userId = req.user.id;

    // Validate educational context
    const course = await Course.findById(courseId);
    if (!course) {
      return res.status(404).json({
        error: 'Course not found',
        educationalContext: 'Cannot track progress for non-existent educational content'
      });
    }

    const lesson = course.lessons.find(l => l.id === lessonId);
    if (!lesson) {
      return res.status(404).json({
        error: 'Lesson not found',
        educationalContext: 'Cannot track progress for invalid lesson'
      });
    }

    const section = lesson.sections.find(s => s.id === sectionId);
    if (!section) {
      return res.status(404).json({
        error: 'Section not found',
        educationalContext: 'Cannot track progress for invalid section'
      });
    }

    // Find or create progress record
    let progress = await Progress.findOne({ userId, courseId });
    
    if (!progress) {
      progress = new Progress({
        userId,
        courseId,
        courseName: course.title,
        enrolledAt: new Date(),
        completedSections: [],
        timeSpent: 0,
        interactions: [],
        learningPath: []
      });
    }

    // Check if section already completed
    const existingCompletion = progress.completedSections.find(
      cs => cs.lessonId === lessonId && cs.sectionId === sectionId
    );

    if (existingCompletion) {
      return res.status(409).json({
        error: 'Section already completed',
        educationalContext: 'This learning section has already been marked as complete',
        progress: {
          completedAt: existingCompletion.completedAt,
          timeSpent: existingCompletion.timeSpent
        }
      });
    }

    // Record section completion
    const completionRecord = {
      lessonId,
      sectionId,
      lessonTitle: lesson.title,
      sectionTitle: section.title,
      completedAt: new Date(),
      timeSpent: Math.max(timeSpent || 0, 30), // Minimum 30 seconds for valid learning
      interactions: interactions || [],
      learningNotes: notes || ''
    };

    progress.completedSections.push(completionRecord);
    progress.timeSpent += completionRecord.timeSpent;
    progress.lastAccessedAt = new Date();

    // Calculate educational progress metrics
    const totalSections = course.lessons.reduce((sum, l) => sum + l.sections.length, 0);
    progress.completionPercentage = Math.round(
      (progress.completedSections.length / totalSections) * 100
    );

    // Update learning path with educational context
    progress.learningPath.push({
      timestamp: new Date(),
      action: 'section_completed',
      lessonId,
      sectionId,
      timeSpent: completionRecord.timeSpent,
      learningVelocity: calculateLearningVelocity(progress),
      difficultyLevel: section.difficulty || lesson.difficulty || course.difficulty
    });

    // Check for lesson completion
    const lessonSections = lesson.sections.map(s => s.id);
    const completedLessonSections = progress.completedSections
      .filter(cs => cs.lessonId === lessonId)
      .map(cs => cs.sectionId);
    
    const lessonCompleted = lessonSections.every(
      sectionId => completedLessonSections.includes(sectionId)
    );

    if (lessonCompleted && !progress.completedLessons.includes(lessonId)) {
      progress.completedLessons.push(lessonId);
      
      // Lesson completion achievement
      progress.achievements.push({
        type: 'lesson_completed',
        lessonId,
        lessonTitle: lesson.title,
        completedAt: new Date(),
        timeToComplete: calculateLessonCompletionTime(progress, lessonId)
      });
    }

    // Check for course completion
    const allLessons = course.lessons.map(l => l.id);
    const courseCompleted = allLessons.every(
      lessonId => progress.completedLessons.includes(lessonId)
    );

    if (courseCompleted && !progress.courseCompleted) {
      progress.courseCompleted = true;
      progress.courseCompletedAt = new Date();
      
      // Course completion achievement
      progress.achievements.push({
        type: 'course_completed',
        courseId,
        courseName: course.title,
        completedAt: new Date(),
        totalTimeSpent: progress.timeSpent,
        finalScore: calculateFinalScore(progress)
      });
    }

    await progress.save();

    // Return educational progress response
    res.json({
      success: true,
      educationalProgress: {
        sectionCompleted: {
          lessonTitle: lesson.title,
          sectionTitle: section.title,
          timeSpent: completionRecord.timeSpent
        },
        overallProgress: {
          completionPercentage: progress.completionPercentage,
          sectionsCompleted: progress.completedSections.length,
          totalSections,
          lessonsCompleted: progress.completedLessons.length,
          totalLessons: course.lessons.length,
          timeSpent: progress.timeSpent,
          estimatedTimeRemaining: calculateEstimatedTimeRemaining(progress, course)
        },
        achievements: lessonCompleted || courseCompleted ? {
          lessonCompleted,
          courseCompleted,
          newAchievements: progress.achievements.slice(-1)
        } : null,
        nextRecommendation: getNextLearningRecommendation(progress, course)
      }
    });

  } catch (error) {
    console.error('Progress tracking error:', error);
    res.status(500).json({
      error: 'Failed to track learning progress',
      educationalImpact: 'Your learning progress was not saved. Please try completing the section again.'
    });
  }
};

// Batch Progress Updates for Performance
const batchUpdateProgress = async (req, res) => {
  try {
    const { courseId } = req.params;
    const { updates } = req.body;
    const userId = req.user.id;

    if (!Array.isArray(updates) || updates.length === 0) {
      return res.status(400).json({
        error: 'Invalid batch update format',
        educationalContext: 'Please provide valid learning progress updates'
      });
    }

    const results = [];
    let progress = await Progress.findOne({ userId, courseId });

    if (!progress) {
      const course = await Course.findById(courseId);
      progress = new Progress({
        userId,
        courseId,
        courseName: course?.title,
        enrolledAt: new Date(),
        completedSections: [],
        timeSpent: 0,
        interactions: [],
        learningPath: []
      });
    }

    // Process educational updates in batch
    for (const update of updates) {
      try {
        switch (update.type) {
          case 'section_completed':
            // Process section completion
            const existingCompletion = progress.completedSections.find(
              cs => cs.lessonId === update.lessonId && cs.sectionId === update.sectionId
            );

            if (!existingCompletion) {
              progress.completedSections.push({
                lessonId: update.lessonId,
                sectionId: update.sectionId,
                completedAt: new Date(update.timestamp),
                timeSpent: update.timeSpent || 0
              });
              results.push({ type: 'section_completed', status: 'success' });
            } else {
              results.push({ type: 'section_completed', status: 'already_completed' });
            }
            break;

          case 'interaction':
            // Track learning interactions
            progress.interactions.push({
              type: update.interactionType,
              timestamp: new Date(update.timestamp),
              data: update.data
            });
            results.push({ type: 'interaction', status: 'recorded' });
            break;

          case 'time_update':
            // Update time spent
            progress.timeSpent += update.timeSpent || 0;
            results.push({ type: 'time_update', status: 'updated' });
            break;
        }
      } catch (updateError) {
        results.push({ 
          type: update.type, 
          status: 'error', 
          error: updateError.message 
        });
      }
    }

    progress.lastAccessedAt = new Date();
    await progress.save();

    res.json({
      success: true,
      batchResults: results,
      updatedProgress: {
        completionPercentage: progress.completionPercentage,
        sectionsCompleted: progress.completedSections.length,
        timeSpent: progress.timeSpent
      }
    });

  } catch (error) {
    console.error('Batch progress update error:', error);
    res.status(500).json({
      error: 'Failed to process batch learning progress updates',
      educationalImpact: 'Some learning progress may not have been saved'
    });
  }
};

// Educational Analytics Helper Functions
const calculateLearningVelocity = (progress) => {
  if (progress.completedSections.length < 2) return 0;
  
  const recentSections = progress.completedSections.slice(-5);
  const totalTime = recentSections.reduce((sum, cs) => sum + cs.timeSpent, 0);
  
  return recentSections.length / (totalTime / 60000); // sections per minute
};

const calculateEstimatedTimeRemaining = (progress, course) => {
  const totalSections = course.lessons.reduce((sum, l) => sum + l.sections.length, 0);
  const remainingSections = totalSections - progress.completedSections.length;
  
  if (progress.completedSections.length === 0) {
    return course.estimatedDuration * 60; // Convert to seconds
  }
  
  const averageTimePerSection = progress.timeSpent / progress.completedSections.length;
  return Math.round(remainingSections * averageTimePerSection);
};

const getNextLearningRecommendation = (progress, course) => {
  // Find next uncompleted section
  for (const lesson of course.lessons) {
    for (const section of lesson.sections) {
      const isCompleted = progress.completedSections.some(
        cs => cs.lessonId === lesson.id && cs.sectionId === section.id
      );
      
      if (!isCompleted) {
        return {
          lessonId: lesson.id,
          lessonTitle: lesson.title,
          sectionId: section.id,
          sectionTitle: section.title,
          estimatedDuration: section.estimatedDuration
        };
      }
    }
  }
  
  return null; // Course completed
};

module.exports = {
  completeSection,
  batchUpdateProgress
};
```

## AI Integration Patterns for Educational Content

### Claude API Integration for Course Generation
✅ **Good Example: Professional AI Service Integration**
```javascript
// services/aiContentService.js - Educational AI content generation
const { Anthropic } = require('@anthropic-ai/sdk');
const anthropic = new Anthropic({
  apiKey: process.env.CLAUDE_API_KEY
});

class AIContentService {
  constructor() {
    this.maxRetries = 3;
    this.retryDelay = 1000;
  }

  // Course Structure Planning with Educational Framework
  async planCourseStructure({ title, description, difficulty, objectives, estimatedDuration }) {
    const prompt = `As an expert educational content designer, create a comprehensive course structure for:

COURSE DETAILS:
- Title: ${title}
- Description: ${description}
- Difficulty Level: ${difficulty}
- Learning Objectives: ${objectives.join(', ')}
- Estimated Duration: ${estimatedDuration} minutes

REQUIREMENTS:
1. Create 4-8 lessons that progressively build knowledge
2. Each lesson should have 2-3 sections (10-15 minutes each)
3. Ensure proper educational scaffolding from basic to advanced concepts
4. Include practical applications and real-world examples
5. Design for ${difficulty} level learners with appropriate complexity

EDUCATIONAL FRAMEWORK:
- Use Bloom's Taxonomy for learning objectives
- Include formative assessment opportunities
- Ensure content accessibility and inclusivity
- Design for active learning and engagement

Please respond with a JSON structure containing the course plan.`;

    try {
      const response = await this.callClaudeWithRetry(prompt, {
        max_tokens: 4000,
        temperature: 0.7
      });

      return this.validateAndParseEducationalContent(response.content[0].text, 'course_structure');
    } catch (error) {
      console.error('Course structure planning error:', error);
      throw new Error('Failed to plan educational course structure');
    }
  }

  // Lesson Content Generation with Educational Standards
  async generateLesson({ title, description, objectives, sections, courseContext, lessonIndex, totalLessons }) {
    const prompt = `As an expert educational content creator, generate comprehensive lesson content for:

LESSON CONTEXT:
- Lesson Title: ${title}
- Description: ${description}
- Learning Objectives: ${objectives.join(', ')}
- Course Context: ${courseContext.title} (${courseContext.difficulty} level)
- Lesson Position: ${lessonIndex} of ${totalLessons}

SECTION REQUIREMENTS:
Generate content for ${sections.length} sections:
${sections.map((section, index) => `
Section ${index + 1}: ${section.title}
- Duration: ${section.estimatedDuration} minutes
- Focus: ${section.focus}
- Learning Outcome: ${section.learningOutcome}
`).join('')}

EDUCATIONAL STANDARDS:
1. Use clear, engaging explanations appropriate for ${courseContext.difficulty} learners
2. Include practical examples and real-world applications
3. Provide step-by-step instructions where applicable
4. Design interactive elements and knowledge checks
5. Ensure content accessibility (reading level, visual descriptions)
6. Include reflection questions and application exercises

CONTENT STRUCTURE:
For each section, provide:
- Engaging introduction that connects to previous learning
- Main content with clear explanations and examples
- Practice activities or knowledge checks
- Summary that reinforces key concepts
- Connection to next section or lesson

Please respond with detailed educational content in JSON format.`;

    try {
      const response = await this.callClaudeWithRetry(prompt, {
        max_tokens: 6000,
        temperature: 0.8
      });

      const lessonContent = this.validateAndParseEducationalContent(response.content[0].text, 'lesson_content');
      
      // Add educational metadata
      return {
        ...lessonContent,
        metadata: {
          generatedAt: new Date().toISOString(),
          aiModel: 'claude-3-sonnet',
          educationalLevel: courseContext.difficulty,
          estimatedReadingTime: this.calculateReadingTime(lessonContent),
          accessibilityFeatures: this.generateAccessibilityMetadata(lessonContent)
        }
      };
    } catch (error) {
      console.error('Lesson generation error:', error);
      throw new Error(`Failed to generate educational content for lesson: ${title}`);
    }
  }

  // Claude API Call with Educational Context Error Handling
  async callClaudeWithRetry(prompt, options = {}, retryCount = 0) {
    try {
      const response = await anthropic.messages.create({
        model: 'claude-3-sonnet-20240229',
        max_tokens: options.max_tokens || 4000,
        temperature: options.temperature || 0.7,
        messages: [{
          role: 'user',
          content: prompt
        }],
        system: "You are an expert educational content designer with deep knowledge of learning sciences, instructional design, and accessibility standards. Always create content that is engaging, accurate, and pedagogically sound."
      });

      return response;
    } catch (error) {
      console.error(`Claude API error (attempt ${retryCount + 1}):`, error);
      
      if (retryCount < this.maxRetries) {
        await new Promise(resolve => setTimeout(resolve, this.retryDelay * (retryCount + 1)));
        return this.callClaudeWithRetry(prompt, options, retryCount + 1);
      }
      
      throw new Error(`Educational content generation failed after ${this.maxRetries} attempts: ${error.message}`);
    }
  }

  // Educational Content Validation
  validateAndParseEducationalContent(content, type) {
    try {
      // Extract JSON from Claude's response
      const jsonMatch = content.match(/```json\n([\s\S]*?)\n```/) || content.match(/\{[\s\S]*\}/);
      
      if (!jsonMatch) {
        throw new Error('No valid JSON found in educational content response');
      }

      const parsedContent = JSON.parse(jsonMatch[1] || jsonMatch[0]);
      
      // Validate educational content structure
      if (type === 'course_structure') {
        this.validateCourseStructure(parsedContent);
      } else if (type === 'lesson_content') {
        this.validateLessonContent(parsedContent);
      }

      return parsedContent;
    } catch (error) {
      console.error('Educational content validation error:', error);
      throw new Error(`Invalid educational content format: ${error.message}`);
    }
  }

  validateCourseStructure(structure) {
    if (!structure.lessons || !Array.isArray(structure.lessons)) {
      throw new Error('Course structure must include lessons array');
    }

    if (structure.lessons.length < 3 || structure.lessons.length > 12) {
      throw new Error('Course must have 3-12 lessons for effective learning');
    }

    structure.lessons.forEach((lesson, index) => {
      if (!lesson.title || !lesson.description || !lesson.sections) {
        throw new Error(`Lesson ${index + 1} missing required educational components`);
      }

      if (!Array.isArray(lesson.sections) || lesson.sections.length < 2 || lesson.sections.length > 4) {
        throw new Error(`Lesson ${index + 1} must have 2-4 sections for optimal learning`);
      }
    });
  }

  validateLessonContent(content) {
    if (!content.sections || !Array.isArray(content.sections)) {
      throw new Error('Lesson content must include sections array');
    }

    content.sections.forEach((section, index) => {
      if (!section.title || !section.content) {
        throw new Error(`Section ${index + 1} missing required educational content`);
      }

      if (typeof section.content !== 'string' || section.content.length < 100) {
        throw new Error(`Section ${index + 1} content too brief for effective learning`);
      }
    });
  }

  // Educational Accessibility Features
  calculateReadingTime(content) {
    const wordsPerMinute = 200;
    const totalWords = JSON.stringify(content).split(/\s+/).length;
    return Math.ceil(totalWords / wordsPerMinute);
  }

  generateAccessibilityMetadata(content) {
    return {
      hasHeadings: true,
      hasStructuredContent: true,
      estimatedReadingLevel: 'college', // Could be enhanced with actual analysis
      hasInteractiveElements: content.sections?.some(s => s.activities) || false,
      supportsMobileDevices: true,
      hasVisualDescriptions: true
    };
  }
}

module.exports = new AIContentService();
```

## Key Express.js LMS Development Takeaways

### Critical Backend Patterns for Educational Platforms ✅

1. **Educational API Architecture**
   - Structure Express applications for educational workflows
   - Implement proper rate limiting for course generation APIs
   - Use Server-Sent Events for real-time progress tracking
   - Configure security headers appropriate for educational data
   - Set up educational-specific middleware and error handling

2. **Course Management APIs**
   - Design course generation with AI integration and progress tracking
   - Implement proper validation for educational content inputs
   - Use real-time progress updates during course creation
   - Handle educational content errors with appropriate user messaging
   - Structure course data with educational metadata and analytics

3. **Progress Tracking Systems**
   - Build comprehensive learning analytics with time tracking
   - Implement batch progress updates for performance optimization
   - Calculate educational metrics (completion percentage, learning velocity)
   - Track learning achievements and milestone completion
   - Provide next learning recommendations based on progress

4. **AI Service Integration**
   - Integrate Claude API for educational content generation
   - Implement proper error handling and retry logic for AI services
   - Validate generated educational content for quality and structure
   - Use educational prompts with pedagogical frameworks
   - Generate accessibility metadata for inclusive learning

5. **Educational Data Security**
   - Implement role-based access control for educational roles
   - Secure educational data with appropriate CORS and helmet configuration
   - Rate limit educational APIs to prevent abuse
   - Log educational activities for compliance and analytics
   - Handle educational data privacy requirements

### Express.js LMS Anti-Patterns to Avoid ❌

1. **Poor API Structure** - Mixing business logic in routes, no educational context validation
2. **Inadequate Security** - Missing rate limiting on course generation, no educational data protection
3. **Poor Error Handling** - Generic errors without educational context or user guidance
4. **Inefficient Progress Tracking** - Individual API calls instead of batch updates, no educational analytics
5. **Weak AI Integration** - No retry logic, poor error handling, missing content validation

Express.js development for educational platforms requires specialized attention to learning workflows, progress tracking, content generation, and educational data security while maintaining performance and scalability for growing student populations. 