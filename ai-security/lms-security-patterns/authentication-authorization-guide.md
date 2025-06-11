# LMS Security Patterns and Best Practices

## Authentication Systems for Educational Platforms

### JWT-Based Authentication
**Implement secure token-based authentication for LMS systems**

✅ **Good Example: Secure JWT Implementation**
```javascript
// authService.js
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');
const crypto = require('crypto');

class AuthService {
  static async generateTokens(user) {
    const tokenPayload = {
      userId: user.id,
      email: user.email,
      roles: user.roles.map(role => role.name),
      organizationId: user.organizationId
    };

    // Short-lived access token (15 minutes)
    const accessToken = jwt.sign(
      tokenPayload,
      process.env.JWT_ACCESS_SECRET,
      { 
        expiresIn: '15m',
        issuer: 'lms-platform',
        audience: 'lms-users'
      }
    );

    // Long-lived refresh token (7 days)
    const refreshToken = jwt.sign(
      { userId: user.id, tokenVersion: user.tokenVersion },
      process.env.JWT_REFRESH_SECRET,
      { 
        expiresIn: '7d',
        issuer: 'lms-platform'
      }
    );

    // Store refresh token hash in database
    const refreshTokenHash = crypto
      .createHash('sha256')
      .update(refreshToken)
      .digest('hex');

    await this.storeRefreshToken(user.id, refreshTokenHash);

    return { accessToken, refreshToken };
  }

  static async verifyAccessToken(token) {
    try {
      const decoded = jwt.verify(token, process.env.JWT_ACCESS_SECRET, {
        issuer: 'lms-platform',
        audience: 'lms-users'
      });
      
      // Additional validation
      const user = await this.validateUser(decoded.userId);
      if (!user || !user.isActive) {
        throw new Error('User not active');
      }

      return decoded;
    } catch (error) {
      throw new Error('Invalid access token');
    }
  }

  static async refreshAccessToken(refreshToken) {
    try {
      const decoded = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET);
      
      // Verify refresh token exists in database
      const tokenHash = crypto
        .createHash('sha256')
        .update(refreshToken)
        .digest('hex');
      
      const storedToken = await this.getStoredRefreshToken(decoded.userId, tokenHash);
      if (!storedToken) {
        throw new Error('Refresh token not found');
      }

      const user = await this.getUserWithRoles(decoded.userId);
      if (!user || user.tokenVersion !== decoded.tokenVersion) {
        throw new Error('Token version mismatch');
      }

      // Generate new tokens
      return await this.generateTokens(user);
    } catch (error) {
      throw new Error('Invalid refresh token');
    }
  }
}

module.exports = AuthService;
```

❌ **Bad Example: Insecure Authentication**
```javascript
// Poor authentication implementation
class BadAuthService {
  static generateToken(user) {
    // No expiration, weak secret, sensitive data in payload
    return jwt.sign(
      { 
        ...user, // Includes password hash and sensitive data
        isAdmin: true // Trusting client-side data
      },
      'weak-secret', // Hardcoded weak secret
      {} // No options
    );
  }

  static verifyToken(token) {
    // No error handling, no validation
    return jwt.verify(token, 'weak-secret');
  }
}
```

### Role-Based Access Control (RBAC)
✅ **Good Example: Comprehensive RBAC System**
```javascript
// rbacService.js
class RBACService {
  static permissions = {
    // Course management
    'course:create': 'Create new courses',
    'course:read': 'View course content',
    'course:update': 'Edit course content',
    'course:delete': 'Delete courses',
    'course:publish': 'Publish/unpublish courses',
    
    // User management
    'user:read': 'View user profiles',
    'user:update': 'Edit user profiles',
    'user:delete': 'Delete users',
    'user:manage_roles': 'Assign/remove user roles',
    
    // Progress tracking
    'progress:read_own': 'View own progress',
    'progress:read_all': 'View all user progress',
    'progress:update': 'Update progress records',
    
    // Organization management
    'org:manage': 'Manage organization settings',
    'org:analytics': 'View organization analytics'
  };

  static rolePermissions = {
    'student': [
      'course:read',
      'progress:read_own'
    ],
    'instructor': [
      'course:create',
      'course:read',
      'course:update',
      'course:publish',
      'progress:read_all',
      'user:read'
    ],
    'admin': [
      'course:create',
      'course:read',
      'course:update',
      'course:delete',
      'course:publish',
      'user:read',
      'user:update',
      'user:manage_roles',
      'progress:read_all',
      'progress:update'
    ],
    'super_admin': Object.keys(this.permissions)
  };

  static async checkPermission(userId, permission, resourceId = null) {
    try {
      const user = await this.getUserWithRoles(userId);
      if (!user || !user.isActive) {
        return false;
      }

      // Get all permissions for user's roles
      const userPermissions = new Set();
      for (const role of user.roles) {
        const rolePerms = this.rolePermissions[role.name] || [];
        rolePerms.forEach(perm => userPermissions.add(perm));
      }

      // Check basic permission
      if (!userPermissions.has(permission)) {
        return false;
      }

      // Additional resource-specific checks
      if (resourceId) {
        return await this.checkResourceAccess(userId, permission, resourceId);
      }

      return true;
    } catch (error) {
      console.error('Permission check failed:', error);
      return false;
    }
  }

  static async checkResourceAccess(userId, permission, resourceId) {
    // Example: Check if user can access specific course
    if (permission.startsWith('course:')) {
      const course = await this.getCourse(resourceId);
      if (!course) return false;

      // Students can only access courses they're enrolled in
      if (permission === 'course:read') {
        const enrollment = await this.getEnrollment(userId, resourceId);
        return enrollment && enrollment.status === 'ACTIVE';
      }

      // Instructors can access courses they created or are assigned to
      if (permission === 'course:update') {
        return course.createdBy === userId || 
               await this.isInstructorAssigned(userId, resourceId);
      }
    }

    return true;
  }

  // Middleware for protecting routes
  static requirePermission(permission) {
    return async (req, res, next) => {
      try {
        const userId = req.user?.userId;
        const resourceId = req.params.id || req.params.courseId;

        const hasPermission = await this.checkPermission(userId, permission, resourceId);
        
        if (!hasPermission) {
          return res.status(403).json({
            success: false,
            message: 'Insufficient permissions',
            code: 'FORBIDDEN'
          });
        }

        next();
      } catch (error) {
        return res.status(500).json({
          success: false,
          message: 'Permission check failed'
        });
      }
    };
  }
}

// Usage in routes
app.get('/api/courses/:id', 
  authenticateToken,
  RBACService.requirePermission('course:read'),
  courseController.getCourse
);

app.post('/api/courses', 
  authenticateToken,
  RBACService.requirePermission('course:create'),
  courseController.createCourse
);
```

## Data Protection and Privacy

### Sensitive Data Encryption
✅ **Good Example: Data Encryption at Rest and in Transit**
```javascript
// encryptionService.js
const crypto = require('crypto');

class EncryptionService {
  static algorithm = 'aes-256-gcm';
  static keyLength = 32;
  static ivLength = 16;
  static tagLength = 16;

  static encrypt(text, key = process.env.ENCRYPTION_KEY) {
    try {
      const iv = crypto.randomBytes(this.ivLength);
      const cipher = crypto.createCipher(this.algorithm, Buffer.from(key, 'hex'));
      
      cipher.setAAD(Buffer.from('LMS-Platform', 'utf8'));
      
      let encrypted = cipher.update(text, 'utf8', 'hex');
      encrypted += cipher.final('hex');
      
      const tag = cipher.getAuthTag();
      
      return {
        iv: iv.toString('hex'),
        encryptedData: encrypted,
        tag: tag.toString('hex')
      };
    } catch (error) {
      throw new Error('Encryption failed');
    }
  }

  static decrypt(encryptedObj, key = process.env.ENCRYPTION_KEY) {
    try {
      const decipher = crypto.createDecipher(
        this.algorithm,
        Buffer.from(key, 'hex')
      );
      
      decipher.setAAD(Buffer.from('LMS-Platform', 'utf8'));
      decipher.setAuthTag(Buffer.from(encryptedObj.tag, 'hex'));
      
      let decrypted = decipher.update(encryptedObj.encryptedData, 'hex', 'utf8');
      decrypted += decipher.final('utf8');
      
      return decrypted;
    } catch (error) {
      throw new Error('Decryption failed');
    }
  }

  // Hash sensitive data (one-way)
  static hashSensitiveData(data, salt = null) {
    const saltToUse = salt || crypto.randomBytes(32).toString('hex');
    const hash = crypto.pbkdf2Sync(data, saltToUse, 10000, 64, 'sha512');
    
    return {
      hash: hash.toString('hex'),
      salt: saltToUse
    };
  }

  // Verify hashed data
  static verifyHashedData(data, hash, salt) {
    const hashToVerify = crypto.pbkdf2Sync(data, salt, 10000, 64, 'sha512');
    return hashToVerify.toString('hex') === hash;
  }
}

// Usage in user service
class UserService {
  static async createUser(userData) {
    // Hash password
    const hashedPassword = await bcrypt.hash(userData.password, 12);
    
    // Encrypt PII (Personally Identifiable Information)
    const encryptedEmail = EncryptionService.encrypt(userData.email);
    const encryptedPhone = userData.phone ? 
      EncryptionService.encrypt(userData.phone) : null;

    const user = await prisma.user.create({
      data: {
        username: userData.username,
        passwordHash: hashedPassword,
        encryptedEmail: JSON.stringify(encryptedEmail),
        encryptedPhone: encryptedPhone ? JSON.stringify(encryptedPhone) : null,
        firstName: userData.firstName,
        lastName: userData.lastName
      }
    });

    return this.formatUserResponse(user);
  }

  static formatUserResponse(user) {
    // Decrypt PII when returning user data
    const email = EncryptionService.decrypt(JSON.parse(user.encryptedEmail));
    const phone = user.encryptedPhone ? 
      EncryptionService.decrypt(JSON.parse(user.encryptedPhone)) : null;

    return {
      id: user.id,
      username: user.username,
      email,
      phone,
      firstName: user.firstName,
      lastName: user.lastName,
      isActive: user.isActive
      // Never return password hash
    };
  }
}
```

### Input Validation and Sanitization
✅ **Good Example: Comprehensive Input Validation**
```javascript
// validationService.js
const Joi = require('joi');
const validator = require('validator');
const DOMPurify = require('isomorphic-dompurify');

class ValidationService {
  // Course content validation
  static courseSchema = Joi.object({
    title: Joi.string()
      .min(3)
      .max(100)
      .pattern(/^[a-zA-Z0-9\s\-_.:]+$/)
      .required(),
    description: Joi.string()
      .max(1000)
      .optional(),
    difficulty: Joi.string()
      .valid('BEGINNER', 'INTERMEDIATE', 'ADVANCED', 'EXPERT')
      .required(),
    estimatedHours: Joi.number()
      .integer()
      .min(1)
      .max(1000)
      .required(),
    content: Joi.string()
      .max(100000) // 100KB limit for content
      .custom(this.sanitizeHTML)
      .required()
  });

  // User input validation
  static userSchema = Joi.object({
    username: Joi.string()
      .alphanum()
      .min(3)
      .max(30)
      .required(),
    email: Joi.string()
      .email()
      .custom(this.validateEmail)
      .required(),
    password: Joi.string()
      .min(8)
      .max(128)
      .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/)
      .required(),
    firstName: Joi.string()
      .min(1)
      .max(50)
      .pattern(/^[a-zA-Z\s\-']+$/)
      .required(),
    lastName: Joi.string()
      .min(1)
      .max(50)
      .pattern(/^[a-zA-Z\s\-']+$/)
      .required()
  });

  static sanitizeHTML(value, helpers) {
    try {
      // Allow only safe HTML tags for educational content
      const cleanHTML = DOMPurify.sanitize(value, {
        ALLOWED_TAGS: [
          'p', 'br', 'strong', 'em', 'u', 'ol', 'ul', 'li',
          'h1', 'h2', 'h3', 'h4', 'h5', 'h6',
          'blockquote', 'code', 'pre',
          'a', 'img'
        ],
        ALLOWED_ATTR: [
          'href', 'src', 'alt', 'title', 'target'
        ],
        ALLOWED_URI_REGEXP: /^(?:(?:(?:f|ht)tps?|mailto|tel|callto|cid|xmpp):|[^a-z]|[a-z+.\-]+(?:[^a-z+.\-:]|$))/i
      });

      if (cleanHTML !== value) {
        return helpers.error('string.unsafe_html');
      }

      return cleanHTML;
    } catch (error) {
      return helpers.error('string.invalid_html');
    }
  }

  static validateEmail(value, helpers) {
    // Additional email validation beyond Joi
    if (!validator.isEmail(value)) {
      return helpers.error('string.email');
    }

    // Check for disposable email domains
    const disposableDomains = [
      '10minutemail.com', 'tempmail.org', 'guerrillamail.com'
      // Add more disposable domains
    ];

    const domain = value.split('@')[1];
    if (disposableDomains.includes(domain.toLowerCase())) {
      return helpers.error('string.disposable_email');
    }

    return value;
  }

  // SQL Injection prevention
  static sanitizeForDatabase(input) {
    if (typeof input !== 'string') return input;
    
    // Remove SQL keywords and special characters
    const sqlKeywords = [
      'SELECT', 'INSERT', 'UPDATE', 'DELETE', 'DROP', 'CREATE',
      'ALTER', 'EXEC', 'UNION', 'SCRIPT', '--', ';'
    ];

    let sanitized = input;
    sqlKeywords.forEach(keyword => {
      const regex = new RegExp(keyword, 'gi');
      sanitized = sanitized.replace(regex, '');
    });

    return validator.escape(sanitized);
  }

  // Middleware for request validation
  static validateRequest(schema) {
    return (req, res, next) => {
      const { error, value } = schema.validate(req.body, {
        abortEarly: false,
        stripUnknown: true
      });

      if (error) {
        const errors = error.details.map(detail => ({
          field: detail.path.join('.'),
          message: detail.message,
          code: detail.type
        }));

        return res.status(400).json({
          success: false,
          message: 'Validation failed',
          errors
        });
      }

      req.body = value;
      next();
    };
  }
}

module.exports = ValidationService;
```

## API Security Patterns

### Rate Limiting and DDoS Protection
✅ **Good Example: Advanced Rate Limiting**
```javascript
// rateLimitService.js
const Redis = require('redis');
const rateLimit = require('express-rate-limit');

class RateLimitService {
  constructor() {
    this.redis = Redis.createClient({
      host: process.env.REDIS_HOST,
      port: process.env.REDIS_PORT,
      password: process.env.REDIS_PASSWORD
    });
  }

  // Different rate limits for different endpoints
  static createAPIRateLimit() {
    return rateLimit({
      windowMs: 15 * 60 * 1000, // 15 minutes
      max: 100, // General API limit
      message: {
        success: false,
        message: 'Too many requests, please try again later',
        code: 'RATE_LIMIT_EXCEEDED'
      },
      standardHeaders: true,
      legacyHeaders: false,
      handler: this.rateLimitHandler
    });
  }

  static createAuthRateLimit() {
    return rateLimit({
      windowMs: 15 * 60 * 1000, // 15 minutes
      max: 5, // Strict limit for auth endpoints
      skipSuccessfulRequests: true,
      message: {
        success: false,
        message: 'Too many authentication attempts',
        code: 'AUTH_RATE_LIMIT_EXCEEDED'
      }
    });
  }

  static createCourseGenerationLimit() {
    return rateLimit({
      windowMs: 60 * 60 * 1000, // 1 hour
      max: 10, // Course generation is expensive
      keyGenerator: (req) => {
        // Rate limit per user for course generation
        return req.user?.userId || req.ip;
      },
      message: {
        success: false,
        message: 'Course generation limit exceeded',
        code: 'GENERATION_LIMIT_EXCEEDED'
      }
    });
  }

  // Advanced rate limiting with sliding window
  async checkSlidingWindowRateLimit(key, limit, windowSeconds) {
    const now = Date.now();
    const window = windowSeconds * 1000;
    const pipeline = this.redis.pipeline();

    // Remove old entries
    pipeline.zremrangebyscore(key, 0, now - window);
    
    // Count current requests
    pipeline.zcard(key);
    
    // Add current request
    pipeline.zadd(key, now, `${now}-${Math.random()}`);
    
    // Set expiration
    pipeline.expire(key, windowSeconds);

    const results = await pipeline.exec();
    const currentCount = results[1][1];

    return {
      allowed: currentCount < limit,
      count: currentCount,
      limit,
      resetTime: now + window
    };
  }

  static rateLimitHandler(req, res) {
    // Log rate limit violations
    console.warn('Rate limit exceeded:', {
      ip: req.ip,
      userAgent: req.get('User-Agent'),
      endpoint: req.path,
      userId: req.user?.userId
    });

    res.status(429).json({
      success: false,
      message: 'Rate limit exceeded',
      retryAfter: Math.round(req.rateLimit.resetTime / 1000)
    });
  }
}

// Usage in routes
app.use('/api/', RateLimitService.createAPIRateLimit());
app.use('/api/auth/', RateLimitService.createAuthRateLimit());
app.use('/api/courses/generate', RateLimitService.createCourseGenerationLimit());
```

### CORS and Security Headers
✅ **Good Example: Secure CORS Configuration**
```javascript
// securityConfig.js
const cors = require('cors');
const helmet = require('helmet');

class SecurityConfig {
  static setupCORS() {
    const allowedOrigins = process.env.ALLOWED_ORIGINS?.split(',') || [
      'http://localhost:3000',
      'https://your-lms-domain.com'
    ];

    return cors({
      origin: (origin, callback) => {
        // Allow requests with no origin (mobile apps, etc.)
        if (!origin) return callback(null, true);
        
        if (allowedOrigins.includes(origin)) {
          return callback(null, true);
        }
        
        callback(new Error('Not allowed by CORS'));
      },
      credentials: true,
      methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
      allowedHeaders: [
        'Content-Type',
        'Authorization',
        'X-Requested-With',
        'X-API-Key'
      ],
      exposedHeaders: ['X-Total-Count', 'X-Rate-Limit-Remaining'],
      maxAge: 86400, // 24 hours
      optionsSuccessStatus: 200
    });
  }

  static setupSecurityHeaders() {
    return helmet({
      contentSecurityPolicy: {
        directives: {
          defaultSrc: ["'self'"],
          styleSrc: ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"],
          fontSrc: ["'self'", "https://fonts.gstatic.com"],
          scriptSrc: ["'self'"],
          imgSrc: ["'self'", "data:", "https:"],
          connectSrc: ["'self'", "https://api.your-lms.com"],
          mediaSrc: ["'self'"],
          objectSrc: ["'none'"],
          baseUri: ["'self'"],
          formAction: ["'self'"],
          frameAncestors: ["'none'"],
          upgradeInsecureRequests: []
        }
      },
      crossOriginEmbedderPolicy: false,
      crossOriginOpenerPolicy: { policy: "cross-origin" },
      crossOriginResourcePolicy: { policy: "cross-origin" },
      dnsPrefetchControl: true,
      frameguard: { action: 'deny' },
      hidePoweredBy: true,
      hsts: {
        maxAge: 31536000,
        includeSubDomains: true,
        preload: true
      },
      ieNoOpen: true,
      noSniff: true,
      originAgentCluster: true,
      permittedCrossDomainPolicies: false,
      referrerPolicy: { policy: "no-referrer" },
      xssFilter: true
    });
  }

  // Custom security middleware
  static additionalSecurityHeaders() {
    return (req, res, next) => {
      // Remove server information
      res.removeHeader('X-Powered-By');
      
      // Add custom security headers
      res.setHeader('X-Content-Type-Options', 'nosniff');
      res.setHeader('X-Frame-Options', 'DENY');
      res.setHeader('X-XSS-Protection', '1; mode=block');
      res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
      res.setHeader('Cache-Control', 'no-store, no-cache, must-revalidate, proxy-revalidate');
      res.setHeader('Pragma', 'no-cache');
      res.setHeader('Expires', '0');
      
      next();
    };
  }
}

// Apply security configuration
app.use(SecurityConfig.setupCORS());
app.use(SecurityConfig.setupSecurityHeaders());
app.use(SecurityConfig.additionalSecurityHeaders());
```

## Audit Logging and Monitoring

### Comprehensive Audit Trail
✅ **Good Example: Security Audit Logging**
```javascript
// auditService.js
class AuditService {
  static async logSecurityEvent(eventType, details, req) {
    const auditLog = {
      eventType,
      timestamp: new Date(),
      userId: req.user?.userId || null,
      sessionId: req.sessionID,
      ipAddress: this.getClientIP(req),
      userAgent: req.get('User-Agent'),
      endpoint: req.originalUrl,
      method: req.method,
      details,
      severity: this.determineSeverity(eventType),
      source: 'LMS-Backend'
    };

    await this.storeAuditLog(auditLog);
    
    // Alert on high severity events
    if (auditLog.severity === 'HIGH' || auditLog.severity === 'CRITICAL') {
      await this.sendSecurityAlert(auditLog);
    }
  }

  static getClientIP(req) {
    return req.headers['x-forwarded-for'] ||
           req.headers['x-real-ip'] ||
           req.connection.remoteAddress ||
           req.socket.remoteAddress ||
           req.ip;
  }

  static determineSeverity(eventType) {
    const severityMap = {
      'LOGIN_SUCCESS': 'LOW',
      'LOGIN_FAILURE': 'MEDIUM',
      'LOGIN_BRUTE_FORCE': 'HIGH',
      'PASSWORD_CHANGE': 'MEDIUM',
      'PERMISSION_DENIED': 'MEDIUM',
      'UNAUTHORIZED_ACCESS': 'HIGH',
      'DATA_EXPORT': 'HIGH',
      'ADMIN_ACTION': 'HIGH',
      'SUSPICIOUS_ACTIVITY': 'CRITICAL',
      'SYSTEM_BREACH': 'CRITICAL'
    };

    return severityMap[eventType] || 'LOW';
  }

  static async logAuthenticationEvent(event, user, req, success = true) {
    const eventType = success ? 'LOGIN_SUCCESS' : 'LOGIN_FAILURE';
    
    await this.logSecurityEvent(eventType, {
      userId: user?.id,
      email: user?.email,
      success,
      event,
      failureReason: success ? null : event
    }, req);

    // Track failed login attempts
    if (!success) {
      await this.trackFailedLoginAttempt(req);
    }
  }

  static async trackFailedLoginAttempt(req) {
    const key = `failed_logins:${this.getClientIP(req)}`;
    const attempts = await redis.incr(key);
    await redis.expire(key, 900); // 15 minutes

    // Detect brute force attempts
    if (attempts >= 5) {
      await this.logSecurityEvent('LOGIN_BRUTE_FORCE', {
        attempts,
        ipAddress: this.getClientIP(req),
        timeWindow: '15 minutes'
      }, req);

      // Implement IP blocking or additional security measures
      await this.blockSuspiciousIP(this.getClientIP(req));
    }
  }

  static async logDataAccess(action, resource, req) {
    await this.logSecurityEvent('DATA_ACCESS', {
      action,
      resource: {
        type: resource.type,
        id: resource.id,
        sensitive: resource.containsPII || false
      },
      userId: req.user?.userId
    }, req);
  }

  // Middleware for automatic audit logging
  static auditMiddleware() {
    return (req, res, next) => {
      const startTime = Date.now();

      // Log the original response end
      const originalEnd = res.end;
      res.end = function(...args) {
        const duration = Date.now() - startTime;
        
        // Log API access
        AuditService.logAPIAccess({
          method: req.method,
          endpoint: req.originalUrl,
          statusCode: res.statusCode,
          duration,
          userId: req.user?.userId
        }, req);

        originalEnd.apply(this, args);
      };

      next();
    };
  }
}

// Usage in authentication routes
app.post('/api/auth/login', async (req, res) => {
  try {
    const { email, password } = req.body;
    const user = await AuthService.login(email, password);
    
    await AuditService.logAuthenticationEvent('LOGIN', user, req, true);
    
    res.json({ success: true, user });
  } catch (error) {
    await AuditService.logAuthenticationEvent(error.message, null, req, false);
    
    res.status(401).json({ success: false, message: 'Authentication failed' });
  }
});
```

## Key Security Takeaways for LMS Systems

### Critical Security Checklist ✅

1. **Authentication & Authorization**
   - Implement JWT with short-lived access tokens and refresh tokens
   - Use role-based access control (RBAC) with granular permissions
   - Enforce strong password policies and account lockout mechanisms

2. **Data Protection**
   - Encrypt sensitive data at rest using AES-256-GCM
   - Use HTTPS/TLS 1.3 for all data in transit
   - Implement field-level encryption for PII

3. **Input Validation**
   - Validate and sanitize all user inputs
   - Use parameterized queries to prevent SQL injection
   - Implement HTML sanitization for rich content

4. **API Security**
   - Implement rate limiting with different tiers for different endpoints
   - Use proper CORS configuration
   - Apply comprehensive security headers

5. **Monitoring & Auditing**
   - Log all security-relevant events
   - Implement real-time monitoring for suspicious activities
   - Set up automated alerts for security incidents

### Common Security Vulnerabilities to Avoid ❌

1. **Weak Authentication** - Using weak secrets, no token expiration, client-side role validation
2. **Insufficient Authorization** - Missing permission checks, role escalation vulnerabilities
3. **Data Exposure** - Logging sensitive data, returning sensitive info in API responses
4. **Injection Attacks** - SQL injection, XSS, command injection vulnerabilities
5. **Insecure Configuration** - Default passwords, exposed debug information, weak CORS policies

Educational platforms handle sensitive student data and must maintain the highest security standards. These patterns ensure comprehensive protection while maintaining usability for legitimate users. 