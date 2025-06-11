# Educational Platform Security Architecture for LMS

## Comprehensive Security Framework for Learning Management Systems

### Multi-Layer Security Architecture
✅ **Good Example: Defense-in-Depth Security Model**
```
┌─────────────────────────────────────────────────────────────┐
│                 FRONTEND SECURITY LAYER                      │
├─────────────────────────────────────────────────────────────┤
│ • Content Security Policy (CSP)                             │
│ • XSS Protection Headers                                    │
│ • Input Validation & Sanitization                          │
│ • Authentication State Management                           │
│ • Secure Session Handling                                  │
└─────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────┐
│                   API SECURITY LAYER                        │
├─────────────────────────────────────────────────────────────┤
│ • JWT Token Authentication                                  │
│ • Rate Limiting & DDoS Protection                          │
│ • API Input Validation                                     │
│ • Request/Response Encryption                              │
│ • CORS Policy Enforcement                                  │
└─────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────┐
│                APPLICATION SECURITY LAYER                   │
├─────────────────────────────────────────────────────────────┤
│ • Role-Based Access Control (RBAC)                         │
│ • Business Logic Validation                                │
│ • Secure File Upload/Download                              │
│ • AI Content Sanitization                                 │
│ • Progress Data Integrity                                  │
└─────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────┐
│                 DATABASE SECURITY LAYER                     │
├─────────────────────────────────────────────────────────────┤
│ • Encrypted Data at Rest                                   │
│ • Parameterized Queries (SQL Injection Prevention)        │
│ • Database Access Controls                                 │
│ • Audit Logging                                           │
│ • Backup Encryption                                       │
└─────────────────────────────────────────────────────────────┘
```

### Security Configuration for LMS Backend
✅ **Good Example: Production-Ready Security Setup**
```javascript
// middleware/security.js - Comprehensive security middleware
import helmet from 'helmet';
import rateLimit from 'express-rate-limit';
import cors from 'cors';
import { body, validationResult } from 'express-validator';
import DOMPurify from 'isomorphic-dompurify';
import jwt from 'jsonwebtoken';

// Content Security Policy for educational platform
export const contentSecurityPolicy = helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: [
        "'self'",
        "'unsafe-inline'", // Needed for AI-generated educational content
        "https://api.anthropic.com",
        "https://api.openai.com",
        "https://cdnjs.cloudflare.com" // For educational libraries
      ],
      styleSrc: [
        "'self'",
        "'unsafe-inline'", // For dynamic educational styling
        "https://fonts.googleapis.com"
      ],
      imgSrc: [
        "'self'",
        "data:",
        "blob:",
        "https:", // For AI-generated images and educational content
        "https://dalle-api.openai.com"
      ],
      mediaSrc: [
        "'self'",
        "blob:",
        "https:" // For educational video content
      ],
      connectSrc: [
        "'self'",
        "https://api.anthropic.com",
        "https://api.openai.com",
        "wss:" // For real-time learning features
      ],
      fontSrc: [
        "'self'",
        "https://fonts.gstatic.com"
      ],
      objectSrc: ["'none'"],
      upgradeInsecureRequests: []
    }
  },
  crossOriginEmbedderPolicy: false // Allows educational content embedding
});

// Rate limiting for different LMS endpoints
export const createEducationalRateLimit = () => {
  return {
    // Course generation - more restrictive due to AI cost
    courseGeneration: rateLimit({
      windowMs: 15 * 60 * 1000, // 15 minutes
      max: 5, // 5 course generations per window
      message: {
        error: 'Too many course generation requests',
        retryAfter: 15 * 60 * 1000
      },
      standardHeaders: true,
      legacyHeaders: false,
      keyGenerator: (req) => {
        return req.user?.id || req.ip; // User-based if authenticated
      }
    }),

    // General API access
    api: rateLimit({
      windowMs: 15 * 60 * 1000,
      max: 1000, // Liberal for learning activities
      message: { error: 'Too many API requests' }
    }),

    // Progress updates - more liberal for active learning
    progress: rateLimit({
      windowMs: 1 * 60 * 1000, // 1 minute
      max: 100, // Allow frequent progress updates
      message: { error: 'Too many progress updates' }
    }),

    // Authentication attempts
    auth: rateLimit({
      windowMs: 15 * 60 * 1000,
      max: 10, // Conservative for security
      message: { error: 'Too many authentication attempts' }
    })
  };
};

// CORS configuration for educational platform
export const corsConfig = cors({
  origin: (origin, callback) => {
    const allowedOrigins = [
      process.env.FRONTEND_URL,
      'http://localhost:3000', // Development
      'http://localhost:5173', // Vite development
      ...process.env.ADDITIONAL_ORIGINS?.split(',') || []
    ];

    // Allow requests with no origin (mobile apps, Postman, etc.)
    if (!origin) return callback(null, true);

    if (allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS policy'));
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS', 'PATCH'],
  allowedHeaders: [
    'Content-Type',
    'Authorization',
    'X-Requested-With',
    'Accept',
    'Origin',
    'X-API-Key'
  ],
  exposedHeaders: ['X-Total-Count', 'X-Rate-Limit-Remaining']
});

// Input sanitization for educational content
export const sanitizeEducationalContent = (content) => {
  // Allow educational HTML tags but sanitize dangerous content
  const allowedTags = [
    'p', 'br', 'strong', 'em', 'u', 'h1', 'h2', 'h3', 'h4', 'h5', 'h6',
    'ul', 'ol', 'li', 'blockquote', 'code', 'pre', 'table', 'thead', 'tbody',
    'tr', 'td', 'th', 'a', 'img', 'video', 'audio', 'iframe'
  ];

  const allowedAttributes = {
    'a': ['href', 'title', 'target'],
    'img': ['src', 'alt', 'width', 'height', 'title'],
    'video': ['src', 'controls', 'width', 'height'],
    'audio': ['src', 'controls'],
    'iframe': ['src', 'width', 'height', 'frameborder', 'allowfullscreen']
  };

  return DOMPurify.sanitize(content, {
    ALLOWED_TAGS: allowedTags,
    ALLOWED_ATTR: Object.values(allowedAttributes).flat(),
    ALLOW_DATA_ATTR: false,
    FORBID_SCRIPT: true,
    FORBID_TAGS: ['script', 'style', 'object', 'embed', 'form', 'input']
  });
};

// Educational content validation middleware
export const validateEducationalContent = [
  body('title')
    .isLength({ min: 3, max: 200 })
    .trim()
    .escape()
    .withMessage('Title must be 3-200 characters'),
  
  body('description')
    .isLength({ min: 10, max: 2000 })
    .trim()
    .custom((value) => {
      // Sanitize educational description
      return sanitizeEducationalContent(value);
    })
    .withMessage('Description must be 10-2000 characters'),
  
  body('content')
    .optional()
    .isLength({ max: 50000 })
    .custom((value) => {
      // Sanitize and validate educational content
      return sanitizeEducationalContent(value);
    }),
  
  body('difficulty')
    .isIn(['beginner', 'intermediate', 'advanced'])
    .withMessage('Invalid difficulty level'),
  
  body('tags')
    .optional()
    .isArray({ max: 10 })
    .custom((tags) => {
      return tags.every(tag => 
        typeof tag === 'string' && 
        tag.length <= 50 && 
        /^[a-zA-Z0-9\s-_]+$/.test(tag)
      );
    })
    .withMessage('Invalid tags format')
];

// Request validation error handler
export const handleValidationErrors = (req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({
      error: 'Validation failed',
      details: errors.array().map(err => ({
        field: err.param,
        message: err.msg,
        value: err.value
      }))
    });
  }
  next();
};
```

## Authentication and Authorization for Educational Platforms

### JWT-Based Authentication with Educational Roles
✅ **Good Example: Educational Role-Based Authentication**
```javascript
// middleware/auth.js - Educational platform authentication
import jwt from 'jsonwebtoken';
import bcrypt from 'bcryptjs';
import { prisma } from '../database/connection.js';

// Educational platform roles with specific permissions
const EDUCATIONAL_ROLES = {
  STUDENT: {
    name: 'student',
    permissions: [
      'course:view',
      'course:enroll',
      'lesson:access',
      'progress:update',
      'progress:view_own',
      'ai_teacher:interact'
    ]
  },
  INSTRUCTOR: {
    name: 'instructor',
    permissions: [
      'course:view',
      'course:create',
      'course:edit_own',
      'course:delete_own',
      'lesson:create',
      'lesson:edit',
      'student:view_progress',
      'analytics:view_course',
      'ai_teacher:configure'
    ]
  },
  ADMIN: {
    name: 'admin',
    permissions: [
      'course:*',
      'user:*',
      'system:*',
      'analytics:*',
      'security:audit'
    ]
  },
  CONTENT_MODERATOR: {
    name: 'content_moderator',
    permissions: [
      'course:view',
      'course:moderate',
      'content:review',
      'content:approve',
      'content:flag',
      'ai_content:review'
    ]
  }
};

// Generate secure JWT tokens for educational platform
export const generateAuthTokens = async (user) => {
  const payload = {
    userId: user.id,
    email: user.email,
    role: user.role,
    permissions: EDUCATIONAL_ROLES[user.role.toUpperCase()]?.permissions || [],
    lastPasswordChange: user.passwordChangedAt?.getTime()
  };

  const accessToken = jwt.sign(payload, process.env.JWT_ACCESS_SECRET, {
    expiresIn: '1h', // Short-lived for security
    issuer: 'lms-platform',
    audience: 'lms-users'
  });

  const refreshToken = jwt.sign(
    { userId: user.id, tokenType: 'refresh' },
    process.env.JWT_REFRESH_SECRET,
    {
      expiresIn: '7d', // Longer-lived for convenience
      issuer: 'lms-platform',
      audience: 'lms-users'
    }
  );

  // Store refresh token in database with expiration
  await prisma.refreshToken.create({
    data: {
      token: refreshToken,
      userId: user.id,
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000), // 7 days
      userAgent: 'web-app', // Could be extracted from request
      ipAddress: 'localhost' // Could be extracted from request
    }
  });

  return { accessToken, refreshToken };
};

// Verify and decode JWT tokens
export const verifyAuthToken = async (req, res, next) => {
  try {
    const authHeader = req.headers.authorization;
    
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      return res.status(401).json({ 
        error: 'Authentication required',
        code: 'MISSING_TOKEN'
      });
    }

    const token = authHeader.substring(7); // Remove 'Bearer ' prefix
    
    const decoded = jwt.verify(token, process.env.JWT_ACCESS_SECRET, {
      issuer: 'lms-platform',
      audience: 'lms-users'
    });

    // Check if user still exists and is active
    const user = await prisma.user.findUnique({
      where: { id: decoded.userId },
      select: {
        id: true,
        email: true,
        role: true,
        isActive: true,
        passwordChangedAt: true,
        lastLoginAt: true
      }
    });

    if (!user || !user.isActive) {
      return res.status(401).json({ 
        error: 'User account not found or deactivated',
        code: 'INVALID_USER'
      });
    }

    // Check if password was changed after token was issued
    if (user.passwordChangedAt && 
        user.passwordChangedAt.getTime() > decoded.lastPasswordChange) {
      return res.status(401).json({ 
        error: 'Token invalid due to password change',
        code: 'TOKEN_INVALIDATED'
      });
    }

    // Add user and permissions to request
    req.user = user;
    req.permissions = EDUCATIONAL_ROLES[user.role.toUpperCase()]?.permissions || [];
    
    next();
  } catch (error) {
    if (error.name === 'TokenExpiredError') {
      return res.status(401).json({ 
        error: 'Token expired',
        code: 'TOKEN_EXPIRED'
      });
    }
    
    if (error.name === 'JsonWebTokenError') {
      return res.status(401).json({ 
        error: 'Invalid token',
        code: 'INVALID_TOKEN'
      });
    }

    console.error('Auth verification error:', error);
    return res.status(500).json({ 
      error: 'Authentication verification failed',
      code: 'AUTH_ERROR'
    });
  }
};

// Permission-based authorization middleware
export const requirePermission = (requiredPermission) => {
  return (req, res, next) => {
    if (!req.user || !req.permissions) {
      return res.status(401).json({ 
        error: 'Authentication required',
        code: 'NOT_AUTHENTICATED'
      });
    }

    // Check for wildcard permissions (admin access)
    const hasWildcard = req.permissions.some(permission => 
      permission.endsWith(':*') && 
      requiredPermission.startsWith(permission.replace(':*', ':'))
    );

    if (hasWildcard || req.permissions.includes(requiredPermission)) {
      return next();
    }

    return res.status(403).json({ 
      error: 'Insufficient permissions',
      code: 'FORBIDDEN',
      required: requiredPermission,
      user_permissions: req.permissions
    });
  };
};

// Resource ownership verification for educational content
export const requireResourceOwnership = (resourceType) => {
  return async (req, res, next) => {
    try {
      const resourceId = req.params.courseId || req.params.lessonId || req.params.id;
      
      if (!resourceId) {
        return res.status(400).json({ 
          error: 'Resource ID required',
          code: 'MISSING_RESOURCE_ID'
        });
      }

      let resource;
      switch (resourceType) {
        case 'course':
          resource = await prisma.course.findUnique({
            where: { id: resourceId },
            select: { instructorId: true, id: true }
          });
          break;
        case 'lesson':
          resource = await prisma.lesson.findUnique({
            where: { id: resourceId },
            include: { course: { select: { instructorId: true } } }
          });
          break;
        default:
          return res.status(400).json({ 
            error: 'Invalid resource type',
            code: 'INVALID_RESOURCE_TYPE'
          });
      }

      if (!resource) {
        return res.status(404).json({ 
          error: `${resourceType} not found`,
          code: 'RESOURCE_NOT_FOUND'
        });
      }

      // Check ownership
      const ownerId = resource.instructorId || resource.course?.instructorId;
      const isOwner = ownerId === req.user.id;
      const isAdmin = req.user.role === 'admin';

      if (!isOwner && !isAdmin) {
        return res.status(403).json({ 
          error: 'Access denied - not resource owner',
          code: 'NOT_OWNER'
        });
      }

      req.resource = resource;
      next();
    } catch (error) {
      console.error('Ownership verification error:', error);
      return res.status(500).json({ 
        error: 'Ownership verification failed',
        code: 'OWNERSHIP_CHECK_ERROR'
      });
    }
  };
};

// Secure password hashing for educational accounts
export const hashPassword = async (password) => {
  // Validate password strength for educational accounts
  if (password.length < 8) {
    throw new Error('Password must be at least 8 characters long');
  }
  
  if (!/(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(password)) {
    throw new Error('Password must contain uppercase, lowercase, and numeric characters');
  }

  const saltRounds = 12; // High security for educational data
  return await bcrypt.hash(password, saltRounds);
};

// Verify password with timing attack protection
export const verifyPassword = async (password, hashedPassword) => {
  return await bcrypt.compare(password, hashedPassword);
};
```

## AI Content Security and Validation

### Secure AI Integration for Educational Content
✅ **Good Example: AI Content Security Framework**
```javascript
// services/aiContentSecurity.js - AI content security and validation
import { z } from 'zod';
import DOMPurify from 'isomorphic-dompurify';
import { rateLimit } from 'express-rate-limit';

// AI content validation schema
const AIContentSchema = z.object({
  title: z.string().min(1).max(200),
  content: z.string().min(10).max(50000),
  type: z.enum(['text', 'quiz', 'exercise', 'video_script']),
  metadata: z.object({
    difficulty: z.enum(['beginner', 'intermediate', 'advanced']),
    subject: z.string().max(100),
    estimatedMinutes: z.number().min(1).max(180)
  }).optional()
});

// Prompt injection detection patterns
const PROMPT_INJECTION_PATTERNS = [
  // Direct instruction attempts
  /ignore\s+previous\s+instructions?/i,
  /forget\s+everything\s+above/i,
  /new\s+instructions?:/i,
  /system\s*:\s*/i,
  /developer\s+mode/i,
  
  // Role manipulation attempts
  /you\s+are\s+now\s+a/i,
  /pretend\s+to\s+be/i,
  /act\s+as\s+if/i,
  /roleplay\s+as/i,
  
  // Output manipulation
  /respond\s+with\s+only/i,
  /output\s+format:/i,
  /answer\s+in\s+the\s+style\s+of/i,
  
  // Harmful content attempts
  /generate\s+code\s+for/i,
  /create\s+malware/i,
  /hack\s+into/i,
  /social\s+security\s+number/i,
  /credit\s+card\s+number/i
];

// Educational content appropriateness filters
const INAPPROPRIATE_CONTENT_PATTERNS = [
  // Violence and harmful content
  /\b(kill|murder|suicide|self-harm|violence)\b/i,
  
  // Adult content
  /\b(sex|sexual|porn|nude|naked)\b/i,
  
  // Discrimination and hate speech
  /\b(racist|sexist|homophobic|transphobic)\b/i,
  
  // Illegal activities
  /\b(drugs|illegal|stealing|fraud|scam)\b/i,
  
  // Personal information
  /\b\d{3}-\d{2}-\d{4}\b/, // SSN pattern
  /\b\d{4}\s?\d{4}\s?\d{4}\s?\d{4}\b/, // Credit card pattern
  /\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/ // Email pattern
];

export class AIContentSecurityService {
  constructor() {
    this.rateLimiter = rateLimit({
      windowMs: 15 * 60 * 1000, // 15 minutes
      max: 50, // Limit each user to 50 AI requests per window
      keyGenerator: (req) => req.user?.id || req.ip,
      message: { error: 'Too many AI content requests' }
    });
  }

  // Validate and sanitize AI prompts before sending to AI service
  validateAIPrompt(prompt, context = {}) {
    // Check for prompt injection attempts
    for (const pattern of PROMPT_INJECTION_PATTERNS) {
      if (pattern.test(prompt)) {
        throw new Error(`Potential prompt injection detected: ${pattern.source}`);
      }
    }

    // Check for inappropriate content requests
    for (const pattern of INAPPROPRIATE_CONTENT_PATTERNS) {
      if (pattern.test(prompt)) {
        throw new Error(`Inappropriate content request detected: ${pattern.source}`);
      }
    }

    // Sanitize prompt while preserving educational intent
    const sanitizedPrompt = this.sanitizePrompt(prompt);
    
    // Add educational context and safety instructions
    return this.addEducationalContext(sanitizedPrompt, context);
  }

  sanitizePrompt(prompt) {
    // Remove potentially dangerous HTML/script content
    let sanitized = DOMPurify.sanitize(prompt, { 
      ALLOWED_TAGS: [], 
      ALLOWED_ATTR: [] 
    });

    // Remove excessive whitespace and normalize
    sanitized = sanitized.replace(/\s+/g, ' ').trim();

    // Remove suspicious character sequences
    sanitized = sanitized.replace(/[^\w\s.,!?;:()\-'"]/g, '');

    return sanitized;
  }

  addEducationalContext(prompt, context) {
    const educationalPrefix = `
    You are an educational content creator for a learning management system.
    Create appropriate, educational content suitable for ${context.difficulty || 'general'} level learners.
    Focus on accuracy, clarity, and educational value.
    Do not include any inappropriate, harmful, or non-educational content.
    
    User Request: ${prompt}
    
    Educational Content Guidelines:
    - Keep content appropriate for educational environments
    - Use clear, understandable language
    - Include practical examples where relevant
    - Ensure factual accuracy
    - Structure content for learning objectives
    `;

    return educationalPrefix;
  }

  // Validate AI-generated content before saving
  async validateAIGeneratedContent(content, contentType) {
    try {
      // Schema validation
      const validatedContent = AIContentSchema.parse({
        title: content.title,
        content: content.content,
        type: contentType,
        metadata: content.metadata
      });

      // Content appropriateness check
      this.checkContentAppropriateness(validatedContent.content);

      // Educational quality validation
      await this.validateEducationalQuality(validatedContent);

      // Sanitize HTML content
      const sanitizedContent = {
        ...validatedContent,
        content: this.sanitizeEducationalHTML(validatedContent.content)
      };

      return sanitizedContent;
    } catch (error) {
      throw new Error(`AI content validation failed: ${error.message}`);
    }
  }

  checkContentAppropriateness(content) {
    for (const pattern of INAPPROPRIATE_CONTENT_PATTERNS) {
      if (pattern.test(content)) {
        throw new Error(`Inappropriate content detected in AI response: ${pattern.source}`);
      }
    }

    // Additional checks for educational appropriateness
    if (content.length < 50) {
      throw new Error('AI-generated content too short for educational value');
    }

    if (content.length > 50000) {
      throw new Error('AI-generated content too long for practical use');
    }
  }

  async validateEducationalQuality(content) {
    // Check for educational structure
    const hasEducationalStructure = this.checkEducationalStructure(content.content);
    if (!hasEducationalStructure) {
      console.warn('AI content may lack proper educational structure');
    }

    // Verify content matches declared difficulty
    const difficultyMatch = this.verifyDifficultyLevel(
      content.content, 
      content.metadata?.difficulty
    );
    if (!difficultyMatch) {
      console.warn('AI content difficulty may not match declared level');
    }

    return true;
  }

  checkEducationalStructure(content) {
    // Look for educational markers
    const educationalPatterns = [
      /\b(learn|understand|concept|example|practice|exercise)\b/i,
      /\b(introduction|explanation|definition|summary)\b/i,
      /\b(step\s*\d+|first|second|next|finally)\b/i,
      /\?\s*$/m // Questions
    ];

    return educationalPatterns.some(pattern => pattern.test(content));
  }

  verifyDifficultyLevel(content, declaredDifficulty) {
    const complexityMarkers = {
      beginner: /\b(basic|simple|introduction|easy|start)\b/i,
      intermediate: /\b(moderate|standard|regular|typical)\b/i,
      advanced: /\b(complex|advanced|sophisticated|detailed)\b/i
    };

    if (!declaredDifficulty) return true;

    const pattern = complexityMarkers[declaredDifficulty];
    return pattern ? pattern.test(content) : true;
  }

  sanitizeEducationalHTML(content) {
    // Allow educational HTML tags while removing dangerous content
    return DOMPurify.sanitize(content, {
      ALLOWED_TAGS: [
        'p', 'br', 'strong', 'em', 'u', 'h1', 'h2', 'h3', 'h4', 'h5', 'h6',
        'ul', 'ol', 'li', 'blockquote', 'code', 'pre', 'table', 'thead', 'tbody',
        'tr', 'td', 'th', 'a', 'img'
      ],
      ALLOWED_ATTR: {
        'a': ['href', 'title'],
        'img': ['src', 'alt', 'width', 'height'],
        'table': ['border', 'cellpadding', 'cellspacing'],
        'td': ['colspan', 'rowspan'],
        'th': ['colspan', 'rowspan']
      },
      FORBID_SCRIPT: true,
      FORBID_TAGS: ['script', 'style', 'object', 'embed', 'iframe'],
      FORBID_ATTR: ['onclick', 'onload', 'onerror', 'onmouseover']
    });
  }

  // Monitor AI service usage and detect anomalies
  async monitorAIUsage(userId, requestType, prompt, response) {
    try {
      await prisma.aiUsageLog.create({
        data: {
          userId,
          requestType,
          promptHash: this.hashSensitiveData(prompt),
          responseLength: response.length,
          timestamp: new Date(),
          ipAddress: 'request-ip', // Extract from request
          userAgent: 'request-user-agent' // Extract from request
        }
      });

      // Check for suspicious usage patterns
      await this.detectSuspiciousUsage(userId);
    } catch (error) {
      console.error('AI usage monitoring error:', error);
    }
  }

  async detectSuspiciousUsage(userId) {
    const recentUsage = await prisma.aiUsageLog.findMany({
      where: {
        userId,
        timestamp: {
          gte: new Date(Date.now() - 60 * 60 * 1000) // Last hour
        }
      },
      orderBy: { timestamp: 'desc' }
    });

    // Flag excessive usage
    if (recentUsage.length > 100) {
      await this.flagSuspiciousActivity(userId, 'excessive_ai_usage', {
        count: recentUsage.length,
        timeframe: '1_hour'
      });
    }

    // Check for repeated identical requests (potential automation)
    const uniqueHashes = new Set(recentUsage.map(log => log.promptHash));
    if (uniqueHashes.size < recentUsage.length * 0.1) {
      await this.flagSuspiciousActivity(userId, 'repeated_requests', {
        totalRequests: recentUsage.length,
        uniqueRequests: uniqueHashes.size
      });
    }
  }

  async flagSuspiciousActivity(userId, activityType, details) {
    await prisma.securityAlert.create({
      data: {
        userId,
        alertType: 'ai_usage_anomaly',
        severity: 'medium',
        details: {
          activityType,
          ...details,
          timestamp: new Date()
        },
        status: 'pending_review'
      }
    });

    console.warn(`Suspicious AI usage detected for user ${userId}: ${activityType}`);
  }

  hashSensitiveData(data) {
    const crypto = require('crypto');
    return crypto.createHash('sha256').update(data).digest('hex');
  }
}

export const aiContentSecurity = new AIContentSecurityService();
```

## Data Protection and Privacy for Educational Platforms

### FERPA and GDPR Compliance Framework
✅ **Good Example: Educational Data Privacy Implementation**
```javascript
// services/dataPrivacy.js - Educational data privacy and compliance
import crypto from 'crypto';
import { prisma } from '../database/connection.js';

export class EducationalDataPrivacyService {
  constructor() {
    this.encryptionKey = process.env.DATA_ENCRYPTION_KEY;
    this.algorithm = 'aes-256-gcm';
  }

  // Encrypt sensitive educational data (FERPA compliance)
  encryptSensitiveData(data) {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipher(this.algorithm, this.encryptionKey);
    cipher.setAAD(Buffer.from('educational-data', 'utf8'));
    
    let encrypted = cipher.update(JSON.stringify(data), 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    const authTag = cipher.getAuthTag();
    
    return {
      encryptedData: encrypted,
      iv: iv.toString('hex'),
      authTag: authTag.toString('hex')
    };
  }

  // Decrypt sensitive educational data
  decryptSensitiveData(encryptedData, iv, authTag) {
    const decipher = crypto.createDecipher(this.algorithm, this.encryptionKey);
    decipher.setAAD(Buffer.from('educational-data', 'utf8'));
    decipher.setAuthTag(Buffer.from(authTag, 'hex'));
    
    let decrypted = decipher.update(encryptedData, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return JSON.parse(decrypted);
  }

  // FERPA-compliant data access logging
  async logDataAccess(accessorId, studentId, dataType, purpose, ipAddress) {
    await prisma.educationalDataAccess.create({
      data: {
        accessorId,
        studentId,
        dataType,
        purpose,
        accessTime: new Date(),
        ipAddress,
        compliance: 'FERPA',
        legitimate_interest: purpose
      }
    });
  }

  // Data retention policy enforcement
  async enforceDataRetention() {
    const retentionPolicies = {
      // Student progress data - keep for 7 years after graduation
      progress_data: { years: 7, after: 'graduation' },
      
      // AI interaction logs - keep for 1 year
      ai_interactions: { years: 1, after: 'creation' },
      
      // Course content - keep for 5 years after course end
      course_content: { years: 5, after: 'course_end' },
      
      // Security logs - keep for 2 years
      security_logs: { years: 2, after: 'creation' }
    };

    for (const [dataType, policy] of Object.entries(retentionPolicies)) {
      await this.purgeExpiredData(dataType, policy);
    }
  }

  async purgeExpiredData(dataType, policy) {
    const cutoffDate = new Date();
    cutoffDate.setFullYear(cutoffDate.getFullYear() - policy.years);

    let whereClause;
    switch (policy.after) {
      case 'graduation':
        whereClause = {
          user: {
            graduationDate: { lt: cutoffDate }
          }
        };
        break;
      case 'creation':
        whereClause = {
          createdAt: { lt: cutoffDate }
        };
        break;
      case 'course_end':
        whereClause = {
          course: {
            endDate: { lt: cutoffDate }
          }
        };
        break;
    }

    if (whereClause) {
      const result = await this.deleteDataByType(dataType, whereClause);
      console.log(`Data retention: Purged ${result.count} ${dataType} records older than ${policy.years} years`);
    }
  }

  async deleteDataByType(dataType, whereClause) {
    switch (dataType) {
      case 'progress_data':
        return await prisma.userProgress.deleteMany({ where: whereClause });
      case 'ai_interactions':
        return await prisma.aiUsageLog.deleteMany({ where: whereClause });
      case 'security_logs':
        return await prisma.securityLog.deleteMany({ where: whereClause });
      default:
        return { count: 0 };
    }
  }

  // GDPR Right to be Forgotten implementation
  async handleDataDeletionRequest(userId, requestType = 'full_deletion') {
    const deletionLog = await prisma.dataDeletionRequest.create({
      data: {
        userId,
        requestType,
        status: 'processing',
        requestedAt: new Date(),
        compliance: 'GDPR'
      }
    });

    try {
      switch (requestType) {
        case 'full_deletion':
          await this.performFullDataDeletion(userId);
          break;
        case 'anonymization':
          await this.performDataAnonymization(userId);
          break;
        case 'progress_only':
          await this.deleteProgressData(userId);
          break;
      }

      await prisma.dataDeletionRequest.update({
        where: { id: deletionLog.id },
        data: {
          status: 'completed',
          completedAt: new Date()
        }
      });

      return { success: true, deletionId: deletionLog.id };
    } catch (error) {
      await prisma.dataDeletionRequest.update({
        where: { id: deletionLog.id },
        data: {
          status: 'failed',
          errorMessage: error.message
        }
      });

      throw error;
    }
  }

  async performFullDataDeletion(userId) {
    // Delete user data in correct order (respecting foreign key constraints)
    const deletionOrder = [
      'aiUsageLog',
      'securityAlert',
      'educationalDataAccess',
      'refreshToken',
      'sectionProgress',
      'lessonProgress',
      'userProgress',
      'enrollment',
      'user'
    ];

    for (const table of deletionOrder) {
      await prisma[table].deleteMany({
        where: { userId }
      });
    }

    // Anonymize references in audit logs
    await prisma.auditLog.updateMany({
      where: { userId },
      data: { userId: null, anonymized: true }
    });
  }

  async performDataAnonymization(userId) {
    // Replace identifiable information with anonymized versions
    const anonymizedData = {
      email: `anonymous_${crypto.randomBytes(8).toString('hex')}@deleted.local`,
      name: 'Deleted User',
      firstName: 'Deleted',
      lastName: 'User',
      dateOfBirth: null,
      phone: null,
      address: null,
      emergencyContact: null,
      anonymized: true,
      originalUserId: userId
    };

    await prisma.user.update({
      where: { id: userId },
      data: anonymizedData
    });

    // Keep educational progress but remove identifying links
    await prisma.userProgress.updateMany({
      where: { userId },
      data: { anonymized: true }
    });
  }

  // Data export for GDPR compliance (Right to Data Portability)
  async exportUserData(userId, format = 'json') {
    const userData = await prisma.user.findUnique({
      where: { id: userId },
      include: {
        enrollments: {
          include: {
            course: {
              select: { id: true, title: true, description: true }
            }
          }
        },
        userProgress: {
          include: {
            course: {
              select: { id: true, title: true }
            },
            lessonProgress: {
              include: {
                lesson: {
                  select: { id: true, title: true }
                },
                sectionProgress: true
              }
            }
          }
        },
        aiUsageLog: {
          where: {
            timestamp: {
              gte: new Date(Date.now() - 365 * 24 * 60 * 60 * 1000) // Last year only
            }
          }
        }
      }
    });

    if (!userData) {
      throw new Error('User not found');
    }

    // Remove sensitive fields
    const exportData = {
      personalInfo: {
        email: userData.email,
        name: userData.name,
        role: userData.role,
        createdAt: userData.createdAt,
        lastLoginAt: userData.lastLoginAt
      },
      enrollments: userData.enrollments.map(enrollment => ({
        courseId: enrollment.course.id,
        courseTitle: enrollment.course.title,
        enrolledAt: enrollment.enrolledAt,
        completedAt: enrollment.completedAt
      })),
      learningProgress: userData.userProgress.map(progress => ({
        courseTitle: progress.course.title,
        totalTimeSpent: progress.totalTimeSpent,
        completed: progress.completed,
        completedAt: progress.completedAt,
        lessonProgress: progress.lessonProgress.map(lp => ({
          lessonTitle: lp.lesson.title,
          completed: lp.completed,
          timeSpent: lp.timeSpent,
          sectionsCompleted: lp.sectionProgress.filter(sp => sp.completed).length,
          totalSections: lp.sectionProgress.length
        }))
      })),
      aiInteractions: {
        totalRequests: userData.aiUsageLog.length,
        requestsByType: this.aggregateAIUsage(userData.aiUsageLog)
      }
    };

    // Log the data export for audit purposes
    await this.logDataAccess(
      'system',
      userId,
      'full_export',
      'GDPR_data_portability',
      'internal'
    );

    return format === 'json' ? exportData : this.convertToCSV(exportData);
  }

  aggregateAIUsage(aiLogs) {
    return aiLogs.reduce((acc, log) => {
      acc[log.requestType] = (acc[log.requestType] || 0) + 1;
      return acc;
    }, {});
  }

  convertToCSV(data) {
    // Convert JSON data to CSV format for data portability
    // Implementation would depend on specific requirements
    return 'CSV conversion not implemented in this example';
  }

  // Consent management for educational platforms
  async updateConsentPreferences(userId, consents) {
    const validConsentTypes = [
      'educational_analytics',
      'marketing_communications',
      'third_party_integrations',
      'ai_content_improvement',
      'performance_tracking'
    ];

    const consentData = {};
    for (const [consentType, granted] of Object.entries(consents)) {
      if (validConsentTypes.includes(consentType)) {
        consentData[consentType] = {
          granted: Boolean(granted),
          timestamp: new Date(),
          version: '1.0' // Track consent policy version
        };
      }
    }

    await prisma.userConsent.upsert({
      where: { userId },
      create: {
        userId,
        consents: consentData,
        updatedAt: new Date()
      },
      update: {
        consents: consentData,
        updatedAt: new Date()
      }
    });

    return consentData;
  }
}

export const dataPrivacy = new EducationalDataPrivacyService();
```

## Key Educational Platform Security Takeaways

### Critical Security Patterns for LMS Development ✅

1. **Multi-Layer Security Architecture**
   - Implement defense-in-depth with security at every layer
   - Use Content Security Policy to prevent XSS attacks
   - Apply rate limiting specific to educational workflows
   - Implement proper CORS policies for educational environments

2. **Educational Data Protection**
   - Encrypt sensitive student data (FERPA compliance)
   - Implement proper access controls for educational records
   - Log all access to student data for audit purposes
   - Enforce data retention policies appropriate for educational institutions

3. **AI Content Security**
   - Validate and sanitize all AI prompts for prompt injection attacks
   - Filter AI-generated content for appropriateness and educational value
   - Monitor AI usage patterns for anomalies and abuse
   - Implement content validation schemas for educational standards

4. **Authentication and Authorization**
   - Use JWT tokens with educational role-based permissions
   - Implement resource ownership checks for courses and content
   - Provide secure password requirements for educational accounts
   - Track authentication events for security monitoring

5. **Privacy Compliance**
   - Implement GDPR Right to be Forgotten for user data deletion
   - Provide data export capabilities for data portability
   - Maintain consent management for educational analytics
   - Ensure FERPA compliance for student educational records

### Security Anti-Patterns to Avoid in Educational Platforms ❌

1. **Insufficient Input Validation** - Not properly validating and sanitizing user inputs, especially AI prompts
2. **Weak Access Controls** - Not implementing proper role-based access for educational data
3. **Inadequate Audit Logging** - Missing comprehensive audit trails for educational data access
4. **Poor Privacy Controls** - Not implementing proper data deletion and export capabilities
5. **Unsafe AI Integration** - Not validating AI-generated content for appropriateness and safety

Educational platform security requires specialized attention to student data privacy, content appropriateness, and regulatory compliance while maintaining usability for effective learning experiences. 