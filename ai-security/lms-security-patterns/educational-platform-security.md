# LMS Educational Platform Security Patterns

## Input Validation and Sanitization for Educational Content

### Course Creation Security
**Critical for preventing injection attacks through AI-generated content**

✅ **Good Example: Comprehensive Input Validation**
```typescript
// security/inputValidation.ts
import DOMPurify from 'isomorphic-dompurify';
import validator from 'validator';

export class LMSInputValidator {
  private static readonly MAX_COURSE_TITLE_LENGTH = 200;
  private static readonly MAX_DESCRIPTION_LENGTH = 2000;
  private static readonly MAX_CONTENT_LENGTH = 50000;
  
  private static readonly ALLOWED_DIFFICULTIES = ['BEGINNER', 'INTERMEDIATE', 'EXPERT'];
  private static readonly ALLOWED_SECTION_TYPES = ['TEXT', 'EXERCISE', 'QUIZ', 'VIDEO_SCRIPT'];

  static validateCourseCreation(input: CourseCreationInput): ValidationResult {
    const errors: string[] = [];

    // Title validation
    if (!input.title || typeof input.title !== 'string') {
      errors.push('Course title is required and must be a string');
    } else {
      // Sanitize and validate title
      const sanitizedTitle = this.sanitizeText(input.title);
      if (sanitizedTitle.length === 0) {
        errors.push('Course title cannot be empty after sanitization');
      }
      if (sanitizedTitle.length > this.MAX_COURSE_TITLE_LENGTH) {
        errors.push(`Course title exceeds maximum length of ${this.MAX_COURSE_TITLE_LENGTH} characters`);
      }
      // Check for potential XSS patterns
      if (this.containsMaliciousPatterns(sanitizedTitle)) {
        errors.push('Course title contains potentially malicious content');
      }
    }

    // Description validation
    if (input.description && typeof input.description === 'string') {
      const sanitizedDescription = this.sanitizeText(input.description);
      if (sanitizedDescription.length > this.MAX_DESCRIPTION_LENGTH) {
        errors.push(`Course description exceeds maximum length of ${this.MAX_DESCRIPTION_LENGTH} characters`);
      }
      if (this.containsMaliciousPatterns(sanitizedDescription)) {
        errors.push('Course description contains potentially malicious content');
      }
    }

    // Difficulty validation
    if (!input.difficulty || !this.ALLOWED_DIFFICULTIES.includes(input.difficulty)) {
      errors.push(`Difficulty must be one of: ${this.ALLOWED_DIFFICULTIES.join(', ')}`);
    }

    // SQL injection prevention - check for common SQL injection patterns
    const sqlInjectionPatterns = [
      /(\bUNION\b|\bSELECT\b|\bINSERT\b|\bUPDATE\b|\bDELETE\b|\bDROP\b)/i,
      /(--|\/\*|\*\/|;|\||&)/,
      /(\bOR\b.*=.*|1=1|' OR ')/i
    ];

    const allInputText = [input.title, input.description].filter(Boolean).join(' ');
    for (const pattern of sqlInjectionPatterns) {
      if (pattern.test(allInputText)) {
        errors.push('Input contains potentially malicious SQL patterns');
        break;
      }
    }

    return {
      isValid: errors.length === 0,
      errors,
      sanitizedInput: {
        title: this.sanitizeText(input.title || ''),
        description: input.description ? this.sanitizeText(input.description) : undefined,
        difficulty: input.difficulty
      }
    };
  }

  static validateGeneratedContent(content: GeneratedContent): ValidationResult {
    const errors: string[] = [];

    // Validate course structure
    if (!content.lessons || !Array.isArray(content.lessons)) {
      errors.push('Generated content must include lessons array');
    } else {
      content.lessons.forEach((lesson, index) => {
        const lessonErrors = this.validateLesson(lesson, index);
        errors.push(...lessonErrors);
      });
    }

    // Content safety validation
    const contentText = this.extractAllText(content);
    if (this.containsInappropriateContent(contentText)) {
      errors.push('Generated content contains inappropriate material');
    }

    // Check for potential prompt injection attempts
    if (this.containsPromptInjection(contentText)) {
      errors.push('Generated content contains potential prompt injection attempts');
    }

    return {
      isValid: errors.length === 0,
      errors,
      sanitizedInput: this.sanitizeGeneratedContent(content)
    };
  }

  private static validateLesson(lesson: any, index: number): string[] {
    const errors: string[] = [];

    if (!lesson.title || typeof lesson.title !== 'string') {
      errors.push(`Lesson ${index + 1}: Title is required`);
    }

    if (!lesson.sections || !Array.isArray(lesson.sections)) {
      errors.push(`Lesson ${index + 1}: Sections array is required`);
    } else {
      lesson.sections.forEach((section: any, sIndex: number) => {
        // Validate section type
        if (!section.type || !this.ALLOWED_SECTION_TYPES.includes(section.type)) {
          errors.push(`Lesson ${index + 1}, Section ${sIndex + 1}: Invalid section type`);
        }

        // Validate content length
        if (section.content && section.content.length > this.MAX_CONTENT_LENGTH) {
          errors.push(`Lesson ${index + 1}, Section ${sIndex + 1}: Content exceeds maximum length`);
        }

        // Validate content security
        if (section.content && this.containsMaliciousPatterns(section.content)) {
          errors.push(`Lesson ${index + 1}, Section ${sIndex + 1}: Content contains malicious patterns`);
        }
      });
    }

    return errors;
  }

  private static sanitizeText(text: string): string {
    if (!text) return '';
    
    // Remove potential XSS vectors
    let sanitized = DOMPurify.sanitize(text, {
      ALLOWED_TAGS: [], // No HTML tags allowed in titles/descriptions
      ALLOWED_ATTR: [],
      FORBID_TAGS: ['script', 'object', 'embed', 'iframe'],
      FORBID_ATTR: ['onerror', 'onload', 'onclick']
    });

    // Additional sanitization
    sanitized = validator.escape(sanitized);
    
    // Remove potential command injection patterns
    sanitized = sanitized.replace(/[`${}]/g, '');
    
    return sanitized.trim();
  }

  private static containsMaliciousPatterns(text: string): boolean {
    const maliciousPatterns = [
      /<script[^>]*>.*?<\/script>/gi,
      /javascript:/gi,
      /on\w+\s*=/gi,
      /data:text\/html/gi,
      /vbscript:/gi,
      /<iframe[^>]*>/gi,
      /<object[^>]*>/gi,
      /<embed[^>]*>/gi
    ];

    return maliciousPatterns.some(pattern => pattern.test(text));
  }

  private static containsInappropriateContent(text: string): boolean {
    // Implement content moderation logic
    const inappropriatePatterns = [
      /\b(explicit sexual content|hate speech|violence|illegal activities)\b/gi,
      // Add more patterns based on educational content standards
    ];

    return inappropriatePatterns.some(pattern => pattern.test(text));
  }

  private static containsPromptInjection(text: string): boolean {
    const promptInjectionPatterns = [
      /ignore previous instructions/gi,
      /system prompt/gi,
      /you are now/gi,
      /forget everything/gi,
      /new instructions:/gi
    ];

    return promptInjectionPatterns.some(pattern => pattern.test(text));
  }

  private static extractAllText(content: GeneratedContent): string {
    let allText = content.title || '';
    allText += ' ' + (content.description || '');
    
    if (content.lessons) {
      content.lessons.forEach(lesson => {
        allText += ' ' + lesson.title;
        if (lesson.sections) {
          lesson.sections.forEach(section => {
            allText += ' ' + (section.content || '');
          });
        }
      });
    }
    
    return allText;
  }

  private static sanitizeGeneratedContent(content: GeneratedContent): GeneratedContent {
    return {
      ...content,
      title: this.sanitizeText(content.title),
      description: content.description ? this.sanitizeText(content.description) : undefined,
      lessons: content.lessons?.map(lesson => ({
        ...lesson,
        title: this.sanitizeText(lesson.title),
        sections: lesson.sections?.map(section => ({
          ...section,
          title: this.sanitizeText(section.title),
          content: this.sanitizeEducationalContent(section.content)
        }))
      }))
    };
  }

  private static sanitizeEducationalContent(content: string): string {
    if (!content) return '';

    // Allow specific educational HTML tags
    const sanitized = DOMPurify.sanitize(content, {
      ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'ul', 'ol', 'li', 'h1', 'h2', 'h3', 'h4', 'code', 'pre'],
      ALLOWED_ATTR: [],
      FORBID_TAGS: ['script', 'style', 'iframe', 'object', 'embed'],
      FORBID_ATTR: ['style', 'class', 'id', 'onclick']
    });

    return sanitized;
  }
}

// Type definitions
interface CourseCreationInput {
  title: string;
  description?: string;
  difficulty: string;
}

interface GeneratedContent {
  title: string;
  description?: string;
  lessons?: Array<{
    title: string;
    sections?: Array<{
      title: string;
      content: string;
      type: string;
    }>;
  }>;
}

interface ValidationResult {
  isValid: boolean;
  errors: string[];
  sanitizedInput?: any;
}
```

❌ **Bad Example: Insufficient Input Validation**
```typescript
// Poor security - no validation or sanitization
export class BadInputValidator {
  static validateCourseCreation(input: any) {
    return { isValid: !!input.title }; // No sanitization or security checks
  }
}
```

### API Security and Rate Limiting
✅ **Good Example: Comprehensive API Security**
```typescript
// security/apiSecurity.ts
import rateLimit from 'express-rate-limit';
import helmet from 'helmet';
import { Request, Response, NextFunction } from 'express';

export class LMSAPISecurity {
  
  // Rate limiting for different endpoints
  static getCourseGenerationLimiter() {
    return rateLimit({
      windowMs: 15 * 60 * 1000, // 15 minutes
      max: 5, // Limit to 5 course generations per 15 minutes
      message: {
        error: 'Too many course generation requests',
        retryAfter: '15 minutes'
      },
      standardHeaders: true,
      legacyHeaders: false,
      keyGenerator: (req) => {
        // Use IP + user agent for more precise limiting
        return req.ip + req.get('User-Agent');
      }
    });
  }

  static getAPILimiter() {
    return rateLimit({
      windowMs: 1 * 60 * 1000, // 1 minute
      max: 100, // Limit to 100 requests per minute
      message: {
        error: 'Too many API requests',
        retryAfter: '1 minute'
      }
    });
  }

  // Security headers middleware
  static getSecurityHeaders() {
    return helmet({
      contentSecurityPolicy: {
        directives: {
          defaultSrc: ["'self'"],
          styleSrc: ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"],
          fontSrc: ["'self'", "https://fonts.gstatic.com"],
          imgSrc: ["'self'", "data:", "https:", "blob:"],
          scriptSrc: ["'self'"],
          connectSrc: ["'self'", "https://api.anthropic.com", "https://api.openai.com"],
          objectSrc: ["'none'"],
          mediaSrc: ["'self'"],
          frameSrc: ["'none'"]
        }
      },
      crossOriginEmbedderPolicy: false, // Disable for AI API compatibility
      hsts: {
        maxAge: 31536000,
        includeSubDomains: true,
        preload: true
      }
    });
  }

  // Request validation middleware
  static validateRequest(req: Request, res: Response, next: NextFunction) {
    // Validate Content-Type for POST requests
    if (req.method === 'POST' && !req.is('application/json')) {
      return res.status(400).json({
        error: 'Content-Type must be application/json'
      });
    }

    // Validate request size
    if (req.get('content-length')) {
      const contentLength = parseInt(req.get('content-length')!);
      if (contentLength > 1024 * 1024) { // 1MB limit
        return res.status(413).json({
          error: 'Request too large'
        });
      }
    }

    // Validate required headers
    if (!req.get('User-Agent')) {
      return res.status(400).json({
        error: 'User-Agent header required'
      });
    }

    next();
  }

  // API key validation (for future authentication)
  static validateAPIKey(req: Request, res: Response, next: NextFunction) {
    const apiKey = req.get('X-API-Key') || req.query.apiKey;
    
    if (!apiKey) {
      return res.status(401).json({
        error: 'API key required'
      });
    }

    // Validate API key format
    if (typeof apiKey !== 'string' || apiKey.length < 32) {
      return res.status(401).json({
        error: 'Invalid API key format'
      });
    }

    // Additional API key validation logic would go here
    next();
  }

  // CORS configuration for LMS
  static getCORSConfig() {
    return {
      origin: process.env.NODE_ENV === 'production' 
        ? [process.env.FRONTEND_URL] 
        : true,
      credentials: true,
      methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
      allowedHeaders: ['Content-Type', 'Authorization', 'X-API-Key'],
      maxAge: 86400 // 24 hours
    };
  }

  // Input sanitization middleware
  static sanitizeInput(req: Request, res: Response, next: NextFunction) {
    if (req.body) {
      req.body = this.deepSanitizeObject(req.body);
    }
    
    if (req.query) {
      req.query = this.deepSanitizeObject(req.query);
    }
    
    next();
  }

  private static deepSanitizeObject(obj: any): any {
    if (typeof obj === 'string') {
      return validator.escape(obj.trim());
    }
    
    if (Array.isArray(obj)) {
      return obj.map(item => this.deepSanitizeObject(item));
    }
    
    if (obj && typeof obj === 'object') {
      const sanitized: any = {};
      for (const [key, value] of Object.entries(obj)) {
        const sanitizedKey = validator.escape(key);
        sanitized[sanitizedKey] = this.deepSanitizeObject(value);
      }
      return sanitized;
    }
    
    return obj;
  }
}
```

### Database Security Patterns
✅ **Good Example: Secure Database Operations**
```typescript
// security/databaseSecurity.ts
import { PrismaClient } from '@prisma/client';
import bcrypt from 'bcryptjs';

export class LMSDatabaseSecurity {
  private prisma: PrismaClient;

  constructor(prisma: PrismaClient) {
    this.prisma = prisma;
  }

  // Secure course creation with validation
  async createCourse(courseData: CourseCreationData, creatorId?: string): Promise<Course> {
    // Validate input data
    const validation = LMSInputValidator.validateCourseCreation(courseData);
    if (!validation.isValid) {
      throw new SecurityError('Invalid course data', validation.errors);
    }

    // Use parameterized queries (Prisma handles this automatically)
    const course = await this.prisma.course.create({
      data: {
        title: validation.sanitizedInput.title,
        description: validation.sanitizedInput.description,
        difficulty: validation.sanitizedInput.difficulty,
        isPublished: false, // Always start unpublished for review
        createdAt: new Date(),
        // creatorId would be added when authentication is implemented
      }
    });

    // Log security-relevant action
    await this.logSecurityEvent('COURSE_CREATED', {
      courseId: course.id,
      creatorId,
      timestamp: new Date()
    });

    return course;
  }

  // Secure progress update with validation
  async updateProgress(
    userId: string, 
    sectionId: string, 
    progressData: ProgressUpdateData
  ): Promise<Progress> {
    // Validate that user can update this progress
    await this.validateProgressUpdatePermission(userId, sectionId);

    // Validate progress data
    if (progressData.timeSpent < 0 || progressData.timeSpent > 24 * 60 * 60) {
      throw new SecurityError('Invalid time spent value');
    }

    const progress = await this.prisma.progress.upsert({
      where: {
        userId_sectionId: {
          userId: this.sanitizeUserId(userId),
          sectionId: this.sanitizeSectionId(sectionId)
        }
      },
      update: {
        timeSpent: { increment: progressData.timeSpent },
        completed: progressData.completed,
        updatedAt: new Date()
      },
      create: {
        userId: this.sanitizeUserId(userId),
        sectionId: this.sanitizeSectionId(sectionId),
        timeSpent: progressData.timeSpent,
        completed: progressData.completed
      }
    });

    return progress;
  }

  // Secure course retrieval with access control
  async getCourse(courseId: string, userId?: string): Promise<Course | null> {
    const sanitizedCourseId = this.sanitizeCourseId(courseId);
    
    const course = await this.prisma.course.findUnique({
      where: { id: sanitizedCourseId },
      include: {
        lessons: {
          include: {
            sections: {
              select: {
                id: true,
                title: true,
                type: true,
                order: true,
                duration: true,
                // Don't include content in listing for performance
              }
            }
          }
        }
      }
    });

    // Apply access control
    if (!course || (!course.isPublished && !this.isAuthorizedToViewUnpublished(userId))) {
      return null;
    }

    return course;
  }

  // Prevent SQL injection in dynamic queries
  async searchCourses(searchTerm: string, filters: SearchFilters): Promise<Course[]> {
    // Sanitize search term
    const sanitizedTerm = validator.escape(searchTerm.trim());
    
    if (sanitizedTerm.length < 2) {
      throw new SecurityError('Search term too short');
    }

    // Use Prisma's built-in protection against SQL injection
    const courses = await this.prisma.course.findMany({
      where: {
        AND: [
          { isPublished: true },
          {
            OR: [
              { title: { contains: sanitizedTerm, mode: 'insensitive' } },
              { description: { contains: sanitizedTerm, mode: 'insensitive' } }
            ]
          },
          filters.difficulty ? { difficulty: filters.difficulty } : {}
        ]
      },
      select: {
        id: true,
        title: true,
        description: true,
        difficulty: true,
        imageUrl: true,
        createdAt: true
      },
      take: 50, // Limit results to prevent large responses
      orderBy: { createdAt: 'desc' }
    });

    return courses;
  }

  private async validateProgressUpdatePermission(userId: string, sectionId: string): Promise<void> {
    // Check if section exists and is accessible
    const section = await this.prisma.section.findUnique({
      where: { id: sectionId },
      include: {
        lesson: {
          include: {
            course: true
          }
        }
      }
    });

    if (!section) {
      throw new SecurityError('Section not found');
    }

    if (!section.lesson.course.isPublished) {
      throw new SecurityError('Cannot update progress on unpublished course');
    }

    // Additional permission checks would go here when authentication is added
  }

  private sanitizeUserId(userId: string): string {
    if (!userId || typeof userId !== 'string') {
      throw new SecurityError('Invalid user ID');
    }
    return validator.escape(userId.trim());
  }

  private sanitizeCourseId(courseId: string): string {
    if (!courseId || typeof courseId !== 'string') {
      throw new SecurityError('Invalid course ID');
    }
    
    // Validate CUID format
    if (!/^c[a-z0-9]{24}$/.test(courseId)) {
      throw new SecurityError('Invalid course ID format');
    }
    
    return courseId;
  }

  private sanitizeSectionId(sectionId: string): string {
    if (!sectionId || typeof sectionId !== 'string') {
      throw new SecurityError('Invalid section ID');
    }
    
    // Validate CUID format
    if (!/^c[a-z0-9]{24}$/.test(sectionId)) {
      throw new SecurityError('Invalid section ID format');
    }
    
    return sectionId;
  }

  private isAuthorizedToViewUnpublished(userId?: string): boolean {
    // Implementation for checking if user can view unpublished content
    // Would integrate with authentication system
    return false; // Conservative default
  }

  private async logSecurityEvent(event: string, data: any): Promise<void> {
    // Log to security audit trail
    console.log(`SECURITY_EVENT: ${event}`, {
      timestamp: new Date().toISOString(),
      ...data
    });
    
    // In production, this would go to a secure logging system
  }
}

// Custom security error class
export class SecurityError extends Error {
  constructor(message: string, public details?: any) {
    super(message);
    this.name = 'SecurityError';
  }
}

// Type definitions
interface CourseCreationData {
  title: string;
  description?: string;
  difficulty: string;
}

interface ProgressUpdateData {
  timeSpent: number;
  completed: boolean;
}

interface SearchFilters {
  difficulty?: string;
  category?: string;
}
```

## Key Security Takeaways

### Critical Security Patterns ✅

1. **Input Validation and Sanitization**
   - Validate all user inputs before processing
   - Sanitize content to prevent XSS attacks
   - Check for SQL injection patterns in user input
   - Validate AI-generated content for safety

2. **API Security**
   - Implement rate limiting for all endpoints
   - Use security headers to prevent common attacks
   - Validate request format and size limits
   - Implement proper CORS configuration

3. **Database Security**
   - Use parameterized queries (Prisma provides this)
   - Implement access control for sensitive data
   - Validate data formats before database operations
   - Log security-relevant actions for audit trail

4. **Content Security**
   - Sanitize AI-generated educational content
   - Prevent prompt injection attacks
   - Implement content moderation for inappropriate material
   - Secure handling of user-generated content

5. **Infrastructure Security**
   - Configure proper security headers
   - Implement request size and rate limiting
   - Use HTTPS for all communications
   - Secure API key and secret management

### Security Anti-Patterns to Avoid ❌

1. **Trusting User Input** - Not validating or sanitizing any user-provided data
2. **Missing Rate Limiting** - Allowing unlimited requests that could abuse AI APIs
3. **Inadequate Content Filtering** - Not checking AI-generated content for safety
4. **Poor Error Handling** - Exposing sensitive information in error messages
5. **Missing Security Headers** - Not implementing basic web security protections

Security is paramount for educational platforms handling AI-generated content and user data. These patterns ensure safe, reliable operation while protecting against common web application vulnerabilities. 