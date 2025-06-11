# Advanced Backend Architecture for LMS Microservices - 2025 Educational Platform Engineering

## 🔄 Fundamental Principle: Constant Research & Web Updates
This guide incorporates 2025 microservices architecture patterns, Node.js Express best practices, and educational platform engineering requirements for scalable LMS development.

## Overview: Educational Microservices Architecture

Modern Learning Management Systems require sophisticated backend architecture with microservices for scalability, educational data compliance, and seamless API integration. This guide covers advanced patterns for educational platform engineering.

## Core Microservices Architecture for Educational Platforms

### 1. Educational Domain-Driven Design (DDD) Architecture
```typescript
// Educational bounded contexts and domain models
interface LearningDomain {
  // Course Management Context
  courseManagement: {
    courses: Course[];
    lessons: Lesson[];
    sections: Section[];
    instructors: Instructor[];
  };
  
  // Learning Progress Context
  learningProgress: {
    enrollments: Enrollment[];
    progress: LearningProgress[];
    achievements: Achievement[];
    analytics: LearningAnalytics[];
  };
  
  // Assessment Context
  assessments: {
    quizzes: Quiz[];
    assignments: Assignment[];
    grades: Grade[];
    feedback: Feedback[];
  };
  
  // User Management Context
  userManagement: {
    students: Student[];
    instructors: Instructor[];
    administrators: Administrator[];
    permissions: Permission[];
  };
  
  // Communication Context
  communication: {
    notifications: Notification[];
    messages: Message[];
    announcements: Announcement[];
    forums: ForumPost[];
  };
}

// Educational aggregate roots with business logic
class CourseAggregate {
  private course: Course;
  private lessons: Lesson[];
  private enrollments: Enrollment[];

  constructor(courseData: CreateCourseData) {
    this.course = new Course(courseData);
    this.lessons = [];
    this.enrollments = [];
  }

  // Educational business rules
  enrollStudent(studentId: string, enrollmentData: EnrollmentData): void {
    if (this.course.isEnrollmentOpen()) {
      if (this.course.hasCapacity()) {
        if (this.course.meetsPrerequirements(studentId)) {
          const enrollment = new Enrollment({
            studentId,
            courseId: this.course.id,
            enrolledAt: new Date(),
            ...enrollmentData
          });
          
          this.enrollments.push(enrollment);
          this.course.incrementEnrollmentCount();
          
          // Emit domain event
          DomainEvents.raise(new StudentEnrolledEvent({
            studentId,
            courseId: this.course.id,
            enrollmentId: enrollment.id
          }));
        } else {
          throw new PrerequisitesNotMetError('Student does not meet course prerequisites');
        }
      } else {
        throw new CourseCapacityExceededError('Course has reached maximum capacity');
      }
    } else {
      throw new EnrollmentClosedError('Enrollment period has ended');
    }
  }

  completeLesson(studentId: string, lessonId: string): void {
    const enrollment = this.findEnrollment(studentId);
    if (!enrollment) {
      throw new NotEnrolledError('Student is not enrolled in this course');
    }

    const lesson = this.findLesson(lessonId);
    if (!lesson) {
      throw new LessonNotFoundError('Lesson not found in this course');
    }

    enrollment.markLessonCompleted(lessonId);
    
    // Check if course is completed
    if (enrollment.isCompleted(this.lessons)) {
      enrollment.markCourseCompleted();
      
      DomainEvents.raise(new CourseCompletedEvent({
        studentId,
        courseId: this.course.id,
        completedAt: new Date()
      }));
    }
  }
}
```

### 2. Advanced Express.js Microservice Architecture
```typescript
// Educational microservice base class with common functionality
abstract class EducationalMicroservice {
  protected app: Express;
  protected server: Server;
  protected database: DatabaseConnection;
  protected eventBus: EventBus;
  protected logger: Logger;

  constructor(
    protected serviceName: string,
    protected port: number,
    protected config: ServiceConfig
  ) {
    this.app = express();
    this.logger = createLogger(serviceName);
    this.setupMiddleware();
    this.setupRoutes();
    this.setupErrorHandling();
  }

  private setupMiddleware(): void {
    // Educational-specific middleware
    this.app.use(helmet({
      contentSecurityPolicy: {
        directives: {
          defaultSrc: ["'self'"],
          styleSrc: ["'self'", "'unsafe-inline'"],
          scriptSrc: ["'self'"],
          mediaSrc: ["'self'", "data:", "https:"],
          frameSrc: ["'none'"], // Educational security
        },
      },
    }));

    this.app.use(compression());
    this.app.use(express.json({ limit: '10mb' })); // Educational content can be large
    this.app.use(express.urlencoded({ extended: true }));
    
    // Educational CORS configuration
    this.app.use(cors({
      origin: this.config.allowedOrigins,
      credentials: true,
      methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
      allowedHeaders: ['Content-Type', 'Authorization', 'X-Student-Id', 'X-Institution-Id']
    }));

    // Educational rate limiting
    const educationalRateLimit = rateLimit({
      windowMs: 15 * 60 * 1000, // 15 minutes
      max: (req) => {
        // Different limits based on user role
        const userRole = req.headers['x-user-role'] as string;
        switch (userRole) {
          case 'student': return 100;
          case 'instructor': return 200;
          case 'admin': return 500;
          default: return 50;
        }
      },
      message: 'Educational platform rate limit exceeded',
      standardHeaders: true,
      legacyHeaders: false,
    });

    this.app.use(educationalRateLimit);

    // Educational request logging
    this.app.use(morgan('combined', {
      stream: {
        write: (message) => {
          this.logger.info(message.trim(), {
            service: this.serviceName,
            type: 'access_log'
          });
        }
      }
    }));

    // Educational context middleware
    this.app.use(this.educationalContextMiddleware.bind(this));
  }

  private educationalContextMiddleware(req: Request, res: Response, next: NextFunction): void {
    // Extract educational context from headers
    const educationalContext = {
      studentId: req.headers['x-student-id'] as string,
      instructorId: req.headers['x-instructor-id'] as string,
      institutionId: req.headers['x-institution-id'] as string,
      courseId: req.headers['x-course-id'] as string,
      userRole: req.headers['x-user-role'] as string,
      sessionId: req.headers['x-session-id'] as string,
    };

    // Attach to request for use in controllers
    (req as any).educationalContext = educationalContext;

    // Educational analytics tracking
    this.trackEducationalRequest(req, educationalContext);

    next();
  }

  private trackEducationalRequest(req: Request, context: any): void {
    this.logger.info('Educational request received', {
      method: req.method,
      path: req.path,
      userRole: context.userRole,
      institutionId: context.institutionId,
      timestamp: new Date().toISOString()
    });
  }

  abstract setupRoutes(): void;

  private setupErrorHandling(): void {
    // Educational error handling
    this.app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
      const educationalContext = (req as any).educationalContext;
      
      // Log educational error with context
      this.logger.error('Educational service error', {
        error: err.message,
        stack: err.stack,
        path: req.path,
        method: req.method,
        userRole: educationalContext?.userRole,
        institutionId: educationalContext?.institutionId,
        service: this.serviceName
      });

      // Educational-specific error responses
      if (err instanceof ValidationError) {
        return res.status(400).json({
          error: 'Educational data validation failed',
          message: err.message,
          details: err.details
        });
      }

      if (err instanceof UnauthorizedError) {
        return res.status(401).json({
          error: 'Educational access denied',
          message: 'Please verify your educational credentials'
        });
      }

      if (err instanceof ForbiddenError) {
        return res.status(403).json({
          error: 'Educational permission denied',
          message: 'Your role does not allow this educational action'
        });
      }

      // Generic educational error response
      res.status(500).json({
        error: 'Educational service unavailable',
        message: 'Please try again or contact educational support',
        timestamp: new Date().toISOString()
      });
    });
  }

  async start(): Promise<void> {
    try {
      await this.connectDatabase();
      await this.connectEventBus();
      
      this.server = this.app.listen(this.port, () => {
        this.logger.info(`${this.serviceName} educational microservice started`, {
          port: this.port,
          environment: process.env.NODE_ENV
        });
      });

      // Graceful shutdown
      process.on('SIGTERM', this.gracefulShutdown.bind(this));
      process.on('SIGINT', this.gracefulShutdown.bind(this));

    } catch (error) {
      this.logger.error(`Failed to start ${this.serviceName}`, { error });
      process.exit(1);
    }
  }

  private async gracefulShutdown(): Promise<void> {
    this.logger.info(`Gracefully shutting down ${this.serviceName}`);
    
    this.server.close(() => {
      this.logger.info(`${this.serviceName} server closed`);
    });

    await this.database.close();
    await this.eventBus.disconnect();
    
    process.exit(0);
  }

  abstract connectDatabase(): Promise<void>;
  abstract connectEventBus(): Promise<void>;
}

// Course Management Microservice Implementation
class CourseManagementService extends EducationalMicroservice {
  private courseRepository: CourseRepository;
  private lessonRepository: LessonRepository;

  constructor() {
    super('course-management', 3001, courseServiceConfig);
  }

  async connectDatabase(): Promise<void> {
    this.database = await createDatabaseConnection({
      host: this.config.database.host,
      port: this.config.database.port,
      database: 'educational_courses',
      username: this.config.database.username,
      password: this.config.database.password,
      pool: {
        min: 5,
        max: 20,
        acquire: 30000,
        idle: 10000
      }
    });

    this.courseRepository = new CourseRepository(this.database);
    this.lessonRepository = new LessonRepository(this.database);
  }

  async connectEventBus(): Promise<void> {
    this.eventBus = new EventBus({
      url: this.config.eventBus.url,
      exchange: 'educational-events',
      serviceName: this.serviceName
    });

    await this.eventBus.connect();
    this.setupEventHandlers();
  }

  private setupEventHandlers(): void {
    // Listen for educational events from other services
    this.eventBus.subscribe('student.enrolled', this.handleStudentEnrolled.bind(this));
    this.eventBus.subscribe('lesson.completed', this.handleLessonCompleted.bind(this));
    this.eventBus.subscribe('course.updated', this.handleCourseUpdated.bind(this));
  }

  setupRoutes(): void {
    const router = express.Router();

    // Educational API versioning
    this.app.use('/api/v1/courses', router);

    // Course management endpoints
    router.get('/', this.getAllCourses.bind(this));
    router.get('/:courseId', this.getCourse.bind(this));
    router.post('/', this.createCourse.bind(this));
    router.put('/:courseId', this.updateCourse.bind(this));
    router.delete('/:courseId', this.deleteCourse.bind(this));

    // Lesson management endpoints
    router.get('/:courseId/lessons', this.getCourseLessons.bind(this));
    router.post('/:courseId/lessons', this.createLesson.bind(this));
    router.put('/:courseId/lessons/:lessonId', this.updateLesson.bind(this));
    router.delete('/:courseId/lessons/:lessonId', this.deleteLesson.bind(this));

    // Educational enrollment endpoints
    router.post('/:courseId/enroll', this.enrollStudent.bind(this));
    router.delete('/:courseId/enroll/:studentId', this.unenrollStudent.bind(this));
    router.get('/:courseId/enrollments', this.getCourseEnrollments.bind(this));
  }

  private async getAllCourses(req: Request, res: Response): Promise<void> {
    try {
      const { page = 1, limit = 20, search, category, level } = req.query;
      const educationalContext = (req as any).educationalContext;

      const filters = {
        search: search as string,
        category: category as string,
        level: level as string,
        institutionId: educationalContext.institutionId
      };

      const result = await this.courseRepository.findAll({
        page: Number(page),
        limit: Number(limit),
        filters
      });

      res.json({
        courses: result.courses,
        pagination: {
          page: Number(page),
          limit: Number(limit),
          total: result.total,
          pages: Math.ceil(result.total / Number(limit))
        }
      });

    } catch (error) {
      this.logger.error('Error fetching courses', { error: error.message });
      res.status(500).json({ error: 'Failed to fetch courses' });
    }
  }

  private async createCourse(req: Request, res: Response): Promise<void> {
    try {
      const educationalContext = (req as any).educationalContext;
      
      // Validate educational permissions
      if (educationalContext.userRole !== 'instructor' && educationalContext.userRole !== 'admin') {
        return res.status(403).json({ error: 'Insufficient permissions to create course' });
      }

      const courseData = {
        ...req.body,
        institutionId: educationalContext.institutionId,
        instructorId: educationalContext.instructorId,
        createdAt: new Date(),
        updatedAt: new Date()
      };

      // Validate educational course data
      const validationResult = await validateCourseData(courseData);
      if (!validationResult.isValid) {
        return res.status(400).json({
          error: 'Educational course validation failed',
          details: validationResult.errors
        });
      }

      const course = await this.courseRepository.create(courseData);

      // Emit educational event
      await this.eventBus.publish('course.created', {
        courseId: course.id,
        instructorId: educationalContext.instructorId,
        institutionId: educationalContext.institutionId,
        timestamp: new Date()
      });

      res.status(201).json({ course });

    } catch (error) {
      this.logger.error('Error creating course', { error: error.message });
      res.status(500).json({ error: 'Failed to create course' });
    }
  }

  private async handleStudentEnrolled(event: StudentEnrolledEvent): Promise<void> {
    try {
      // Update course enrollment count
      await this.courseRepository.incrementEnrollmentCount(event.courseId);
      
      this.logger.info('Course enrollment count updated', {
        courseId: event.courseId,
        studentId: event.studentId
      });

    } catch (error) {
      this.logger.error('Error handling student enrolled event', { error: error.message });
    }
  }
}
```

### 3. Educational Database Patterns and Data Management
```typescript
// Educational repository pattern with advanced querying
interface EducationalRepository<T> {
  findById(id: string): Promise<T | null>;
  findByInstitution(institutionId: string): Promise<T[]>;
  findByEducationalCriteria(criteria: EducationalCriteria): Promise<T[]>;
  create(entity: Partial<T>): Promise<T>;
  update(id: string, updates: Partial<T>): Promise<T>;
  delete(id: string): Promise<void>;
  findWithEducationalRelations(id: string): Promise<T>;
}

class AdvancedCourseRepository implements EducationalRepository<Course> {
  constructor(private database: DatabaseConnection) {}

  async findById(courseId: string): Promise<Course | null> {
    const query = `
      SELECT 
        c.*,
        i.name as instructor_name,
        i.email as instructor_email,
        COUNT(e.student_id) as enrollment_count,
        AVG(r.rating) as average_rating
      FROM courses c
      LEFT JOIN instructors i ON c.instructor_id = i.id
      LEFT JOIN enrollments e ON c.id = e.course_id AND e.status = 'active'
      LEFT JOIN reviews r ON c.id = r.course_id
      WHERE c.id = $1 AND c.deleted_at IS NULL
      GROUP BY c.id, i.id
    `;

    const result = await this.database.query(query, [courseId]);
    return result.rows[0] ? this.mapToCourse(result.rows[0]) : null;
  }

  async findByEducationalCriteria(criteria: EducationalCriteria): Promise<Course[]> {
    let query = `
      SELECT 
        c.*,
        i.name as instructor_name,
        COUNT(e.student_id) as enrollment_count,
        AVG(r.rating) as average_rating,
        COUNT(l.id) as lesson_count
      FROM courses c
      LEFT JOIN instructors i ON c.instructor_id = i.id
      LEFT JOIN enrollments e ON c.id = e.course_id AND e.status = 'active'
      LEFT JOIN reviews r ON c.id = r.course_id
      LEFT JOIN lessons l ON c.id = l.course_id AND l.deleted_at IS NULL
      WHERE c.deleted_at IS NULL
    `;

    const params: any[] = [];
    let paramIndex = 1;

    // Educational filtering
    if (criteria.institutionId) {
      query += ` AND c.institution_id = $${paramIndex}`;
      params.push(criteria.institutionId);
      paramIndex++;
    }

    if (criteria.category) {
      query += ` AND c.category = $${paramIndex}`;
      params.push(criteria.category);
      paramIndex++;
    }

    if (criteria.level) {
      query += ` AND c.level = $${paramIndex}`;
      params.push(criteria.level);
      paramIndex++;
    }

    if (criteria.search) {
      query += ` AND (c.title ILIKE $${paramIndex} OR c.description ILIKE $${paramIndex})`;
      params.push(`%${criteria.search}%`);
      paramIndex++;
    }

    if (criteria.instructorId) {
      query += ` AND c.instructor_id = $${paramIndex}`;
      params.push(criteria.instructorId);
      paramIndex++;
    }

    // Educational content requirements
    if (criteria.hasVideoContent) {
      query += ` AND EXISTS (
        SELECT 1 FROM lessons l2 
        WHERE l2.course_id = c.id 
        AND l2.content_type = 'video'
        AND l2.deleted_at IS NULL
      )`;
    }

    if (criteria.hasAssessments) {
      query += ` AND EXISTS (
        SELECT 1 FROM assessments a 
        WHERE a.course_id = c.id 
        AND a.deleted_at IS NULL
      )`;
    }

    query += ` GROUP BY c.id, i.id`;

    // Educational sorting
    if (criteria.sortBy) {
      switch (criteria.sortBy) {
        case 'popularity':
          query += ` ORDER BY enrollment_count DESC, average_rating DESC`;
          break;
        case 'rating':
          query += ` ORDER BY average_rating DESC NULLS LAST`;
          break;
        case 'newest':
          query += ` ORDER BY c.created_at DESC`;
          break;
        case 'alphabetical':
          query += ` ORDER BY c.title ASC`;
          break;
        default:
          query += ` ORDER BY c.updated_at DESC`;
      }
    }

    // Pagination
    if (criteria.limit) {
      query += ` LIMIT $${paramIndex}`;
      params.push(criteria.limit);
      paramIndex++;
    }

    if (criteria.offset) {
      query += ` OFFSET $${paramIndex}`;
      params.push(criteria.offset);
      paramIndex++;
    }

    const result = await this.database.query(query, params);
    return result.rows.map(row => this.mapToCourse(row));
  }

  async createWithEducationalValidation(courseData: CreateCourseData): Promise<Course> {
    const transaction = await this.database.beginTransaction();

    try {
      // Educational business rule validation
      await this.validateEducationalRequirements(courseData);

      // Create course
      const courseQuery = `
        INSERT INTO courses (
          id, title, description, category, level, 
          instructor_id, institution_id, prerequisites,
          learning_objectives, estimated_duration,
          language, price, currency, status,
          created_at, updated_at
        ) VALUES (
          $1, $2, $3, $4, $5, $6, $7, $8, $9, $10, $11, $12, $13, $14, $15, $16
        ) RETURNING *
      `;

      const courseId = generateId();
      const now = new Date();

      const courseResult = await transaction.query(courseQuery, [
        courseId,
        courseData.title,
        courseData.description,
        courseData.category,
        courseData.level,
        courseData.instructorId,
        courseData.institutionId,
        JSON.stringify(courseData.prerequisites || []),
        JSON.stringify(courseData.learningObjectives || []),
        courseData.estimatedDuration,
        courseData.language || 'en',
        courseData.price || 0,
        courseData.currency || 'USD',
        'draft',
        now,
        now
      ]);

      // Create educational metadata
      if (courseData.educationalMetadata) {
        await this.createEducationalMetadata(transaction, courseId, courseData.educationalMetadata);
      }

      // Create learning standards mapping if provided
      if (courseData.learningStandards) {
        await this.mapLearningStandards(transaction, courseId, courseData.learningStandards);
      }

      await transaction.commit();

      return this.mapToCourse(courseResult.rows[0]);

    } catch (error) {
      await transaction.rollback();
      throw error;
    }
  }

  private async validateEducationalRequirements(courseData: CreateCourseData): Promise<void> {
    // Validate instructor credentials
    const instructor = await this.database.query(
      'SELECT * FROM instructors WHERE id = $1 AND status = $2',
      [courseData.instructorId, 'active']
    );

    if (instructor.rows.length === 0) {
      throw new ValidationError('Invalid or inactive instructor');
    }

    // Validate institutional compliance
    const institution = await this.database.query(
      'SELECT * FROM institutions WHERE id = $1 AND status = $2',
      [courseData.institutionId, 'active']
    );

    if (institution.rows.length === 0) {
      throw new ValidationError('Invalid or inactive institution');
    }

    // Validate educational content requirements
    if (courseData.level === 'advanced' && !courseData.prerequisites?.length) {
      throw new ValidationError('Advanced courses must have prerequisites');
    }

    // Validate learning objectives
    if (!courseData.learningObjectives?.length) {
      throw new ValidationError('Courses must have defined learning objectives');
    }
  }

  private mapToCourse(row: any): Course {
    return {
      id: row.id,
      title: row.title,
      description: row.description,
      category: row.category,
      level: row.level,
      instructorId: row.instructor_id,
      instructorName: row.instructor_name,
      institutionId: row.institution_id,
      prerequisites: JSON.parse(row.prerequisites || '[]'),
      learningObjectives: JSON.parse(row.learning_objectives || '[]'),
      estimatedDuration: row.estimated_duration,
      language: row.language,
      price: row.price,
      currency: row.currency,
      status: row.status,
      enrollmentCount: parseInt(row.enrollment_count || '0'),
      averageRating: parseFloat(row.average_rating || '0'),
      lessonCount: parseInt(row.lesson_count || '0'),
      createdAt: row.created_at,
      updatedAt: row.updated_at
    };
  }
}
```

### 4. Advanced Educational API Gateway Pattern
```typescript
// Educational API Gateway with advanced routing and middleware
class EducationalApiGateway {
  private app: Express;
  private serviceRegistry: ServiceRegistry;
  private loadBalancer: LoadBalancer;
  private cache: CacheManager;

  constructor() {
    this.app = express();
    this.serviceRegistry = new ServiceRegistry();
    this.loadBalancer = new LoadBalancer();
    this.cache = new CacheManager();
    this.setupMiddleware();
    this.setupRoutes();
  }

  private setupMiddleware(): void {
    // Educational authentication middleware
    this.app.use('/api', this.educationalAuthMiddleware.bind(this));
    
    // Educational authorization middleware
    this.app.use('/api', this.educationalAuthorizationMiddleware.bind(this));
    
    // Educational request enrichment
    this.app.use('/api', this.enrichEducationalContext.bind(this));
    
    // Educational caching strategy
    this.app.use('/api', this.educationalCacheMiddleware.bind(this));
  }

  private async educationalAuthMiddleware(req: Request, res: Response, next: NextFunction): Promise<void> {
    try {
      const token = req.headers.authorization?.replace('Bearer ', '');
      
      if (!token) {
        return res.status(401).json({ error: 'Educational authentication required' });
      }

      // Verify educational JWT token
      const decodedToken = await verifyEducationalToken(token);
      
      // Extract educational claims
      const educationalClaims = {
        userId: decodedToken.sub,
        userRole: decodedToken.role,
        institutionId: decodedToken.institutionId,
        permissions: decodedToken.permissions || [],
        studentId: decodedToken.studentId,
        instructorId: decodedToken.instructorId,
        sessionId: decodedToken.sessionId
      };

      // Validate educational session
      const sessionValid = await this.validateEducationalSession(educationalClaims);
      if (!sessionValid) {
        return res.status(401).json({ error: 'Educational session expired' });
      }

      // Attach to request
      (req as any).educationalClaims = educationalClaims;
      next();

    } catch (error) {
      res.status(401).json({ error: 'Educational authentication failed' });
    }
  }

  private async educationalAuthorizationMiddleware(req: Request, res: Response, next: NextFunction): Promise<void> {
    const educationalClaims = (req as any).educationalClaims;
    const requestedPath = req.path;
    const requestedMethod = req.method;

    // Educational permission checking
    const requiredPermission = this.mapRouteToPermission(requestedPath, requestedMethod);
    
    if (requiredPermission) {
      const hasPermission = await this.checkEducationalPermission(
        educationalClaims,
        requiredPermission,
        req.params
      );

      if (!hasPermission) {
        return res.status(403).json({ 
          error: 'Educational access denied',
          requiredPermission,
          userRole: educationalClaims.userRole
        });
      }
    }

    next();
  }

  private async enrichEducationalContext(req: Request, res: Response, next: NextFunction): Promise<void> {
    const educationalClaims = (req as any).educationalClaims;

    // Add educational headers for downstream services
    req.headers['x-user-id'] = educationalClaims.userId;
    req.headers['x-user-role'] = educationalClaims.userRole;
    req.headers['x-institution-id'] = educationalClaims.institutionId;
    req.headers['x-student-id'] = educationalClaims.studentId;
    req.headers['x-instructor-id'] = educationalClaims.instructorId;
    req.headers['x-session-id'] = educationalClaims.sessionId;

    next();
  }

  private setupRoutes(): void {
    // Educational service routing
    this.app.use('/api/v1/courses', this.proxyToService('course-management'));
    this.app.use('/api/v1/enrollments', this.proxyToService('enrollment-service'));
    this.app.use('/api/v1/progress', this.proxyToService('progress-service'));
    this.app.use('/api/v1/assessments', this.proxyToService('assessment-service'));
    this.app.use('/api/v1/users', this.proxyToService('user-service'));
    this.app.use('/api/v1/analytics', this.proxyToService('analytics-service'));
    this.app.use('/api/v1/notifications', this.proxyToService('notification-service'));
  }

  private proxyToService(serviceName: string) {
    return async (req: Request, res: Response) => {
      try {
        // Get healthy service instance
        const serviceInstance = await this.loadBalancer.getHealthyInstance(serviceName);
        
        if (!serviceInstance) {
          return res.status(503).json({ 
            error: 'Educational service unavailable',
            service: serviceName
          });
        }

        // Educational request proxying with circuit breaker
        const response = await this.makeEducationalRequest(serviceInstance, req);
        
        // Educational response transformation
        const transformedResponse = this.transformEducationalResponse(response, req);
        
        res.status(response.status).json(transformedResponse);

      } catch (error) {
        this.handleEducationalProxyError(error, serviceName, res);
      }
    };
  }

  private async makeEducationalRequest(serviceInstance: ServiceInstance, req: Request): Promise<any> {
    const circuitBreaker = this.getCircuitBreaker(serviceInstance.name);
    
    return circuitBreaker.execute(async () => {
      const response = await axios({
        method: req.method,
        url: `${serviceInstance.url}${req.path}`,
        data: req.body,
        headers: {
          ...req.headers,
          'host': undefined, // Remove gateway host
          'x-forwarded-for': req.ip,
          'x-gateway-timestamp': new Date().toISOString()
        },
        timeout: 30000
      });

      return response.data;
    });
  }
}
```

## DO's and DON'Ts for Advanced Educational Backend Architecture

### ✅ DO's:
- Implement domain-driven design with educational bounded contexts
- Use microservices architecture for scalable educational platforms
- Create educational-specific middleware for authentication, authorization, and context enrichment
- Implement comprehensive error handling with educational context
- Use event-driven architecture for loose coupling between educational services
- Implement educational data validation and business rule enforcement
- Use repository patterns for educational data access with advanced querying
- Implement educational caching strategies for performance optimization
- Create educational API gateways with proper routing and load balancing
- Use educational-specific rate limiting and security measures

### ❌ DON'Ts:
- Don't create monolithic applications for complex educational platforms
- Don't ignore educational data privacy and compliance requirements (FERPA, GDPR)
- Don't skip educational business rule validation in domain models
- Don't use synchronous communication between all educational microservices
- Don't ignore educational performance requirements for content-heavy operations
- Don't skip educational audit logging and analytics tracking
- Don't implement educational features without proper error boundaries
- Don't ignore educational accessibility requirements in API design
- Don't skip educational data backup and disaster recovery planning
- Don't use weak authentication and authorization for educational platforms

This comprehensive guide provides advanced backend architecture patterns specifically designed for educational platforms, incorporating microservices best practices, domain-driven design, and educational-specific requirements for building scalable, secure, and compliant Learning Management Systems. 