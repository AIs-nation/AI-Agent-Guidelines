# LMS Privacy & Security Compliance Framework

## Educational Data Protection Overview

### FERPA & GDPR Compliance for Learning Management Systems
✅ **Good Example: Comprehensive Educational Privacy Architecture**
```
🛡️ LMS PRIVACY & SECURITY ARCHITECTURE

┌─────────────────────────────────────────────────────────────┐
│                 EDUCATIONAL DATA GOVERNANCE                 │
├─────────────────────────────────────────────────────────────┤
│ FERPA Compliance (US Educational Records)                  │
│ • Student Educational Records Protection                   │
│ • Parental Rights for Students Under 18                   │
│ • Directory Information Guidelines                         │
│ • Consent Requirements for Disclosure                      │
│                                                            │
│ GDPR Compliance (EU Personal Data Protection)              │
│ • Lawful Basis for Processing Educational Data            │
│ • Data Subject Rights (Access, Rectification, Erasure)    │
│ • Data Protection Impact Assessments                      │
│ • Privacy by Design in Educational Systems                │
│                                                            │
│ COPPA Compliance (Children Under 13)                       │
│ • Parental Consent for Data Collection                    │
│ • Limited Data Collection from Minors                     │
│ • Safe Harbor Provisions for Educational Platforms        │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                    DATA CLASSIFICATION MATRIX              │
├─────────────────────────────────────────────────────────────┤
│ PUBLIC DATA (No Restrictions)                              │
│ • Course Catalog Information                              │
│ • General Educational Content                             │
│ • Non-Personal Learning Resources                         │
│                                                            │
│ EDUCATIONAL RECORDS (FERPA Protected)                      │
│ • Student Grades and Progress                             │
│ • Learning Analytics and Performance Data                 │
│ • Individual Assignment Submissions                       │
│ • Attendance and Participation Records                    │
│                                                            │
│ PERSONAL DATA (GDPR Protected)                             │
│ • Student Names and Contact Information                   │
│ • IP Addresses and Device Information                     │
│ • Behavioral Tracking Data                                │
│ • Communication Records                                   │
│                                                            │
│ SENSITIVE DATA (Highest Protection)                        │
│ • Special Educational Needs Information                   │
│ • Health-Related Learning Accommodations                  │
│ • Financial Aid Information                               │
│ • Disciplinary Records                                    │
└─────────────────────────────────────────────────────────────┘
```

### Educational Privacy Implementation
✅ **Good Example: FERPA-Compliant Data Handling**
```javascript
// services/educationalPrivacyService.js - FERPA/GDPR compliance service
const crypto = require('crypto');
const { logAuditEvent } = require('./auditLoggingService');

class EducationalPrivacyService {
  constructor() {
    this.encryptionKey = process.env.EDUCATIONAL_DATA_KEY;
    this.auditRetentionYears = 7; // FERPA recommendation
  }

  // FERPA-Compliant Student Record Access
  async accessStudentRecord(requestedBy, studentId, purpose, justification) {
    try {
      // Verify legitimate educational interest
      const accessPermission = await this.validateEducationalInterest({
        requestedBy,
        studentId,
        purpose,
        justification
      });

      if (!accessPermission.authorized) {
        await this.logUnauthorizedAccess(requestedBy, studentId, purpose);
        throw new Error('Access denied: No legitimate educational interest');
      }

      // Audit log for FERPA compliance
      await logAuditEvent({
        eventType: 'EDUCATIONAL_RECORD_ACCESS',
        performedBy: requestedBy,
        studentId: this.hashStudentId(studentId),
        purpose,
        timestamp: new Date(),
        complianceFramework: 'FERPA',
        dataClassification: 'EDUCATIONAL_RECORD'
      });

      // Return anonymized data if appropriate
      const studentRecord = await this.getStudentRecord(studentId);
      return this.applyPrivacyFilters(studentRecord, accessPermission.scope);

    } catch (error) {
      console.error('Educational record access error:', error);
      throw new Error('Failed to access educational record in compliance with privacy regulations');
    }
  }

  // GDPR Data Subject Rights Implementation
  async processDataSubjectRequest(requestType, studentId, requestDetails) {
    const supportedRequests = [
      'ACCESS', 'RECTIFICATION', 'ERASURE', 
      'RESTRICTION', 'PORTABILITY', 'OBJECTION'
    ];

    if (!supportedRequests.includes(requestType)) {
      throw new Error(`Unsupported data subject request type: ${requestType}`);
    }

    try {
      switch (requestType) {
        case 'ACCESS':
          return await this.generateDataPortabilityReport(studentId);
        
        case 'RECTIFICATION':
          return await this.rectifyStudentData(studentId, requestDetails.corrections);
        
        case 'ERASURE':
          return await this.processRightToBeForgotten(studentId, requestDetails.reason);
        
        case 'RESTRICTION':
          return await this.restrictDataProcessing(studentId, requestDetails.scope);
        
        case 'PORTABILITY':
          return await this.generatePortableDataExport(studentId);
        
        case 'OBJECTION':
          return await this.processProcessingObjection(studentId, requestDetails.grounds);
      }
    } catch (error) {
      console.error(`GDPR ${requestType} request error:`, error);
      throw new Error(`Failed to process ${requestType} request under GDPR compliance`);
    }
  }

  // Educational Data Anonymization
  async anonymizeEducationalData(studentData, retentionPurpose) {
    const anonymizationLevel = this.determineAnonymizationLevel(retentionPurpose);
    
    const anonymizedData = {
      // Remove direct identifiers
      studentId: this.generatePseudonym(studentData.studentId),
      name: anonymizationLevel >= 2 ? null : this.anonymizeName(studentData.name),
      email: anonymizationLevel >= 2 ? null : this.anonymizeEmail(studentData.email),
      
      // Preserve educational value
      courseProgress: studentData.courseProgress,
      learningAnalytics: {
        timeSpent: studentData.timeSpent,
        completionRates: studentData.completionRates,
        difficultiesEncountered: studentData.difficulties,
        // Geographic data anonymized to region level
        learningRegion: this.anonymizeLocation(studentData.location)
      },
      
      // Educational outcomes (anonymous)
      performanceMetrics: {
        courseCompletions: studentData.completions,
        averageScores: studentData.scores,
        learningVelocity: studentData.velocity,
        engagementLevel: studentData.engagement
      },
      
      // Metadata for compliance
      anonymizationDate: new Date(),
      retentionPurpose,
      complianceFramework: ['FERPA', 'GDPR'],
      dataClassification: 'ANONYMIZED_EDUCATIONAL_ANALYTICS'
    };

    // Audit anonymization process
    await logAuditEvent({
      eventType: 'DATA_ANONYMIZATION',
      originalDataId: this.hashStudentId(studentData.studentId),
      anonymizationLevel,
      retentionPurpose,
      timestamp: new Date(),
      complianceFramework: 'FERPA_GDPR'
    });

    return anonymizedData;
  }

  // Child Protection (COPPA Compliance)
  async validateMinorDataCollection(studentAge, parentalConsent, dataTypes) {
    if (studentAge < 13) {
      // COPPA requirements
      if (!parentalConsent || !parentalConsent.verified) {
        throw new Error('Parental consent required for students under 13');
      }

      // Limit data collection for minors
      const allowedDataTypes = [
        'educational_progress', 'course_completion', 
        'learning_difficulties', 'assignment_submissions'
      ];

      const requestedTypes = dataTypes.filter(type => !allowedDataTypes.includes(type));
      if (requestedTypes.length > 0) {
        throw new Error(`Data types not permitted for minors: ${requestedTypes.join(', ')}`);
      }

      // Additional safeguards for minors
      await this.implementMinorProtections(studentAge);
    }

    return {
      dataCollectionApproved: true,
      appliedProtections: studentAge < 13 ? 'COPPA_ENHANCED' : 'STANDARD',
      parentalConsentRequired: studentAge < 13,
      dataRetentionPeriod: studentAge < 13 ? '1_YEAR' : 'STANDARD_RETENTION'
    };
  }

  // Privacy Helper Methods
  validateEducationalInterest({ requestedBy, purpose, justification }) {
    // FERPA legitimate educational interest validation
    const legitimateReasons = [
      'academic_support', 'progress_monitoring', 'curriculum_improvement',
      'academic_advising', 'learning_accommodation', 'graduation_requirements'
    ];

    const authorized = legitimateReasons.includes(purpose) && 
                     this.verifyEducatorRole(requestedBy) &&
                     justification.length >= 50;

    return {
      authorized,
      scope: this.determinateAccessScope(purpose),
      conditions: ['audit_logged', 'time_limited', 'purpose_limited']
    };
  }

  hashStudentId(studentId) {
    return crypto.createHash('sha256')
      .update(studentId + this.encryptionKey)
      .digest('hex')
      .substring(0, 16);
  }

  async generateDataPortabilityReport(studentId) {
    const studentData = await this.getComprehensiveStudentData(studentId);
    
    return {
      exportFormat: 'JSON',
      dataCategories: {
        personalInformation: studentData.profile,
        educationalRecords: studentData.academicHistory,
        learningAnalytics: studentData.analytics,
        preferences: studentData.settings,
        communications: studentData.messages
      },
      exportDate: new Date(),
      retentionNotice: 'This data export includes all personal data processed for educational purposes',
      rightsInformation: {
        rectification: 'You may request corrections to inaccurate data',
        erasure: 'You may request deletion under certain circumstances',
        restriction: 'You may request processing limitations',
        objection: 'You may object to certain processing activities'
      },
      complianceFramework: 'GDPR_Article_20'
    };
  }
}

module.exports = new EducationalPrivacyService();
```

❌ **Bad Example: Non-Compliant Educational Data Handling**
```javascript
// ❌ Poor privacy implementation
class BadEducationalService {
  // ❌ No privacy controls
  async getStudentData(studentId) {
    // ❌ No access control validation
    // ❌ No audit logging
    // ❌ No data classification
    // ❌ No anonymization options
    return await db.students.findById(studentId); // Exposes all data
  }

  // ❌ No GDPR compliance
  async deleteStudent(studentId) {
    // ❌ No right to be forgotten implementation
    // ❌ No backup data consideration
    // ❌ No audit trail
    await db.students.delete(studentId);
  }

  // ❌ No FERPA considerations
  async shareStudentProgress(studentId, recipientEmail) {
    // ❌ No legitimate educational interest validation
    // ❌ No consent verification
    // ❌ No data minimization
    const data = await this.getStudentData(studentId);
    await emailService.send(recipientEmail, data); // Potential FERPA violation
  }
}
```

## Authentication & Authorization for Educational Platforms

### Role-Based Access Control for Educational Environments
✅ **Good Example: Educational RBAC Implementation**
```javascript
// middleware/educationalAuth.js - Educational platform authorization
const jwt = require('jsonwebtoken');
const { logSecurityEvent } = require('../services/auditLoggingService');

// Educational Platform Role Hierarchy
const EDUCATIONAL_ROLES = {
  SYSTEM_ADMIN: {
    level: 100,
    permissions: ['*'], // Full system access
    dataAccess: ['ALL'],
    auditLevel: 'MAXIMUM'
  },
  INSTITUTION_ADMIN: {
    level: 80,
    permissions: [
      'manage_courses', 'manage_users', 'view_analytics', 
      'manage_content', 'configure_institution'
    ],
    dataAccess: ['INSTITUTION_SCOPE'],
    auditLevel: 'HIGH'
  },
  INSTRUCTOR: {
    level: 60,
    permissions: [
      'create_courses', 'manage_own_courses', 'view_student_progress',
      'grade_assignments', 'communicate_students'
    ],
    dataAccess: ['OWN_COURSES', 'ASSIGNED_STUDENTS'],
    auditLevel: 'MEDIUM',
    ferpaCompliance: true
  },
  TEACHING_ASSISTANT: {
    level: 50,
    permissions: [
      'assist_instruction', 'view_assigned_student_progress',
      'help_students', 'grade_basic_assignments'
    ],
    dataAccess: ['ASSIGNED_STUDENTS_LIMITED'],
    auditLevel: 'MEDIUM',
    supervisionRequired: true
  },
  STUDENT: {
    level: 30,
    permissions: [
      'enroll_courses', 'submit_assignments', 'view_own_progress',
      'communicate_instructors', 'use_learning_tools'
    ],
    dataAccess: ['OWN_DATA'],
    auditLevel: 'BASIC'
  },
  PARENT_GUARDIAN: {
    level: 25,
    permissions: [
      'view_child_progress', 'communicate_instructors',
      'manage_child_settings'
    ],
    dataAccess: ['DEPENDENT_CHILDREN'],
    auditLevel: 'BASIC',
    ferpaRights: true
  },
  GUEST: {
    level: 10,
    permissions: ['view_public_content', 'register_account'],
    dataAccess: ['PUBLIC_ONLY'],
    auditLevel: 'MINIMAL'
  }
};

// Educational Authorization Middleware
const requireEducationalRole = (requiredRoles = [], options = {}) => {
  return async (req, res, next) => {
    try {
      const token = req.headers.authorization?.replace('Bearer ', '');
      
      if (!token) {
        await logSecurityEvent({
          eventType: 'UNAUTHORIZED_ACCESS_ATTEMPT',
          ip: req.ip,
          userAgent: req.get('User-Agent'),
          path: req.path,
          timestamp: new Date()
        });
        
        return res.status(401).json({
          error: 'Authentication required',
          educationalContext: 'Access to educational resources requires valid authentication'
        });
      }

      const decoded = jwt.verify(token, process.env.JWT_SECRET);
      const user = await User.findById(decoded.userId)
        .populate('institutionAffiliation')
        .populate('courseAssignments');

      if (!user || !user.isActive) {
        return res.status(401).json({
          error: 'Invalid authentication',
          educationalContext: 'User account not found or inactive'
        });
      }

      // Validate educational role
      const userRole = EDUCATIONAL_ROLES[user.role];
      if (!userRole) {
        return res.status(403).json({
          error: 'Invalid educational role',
          educationalContext: 'User role not recognized in educational system'
        });
      }

      // Check role authorization
      const hasRequiredRole = requiredRoles.some(role => {
        const requiredLevel = EDUCATIONAL_ROLES[role]?.level || 0;
        return userRole.level >= requiredLevel;
      });

      if (!hasRequiredRole) {
        await logSecurityEvent({
          eventType: 'INSUFFICIENT_PERMISSIONS',
          userId: user.id,
          userRole: user.role,
          requiredRoles,
          path: req.path,
          timestamp: new Date()
        });

        return res.status(403).json({
          error: 'Insufficient educational permissions',
          educationalContext: `Access requires one of the following roles: ${requiredRoles.join(', ')}`
        });
      }

      // Educational data access validation
      if (options.requireDataAccess) {
        const hasDataAccess = await validateEducationalDataAccess(
          user, 
          options.requireDataAccess, 
          req.params
        );

        if (!hasDataAccess) {
          return res.status(403).json({
            error: 'Educational data access denied',
            educationalContext: 'Insufficient permissions for requested educational data'
          });
        }
      }

      // FERPA compliance check
      if (options.ferpaProtected && !userRole.ferpaCompliance) {
        return res.status(403).json({
          error: 'FERPA compliance required',
          educationalContext: 'Access to student educational records requires FERPA compliance training'
        });
      }

      // Attach educational context to request
      req.user = user;
      req.educationalRole = userRole;
      req.institutionContext = user.institutionAffiliation;
      
      next();

    } catch (error) {
      console.error('Educational authorization error:', error);
      res.status(500).json({
        error: 'Educational authorization system error',
        educationalContext: 'Unable to verify educational permissions'
      });
    }
  };
};

// Educational Data Access Validation
const validateEducationalDataAccess = async (user, requiredAccess, requestParams) => {
  const userRole = EDUCATIONAL_ROLES[user.role];
  
  switch (requiredAccess) {
    case 'OWN_DATA':
      return requestParams.userId === user.id;
    
    case 'ASSIGNED_STUDENTS':
      if (user.role === 'INSTRUCTOR') {
        const courseIds = user.courseAssignments.map(c => c.id);
        const student = await User.findById(requestParams.studentId);
        return student?.enrolledCourses.some(courseId => courseIds.includes(courseId));
      }
      return false;
    
    case 'INSTITUTION_SCOPE':
      return user.role === 'INSTITUTION_ADMIN' || user.role === 'SYSTEM_ADMIN';
    
    case 'DEPENDENT_CHILDREN':
      if (user.role === 'PARENT_GUARDIAN') {
        const child = await User.findById(requestParams.studentId);
        return child?.parentGuardians.includes(user.id);
      }
      return false;
    
    default:
      return userRole.dataAccess.includes(requiredAccess) || userRole.dataAccess.includes('ALL');
  }
};

// Course-Specific Authorization
const requireCourseAccess = (accessType = 'view') => {
  return async (req, res, next) => {
    try {
      const { courseId } = req.params;
      const user = req.user;

      const course = await Course.findById(courseId)
        .populate('instructor')
        .populate('teachingAssistants')
        .populate('enrolledStudents');

      if (!course) {
        return res.status(404).json({
          error: 'Course not found',
          educationalContext: 'Requested educational content is not available'
        });
      }

      let hasAccess = false;

      switch (accessType) {
        case 'view':
          hasAccess = course.isPublic ||
            course.instructor.id === user.id ||
            course.teachingAssistants.some(ta => ta.id === user.id) ||
            course.enrolledStudents.some(student => student.id === user.id) ||
            ['SYSTEM_ADMIN', 'INSTITUTION_ADMIN'].includes(user.role);
          break;

        case 'edit':
          hasAccess = course.instructor.id === user.id ||
            ['SYSTEM_ADMIN', 'INSTITUTION_ADMIN'].includes(user.role);
          break;

        case 'manage_students':
          hasAccess = course.instructor.id === user.id ||
            course.teachingAssistants.some(ta => ta.id === user.id) ||
            ['SYSTEM_ADMIN', 'INSTITUTION_ADMIN'].includes(user.role);
          break;

        case 'grade':
          hasAccess = course.instructor.id === user.id ||
            course.teachingAssistants.some(ta => ta.id === user.id);
          break;
      }

      if (!hasAccess) {
        return res.status(403).json({
          error: 'Course access denied',
          educationalContext: `Insufficient permissions for ${accessType} access to this course`
        });
      }

      req.course = course;
      req.courseAccess = accessType;
      
      next();

    } catch (error) {
      console.error('Course authorization error:', error);
      res.status(500).json({
        error: 'Course authorization system error'
      });
    }
  };
};

module.exports = {
  requireEducationalRole,
  requireCourseAccess,
  EDUCATIONAL_ROLES
};
```

## Data Security Implementation

### Educational Data Encryption & Protection
✅ **Good Example: Comprehensive Educational Data Security**
```javascript
// services/educationalSecurityService.js - Educational data protection
const crypto = require('crypto');
const bcrypt = require('bcrypt');
const { promisify } = require('util');

class EducationalSecurityService {
  constructor() {
    this.algorithm = 'aes-256-gcm';
    this.keyLength = 32;
    this.ivLength = 16;
    this.tagLength = 16;
    this.saltRounds = 12;
  }

  // Educational Record Encryption (FERPA Compliant)
  async encryptEducationalRecord(studentData, classification = 'EDUCATIONAL_RECORD') {
    try {
      const dataString = JSON.stringify(studentData);
      const key = await this.generateEducationalKey(classification);
      const iv = crypto.randomBytes(this.ivLength);
      
      const cipher = crypto.createCipher(this.algorithm, key, iv);
      
      let encrypted = cipher.update(dataString, 'utf8', 'hex');
      encrypted += cipher.final('hex');
      
      const authTag = cipher.getAuthTag();
      
      return {
        encryptedData: encrypted,
        iv: iv.toString('hex'),
        authTag: authTag.toString('hex'),
        algorithm: this.algorithm,
        classification,
        encryptedAt: new Date(),
        keyVersion: await this.getCurrentKeyVersion()
      };
    } catch (error) {
      console.error('Educational record encryption error:', error);
      throw new Error('Failed to encrypt educational record');
    }
  }

  // Educational Record Decryption with Audit
  async decryptEducationalRecord(encryptedRecord, requestedBy, purpose) {
    try {
      // Audit decryption request
      await this.auditDecryptionRequest(encryptedRecord, requestedBy, purpose);
      
      const key = await this.getEducationalKey(
        encryptedRecord.classification, 
        encryptedRecord.keyVersion
      );
      
      const decipher = crypto.createDecipher(
        encryptedRecord.algorithm,
        key,
        Buffer.from(encryptedRecord.iv, 'hex')
      );
      
      decipher.setAuthTag(Buffer.from(encryptedRecord.authTag, 'hex'));
      
      let decrypted = decipher.update(encryptedRecord.encryptedData, 'hex', 'utf8');
      decrypted += decipher.final('utf8');
      
      return JSON.parse(decrypted);
    } catch (error) {
      console.error('Educational record decryption error:', error);
      throw new Error('Failed to decrypt educational record');
    }
  }

  // Student Password Security (Educational Context)
  async hashStudentPassword(password, userId) {
    // Educational password requirements
    const passwordRequirements = {
      minLength: 8,
      requireUppercase: true,
      requireLowercase: true,
      requireNumbers: true,
      requireSpecialChars: true,
      prohibitPersonalInfo: true
    };

    await this.validateEducationalPassword(password, userId, passwordRequirements);
    
    const salt = await bcrypt.genSalt(this.saltRounds);
    const hashedPassword = await bcrypt.hash(password, salt);
    
    // Log password change for audit (FERPA requirement)
    await this.auditPasswordChange(userId, 'PASSWORD_CREATED');
    
    return {
      hashedPassword,
      salt,
      algorithm: 'bcrypt',
      saltRounds: this.saltRounds,
      passwordPolicy: passwordRequirements,
      changedAt: new Date()
    };
  }

  // Educational API Rate Limiting & Protection
  async protectEducationalAPI(req, options = {}) {
    const clientId = this.getClientIdentifier(req);
    const endpoint = req.path;
    
    // Educational-specific rate limits
    const rateLimits = {
      'course_generation': { limit: 5, window: 900000 }, // 5 per 15 minutes
      'student_data_access': { limit: 100, window: 60000 }, // 100 per minute
      'grade_submission': { limit: 50, window: 300000 }, // 50 per 5 minutes
      'analytics_query': { limit: 20, window: 60000 }, // 20 per minute
      'default': { limit: 1000, window: 900000 } // 1000 per 15 minutes
    };

    const endpointCategory = this.categorizeEducationalEndpoint(endpoint);
    const limit = rateLimits[endpointCategory] || rateLimits.default;
    
    const usage = await this.getAPIUsage(clientId, endpointCategory);
    
    if (usage.requests >= limit.limit) {
      await this.logRateLimitExceeded(clientId, endpoint, usage);
      
      throw new Error(`Educational API rate limit exceeded for ${endpointCategory}`);
    }
    
    await this.recordAPIUsage(clientId, endpointCategory);
    
    return {
      allowed: true,
      remaining: limit.limit - usage.requests - 1,
      resetTime: new Date(Date.now() + limit.window)
    };
  }

  // Educational Data Backup Security
  async secureEducationalBackup(backupData, institutionId) {
    const backupId = crypto.randomUUID();
    
    // Encrypt backup with institution-specific key
    const encryptedBackup = await this.encryptEducationalRecord(
      backupData, 
      'INSTITUTIONAL_BACKUP'
    );
    
    // Create backup integrity hash
    const integrityHash = crypto.createHash('sha256')
      .update(JSON.stringify(encryptedBackup))
      .digest('hex');
    
    // Generate backup metadata
    const backupMetadata = {
      backupId,
      institutionId,
      createdAt: new Date(),
      dataTypes: this.analyzeBackupDataTypes(backupData),
      encryptionStandard: 'AES-256-GCM',
      complianceFramework: ['FERPA', 'GDPR'],
      integrityHash,
      retentionPolicy: {
        activeRetention: '7_YEARS',
        archivalRetention: '25_YEARS',
        destructionSchedule: 'AFTER_LEGAL_REQUIREMENTS'
      }
    };
    
    return {
      backupId,
      encryptedBackup,
      metadata: backupMetadata,
      verificationHash: integrityHash
    };
  }

  // Educational Session Security
  async createSecureEducationalSession(user, loginContext) {
    const sessionId = crypto.randomUUID();
    const sessionKey = crypto.randomBytes(32).toString('hex');
    
    // Educational session data
    const sessionData = {
      sessionId,
      userId: user.id,
      userRole: user.role,
      institutionId: user.institutionId,
      loginTime: new Date(),
      loginIP: loginContext.ip,
      userAgent: loginContext.userAgent,
      educationalContext: {
        academicYear: await this.getCurrentAcademicYear(),
        semester: await this.getCurrentSemester(),
        timezone: user.preferences?.timezone || 'UTC'
      },
      securityLevel: this.determineSecurityLevel(user.role),
      complianceRequirements: this.getComplianceRequirements(user.role)
    };

    // Encrypt session data
    const encryptedSession = await this.encryptEducationalRecord(
      sessionData, 
      'SESSION_DATA'
    );

    // Store session with TTL
    await this.storeSecureSession(sessionId, encryptedSession, user.role);
    
    // Generate JWT with educational claims
    const token = jwt.sign({
      sessionId,
      userId: user.id,
      role: user.role,
      institutionId: user.institutionId,
      iat: Math.floor(Date.now() / 1000),
      exp: Math.floor(Date.now() / 1000) + this.getSessionDuration(user.role)
    }, process.env.JWT_SECRET);

    // Audit session creation
    await this.auditSessionCreation(user.id, sessionId, loginContext);
    
    return {
      token,
      sessionId,
      expiresAt: new Date(Date.now() + this.getSessionDuration(user.role) * 1000),
      securityLevel: sessionData.securityLevel
    };
  }

  // Helper Methods
  async generateEducationalKey(classification) {
    const baseKey = process.env.EDUCATIONAL_MASTER_KEY;
    const classificationSalt = classification.toLowerCase();
    
    return crypto.pbkdf2Sync(baseKey, classificationSalt, 100000, 32, 'sha256');
  }

  determineSecurityLevel(userRole) {
    const securityLevels = {
      'SYSTEM_ADMIN': 'MAXIMUM',
      'INSTITUTION_ADMIN': 'HIGH',
      'INSTRUCTOR': 'MEDIUM',
      'TEACHING_ASSISTANT': 'MEDIUM',
      'STUDENT': 'STANDARD',
      'PARENT_GUARDIAN': 'STANDARD',
      'GUEST': 'MINIMAL'
    };
    
    return securityLevels[userRole] || 'STANDARD';
  }

  getSessionDuration(userRole) {
    // Session duration in seconds based on role
    const durations = {
      'SYSTEM_ADMIN': 3600, // 1 hour
      'INSTITUTION_ADMIN': 7200, // 2 hours
      'INSTRUCTOR': 14400, // 4 hours
      'TEACHING_ASSISTANT': 14400, // 4 hours
      'STUDENT': 28800, // 8 hours
      'PARENT_GUARDIAN': 7200, // 2 hours
      'GUEST': 1800 // 30 minutes
    };
    
    return durations[userRole] || 7200;
  }
}

module.exports = new EducationalSecurityService();
```

## Key Educational Security Takeaways

### Critical Security Patterns for LMS Platforms ✅

1. **Educational Privacy Compliance**
   - Implement FERPA-compliant student record access controls
   - Build GDPR data subject rights handling for educational data
   - Create COPPA-compliant minor protection systems
   - Develop educational data anonymization for analytics
   - Establish audit trails for all educational data access

2. **Role-Based Educational Authorization**
   - Design educational role hierarchy with appropriate permissions
   - Implement course-specific access controls for instructors and students
   - Create parent/guardian access for dependent children
   - Build legitimate educational interest validation
   - Develop institution-scoped data access controls

3. **Educational Data Security**
   - Encrypt educational records with appropriate classification
   - Implement secure password policies for educational users
   - Create educational API rate limiting based on user roles
   - Build secure backup systems for institutional data
   - Develop educational session management with compliance tracking

4. **Audit & Compliance Systems**
   - Log all educational data access for FERPA compliance
   - Create comprehensive audit trails for privacy regulations
   - Implement data retention policies for educational records
   - Build compliance reporting for institutional requirements
   - Develop incident response for educational data breaches

### Educational Security Anti-Patterns to Avoid ❌

1. **Privacy Violations** - No FERPA/GDPR compliance, unrestricted data access, no anonymization
2. **Weak Authentication** - Basic password policies, no role-based access, missing audit trails
3. **Data Exposure** - Unencrypted educational records, excessive data sharing, no classification
4. **Compliance Gaps** - No consent management, missing retention policies, inadequate audit logging
5. **Session Insecurity** - Long session durations, no role-based timeouts, weak token management

Educational platform security requires specialized attention to student privacy, institutional compliance, role-based access patterns, and comprehensive audit systems while maintaining usability for diverse educational stakeholders. 