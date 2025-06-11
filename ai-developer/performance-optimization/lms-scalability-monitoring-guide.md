# LMS Performance Optimization & Scalability Monitoring Guide

## Table of Contents
1. [LMS Performance Overview](#lms-performance-overview)
2. [Frontend Performance Optimization](#frontend-performance-optimization)
3. [Backend API Performance](#backend-api-performance)
4. [Database Query Optimization](#database-query-optimization)
5. [Real-Time Monitoring Systems](#real-time-monitoring-systems)
6. [Scalability Patterns for Educational Platforms](#scalability-patterns-for-educational-platforms)
7. [Performance Testing & Benchmarking](#performance-testing--benchmarking)
8. [Incident Response & Performance Recovery](#incident-response--performance-recovery)

## LMS Performance Overview

### 1. Educational Platform Performance Requirements

**LMS-Specific Performance Targets:**
```javascript
// ✅ Good: Comprehensive performance metrics for educational systems
const LMS_PERFORMANCE_TARGETS = {
  // Core Learning Experience
  COURSE_LOADING: {
    target: '< 2 seconds',
    critical: '< 5 seconds',
    measurement: 'Time to interactive course content'
  },
  
  LESSON_PROGRESSION: {
    target: '< 1 second',
    critical: '< 3 seconds',
    measurement: 'Lesson navigation response time'
  },
  
  ASSESSMENT_SUBMISSION: {
    target: '< 1.5 seconds',
    critical: '< 4 seconds',
    measurement: 'Quiz answer submission to confirmation'
  },
  
  PROGRESS_TRACKING: {
    target: '< 500ms',
    critical: '< 2 seconds',
    measurement: 'Progress update and sync time'
  },
  
  // Content Generation (AI-Powered)
  COURSE_GENERATION: {
    target: '< 30 seconds',
    critical: '< 60 seconds',
    measurement: 'Complete course creation with AI'
  },
  
  REAL_TIME_FEATURES: {
    target: '< 100ms',
    critical: '< 500ms',
    measurement: 'Live collaboration and chat latency'
  },
  
  // System Scalability
  CONCURRENT_USERS: {
    target: '1000+ simultaneous learners',
    critical: '500+ without degradation',
    measurement: 'Active learning sessions'
  },
  
  DATABASE_QUERIES: {
    target: '< 100ms (95th percentile)',
    critical: '< 500ms (99th percentile)',
    measurement: 'Educational data retrieval'
  }
};

// Performance monitoring class for LMS
class LMSPerformanceTracker {
  constructor() {
    this.metrics = new Map();
    this.thresholds = LMS_PERFORMANCE_TARGETS;
    this.alerts = [];
  }

  trackMetric(metricName, value, context = {}) {
    const timestamp = Date.now();
    const metric = {
      name: metricName,
      value,
      timestamp,
      context,
      threshold: this.thresholds[metricName]
    };

    // Store metric
    if (!this.metrics.has(metricName)) {
      this.metrics.set(metricName, []);
    }
    this.metrics.get(metricName).push(metric);

    // Check thresholds
    this.evaluateThreshold(metric);

    return metric;
  }

  evaluateThreshold(metric) {
    if (!metric.threshold) return;

    const targetMs = this.parseTimeString(metric.threshold.target);
    const criticalMs = this.parseTimeString(metric.threshold.critical);

    if (metric.value > criticalMs) {
      this.triggerAlert('CRITICAL', metric);
    } else if (metric.value > targetMs) {
      this.triggerAlert('WARNING', metric);
    }
  }

  generatePerformanceReport(timeRange = 3600000) { // 1 hour default
    const now = Date.now();
    const report = {};

    for (const [metricName, values] of this.metrics) {
      const recentValues = values.filter(v => now - v.timestamp < timeRange);
      
      if (recentValues.length > 0) {
        const sorted = recentValues.map(v => v.value).sort((a, b) => a - b);
        
        report[metricName] = {
          count: recentValues.length,
          average: sorted.reduce((sum, val) => sum + val, 0) / sorted.length,
          median: sorted[Math.floor(sorted.length / 2)],
          p95: sorted[Math.floor(sorted.length * 0.95)],
          p99: sorted[Math.floor(sorted.length * 0.99)],
          min: sorted[0],
          max: sorted[sorted.length - 1],
          threshold: this.thresholds[metricName]
        };
      }
    }

    return report;
  }
}
```

### 2. Performance Monitoring Architecture

**✅ Good: Comprehensive Monitoring Setup**
```javascript
// monitoring/performance-architecture.js
class LMSMonitoringSystem {
  constructor() {
    this.monitors = {
      frontend: new FrontendPerformanceMonitor(),
      backend: new BackendPerformanceMonitor(),
      database: new DatabasePerformanceMonitor(),
      infrastructure: new InfrastructureMonitor()
    };
    
    this.alerting = new AlertingSystem();
    this.dashboard = new PerformanceDashboard();
  }

  async initializeMonitoring() {
    // Start all monitoring systems
    await Promise.all([
      this.monitors.frontend.start(),
      this.monitors.backend.start(),
      this.monitors.database.start(),
      this.monitors.infrastructure.start()
    ]);

    // Setup alert routing
    this.setupAlertRouting();
    
    // Initialize dashboard
    await this.dashboard.initialize();
  }

  setupAlertRouting() {
    // Route performance alerts based on severity
    this.alerting.on('CRITICAL', (alert) => {
      this.notifyOncallTeam(alert);
      this.autoScaleIfNeeded(alert);
    });

    this.alerting.on('WARNING', (alert) => {
      this.logPerformanceIssue(alert);
      this.suggestOptimizations(alert);
    });
  }

  async notifyOncallTeam(alert) {
    const notification = {
      severity: 'CRITICAL',
      service: 'LMS-Platform',
      metric: alert.metric,
      value: alert.value,
      threshold: alert.threshold,
      timestamp: new Date(),
      runbook: this.getRunbookURL(alert.metric)
    };

    // Send to multiple channels
    await Promise.all([
      this.sendSlackAlert(notification),
      this.sendPagerDutyAlert(notification),
      this.sendEmailAlert(notification)
    ]);
  }

  async autoScaleIfNeeded(alert) {
    // Auto-scaling logic for different performance issues
    switch (alert.metric) {
      case 'CONCURRENT_USERS':
        if (alert.value > 800) {
          await this.scaleWebServers();
        }
        break;
        
      case 'DATABASE_QUERIES':
        if (alert.value > 1000) {
          await this.enableDatabaseReadReplicas();
        }
        break;
        
      case 'COURSE_GENERATION':
        if (alert.value > 45000) {
          await this.scaleAIProcessingWorkers();
        }
        break;
    }
  }
}
```

## Frontend Performance Optimization

### 1. React Performance Optimization for LMS

**✅ Good: LMS-Specific React Optimizations**
```javascript
// components/optimized/CourseRenderer.tsx
import React, { memo, useMemo, useCallback, lazy, Suspense } from 'react';
import { useVirtualizer } from '@tanstack/react-virtual';
import { useInView } from 'react-intersection-observer';

// ✅ Good: Lazy load heavy components
const VideoPlayer = lazy(() => import('./VideoPlayer'));
const InteractiveQuiz = lazy(() => import('./InteractiveQuiz'));
const CodeEditor = lazy(() => import('./CodeEditor'));

// ✅ Good: Memoized course content renderer
const CourseContentRenderer = memo(({ lesson, isActive, onProgress }) => {
  // Memoize expensive computations
  const processedContent = useMemo(() => {
    return lesson.content.map(block => {
      if (block.type === 'code') {
        return {
          ...block,
          processedCode: highlightSyntax(block.code),
          estimatedReadTime: calculateReadTime(block.code)
        };
      }
      return block;
    });
  }, [lesson.content]);

  // Memoize event handlers
  const handleProgress = useCallback((progress) => {
    onProgress(lesson.id, progress);
  }, [lesson.id, onProgress]);

  // Use intersection observer for lazy loading
  const { ref, inView } = useInView({
    threshold: 0.1,
    triggerOnce: true
  });

  return (
    <div ref={ref} className="lesson-content">
      {inView && (
        <>
          {processedContent.map((block, index) => (
            <ContentBlock
              key={`${lesson.id}-${index}`}
              block={block}
              onProgress={handleProgress}
            />
          ))}
        </>
      )}
    </div>
  );
});

// ✅ Good: Virtualized lesson list for large courses
const VirtualizedLessonList = ({ lessons, currentLessonId, onLessonSelect }) => {
  const parentRef = useRef();

  const virtualizer = useVirtualizer({
    count: lessons.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 80, // Estimated lesson item height
    overscan: 5 // Render 5 extra items for smooth scrolling
  });

  return (
    <div
      ref={parentRef}
      className="lesson-list-container"
      style={{ height: '400px', overflow: 'auto' }}
    >
      <div
        style={{
          height: `${virtualizer.getTotalSize()}px`,
          width: '100%',
          position: 'relative'
        }}
      >
        {virtualizer.getVirtualItems().map((virtualItem) => {
          const lesson = lessons[virtualItem.index];
          return (
            <div
              key={virtualItem.key}
              style={{
                position: 'absolute',
                top: 0,
                left: 0,
                width: '100%',
                height: `${virtualItem.size}px`,
                transform: `translateY(${virtualItem.start}px)`
              }}
            >
              <LessonListItem
                lesson={lesson}
                isActive={lesson.id === currentLessonId}
                onSelect={() => onLessonSelect(lesson.id)}
              />
            </div>
          );
        })}
      </div>
    </div>
  );
};

// ✅ Good: Optimized content block with lazy loading
const ContentBlock = memo(({ block, onProgress }) => {
  const renderContent = () => {
    switch (block.type) {
      case 'video':
        return (
          <Suspense fallback={<VideoPlayerSkeleton />}>
            <VideoPlayer
              src={block.videoUrl}
              onProgress={onProgress}
              preload="metadata" // Optimize video loading
            />
          </Suspense>
        );
        
      case 'quiz':
        return (
          <Suspense fallback={<QuizSkeleton />}>
            <InteractiveQuiz
              questions={block.questions}
              onComplete={onProgress}
            />
          </Suspense>
        );
        
      case 'code':
        return (
          <Suspense fallback={<CodeEditorSkeleton />}>
            <CodeEditor
              initialCode={block.code}
              language={block.language}
              readOnly={block.readOnly}
            />
          </Suspense>
        );
        
      default:
        return <TextContent content={block.content} />;
    }
  };

  return (
    <div className={`content-block content-block--${block.type}`}>
      {renderContent()}
    </div>
  );
});

// ✅ Good: Performance-optimized progress tracking
const useOptimizedProgressTracking = (courseId, userId) => {
  const [progress, setProgress] = useState({});
  const progressRef = useRef({});
  const updateQueueRef = useRef([]);

  // Debounced progress updates to reduce API calls
  const debouncedUpdateProgress = useCallback(
    debounce(async (updates) => {
      try {
        await api.updateProgress(courseId, userId, updates);
        // Clear successful updates from queue
        updateQueueRef.current = [];
      } catch (error) {
        console.error('Progress update failed:', error);
        // Keep failed updates in queue for retry
      }
    }, 1000),
    [courseId, userId]
  );

  const updateProgress = useCallback((lessonId, progressValue) => {
    // Update local state immediately for responsive UI
    setProgress(prev => ({
      ...prev,
      [lessonId]: progressValue
    }));

    // Queue for batch API update
    progressRef.current[lessonId] = progressValue;
    updateQueueRef.current.push({ lessonId, progress: progressValue });
    
    // Trigger debounced update
    debouncedUpdateProgress(progressRef.current);
  }, [debouncedUpdateProgress]);

  return { progress, updateProgress };
};
```

### 2. Bundle Optimization and Code Splitting

**✅ Good: Advanced Code Splitting for LMS**
```javascript
// webpack.config.js - LMS-optimized Webpack configuration
const path = require('path');
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');
const CompressionPlugin = require('compression-webpack-plugin');

module.exports = {
  entry: {
    main: './src/index.js',
    // Separate bundles for different user types
    student: './src/student/index.js',
    instructor: './src/instructor/index.js',
    admin: './src/admin/index.js'
  },
  
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        // Vendor libraries
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
        
        // Common LMS components
        common: {
          name: 'common',
          minChunks: 2,
          chunks: 'all',
          enforce: true
        },
        
        // Heavy libraries used conditionally
        videoPlayer: {
          test: /[\\/]node_modules[\\/](video\.js|hls\.js|dash\.js)/,
          name: 'video-player',
          chunks: 'async'
        },
        
        codeEditor: {
          test: /[\\/]node_modules[\\/](monaco-editor|codemirror)/,
          name: 'code-editor',
          chunks: 'async'
        },
        
        aiTools: {
          test: /[\\/]src[\\/](ai|generation)[\\/]/,
          name: 'ai-tools',
          chunks: 'async'
        }
      }
    }
  },
  
  plugins: [
    // Gzip compression for production
    new CompressionPlugin({
      algorithm: 'gzip',
      test: /\.(js|css|html|svg)$/,
      threshold: 8192,
      minRatio: 0.8
    }),
    
    // Bundle analysis for optimization
    process.env.ANALYZE && new BundleAnalyzerPlugin()
  ].filter(Boolean)
};

// Route-based code splitting
const AppRouter = () => {
  return (
    <Router>
      <Routes>
        {/* Lazy load major sections */}
        <Route path="/courses" element={
          <Suspense fallback={<CoursesSkeleton />}>
            <Courses />
          </Suspense>
        } />
        
        <Route path="/course/:id" element={
          <Suspense fallback={<CourseSkeleton />}>
            <CourseView />
          </Suspense>
        } />
        
        <Route path="/instructor/*" element={
          <Suspense fallback={<InstructorSkeleton />}>
            <InstructorDashboard />
          </Suspense>
        } />
        
        <Route path="/admin/*" element={
          <Suspense fallback={<AdminSkeleton />}>
            <AdminPanel />
          </Suspense>
        } />
      </Routes>
    </Router>
  );
};

// ✅ Good: Dynamic imports for features
const useDynamicFeatures = () => {
  const loadVideoPlayer = useCallback(async () => {
    const { VideoPlayer } = await import('../components/VideoPlayer');
    return VideoPlayer;
  }, []);

  const loadCodeEditor = useCallback(async () => {
    const { CodeEditor } = await import('../components/CodeEditor');
    return CodeEditor;
  }, []);

  const loadAITools = useCallback(async () => {
    const { CourseGenerator } = await import('../ai/CourseGenerator');
    return CourseGenerator;
  }, []);

  return { loadVideoPlayer, loadCodeEditor, loadAITools };
};
```

## Backend API Performance

### 1. Node.js Performance Optimization

**✅ Good: Express.js Performance for Educational APIs**
```javascript
// server/performance/optimization.js
const express = require('express');
const compression = require('compression');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const NodeCache = require('node-cache');
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

// ✅ Good: LMS-optimized Express configuration
class LMSServer {
  constructor() {
    this.app = express();
    this.cache = new NodeCache({ stdTTL: 600 }); // 10 minute default
    this.setupMiddleware();
    this.setupRoutes();
  }

  setupMiddleware() {
    // Compression for all responses
    this.app.use(compression({
      level: 6, // Good balance of compression vs CPU
      threshold: 1024, // Only compress files > 1KB
      filter: (req, res) => {
        if (req.headers['x-no-compression']) return false;
        return compression.filter(req, res);
      }
    }));

    // Security headers
    this.app.use(helmet({
      contentSecurityPolicy: {
        directives: {
          defaultSrc: ["'self'"],
          imgSrc: ["'self'", "data:", "https:"],
          scriptSrc: ["'self'", "'unsafe-inline'"], // For educational content
          styleSrc: ["'self'", "'unsafe-inline'"]
        }
      }
    }));

    // Rate limiting with educational-specific rules
    this.app.use('/api/courses', rateLimit({
      windowMs: 15 * 60 * 1000, // 15 minutes
      max: 1000, // Generous limit for learning activities
      message: 'Too many requests, please try again later',
      standardHeaders: true,
      legacyHeaders: false
    }));

    // Stricter rate limiting for resource-intensive operations
    this.app.use('/api/courses/generate', rateLimit({
      windowMs: 60 * 60 * 1000, // 1 hour
      max: 5, // 5 course generations per hour
      message: 'Course generation limit reached'
    }));

    // Response caching middleware
    this.app.use(this.cacheMiddleware.bind(this));
  }

  cacheMiddleware(req, res, next) {
    // Cache strategy based on route
    if (req.method !== 'GET') return next();

    const cacheKey = this.generateCacheKey(req);
    const cached = this.cache.get(cacheKey);

    if (cached) {
      res.set('X-Cache', 'HIT');
      return res.json(cached);
    }

    // Override res.json to cache response
    const originalJson = res.json;
    res.json = (data) => {
      const ttl = this.getCacheTTL(req.path);
      if (ttl > 0) {
        this.cache.set(cacheKey, data, ttl);
      }
      res.set('X-Cache', 'MISS');
      return originalJson.call(res, data);
    };

    next();
  }

  generateCacheKey(req) {
    const userId = req.user?.id || 'anonymous';
    const query = JSON.stringify(req.query);
    return `${req.path}:${userId}:${query}`;
  }

  getCacheTTL(path) {
    // Different cache durations for different content types
    const cacheRules = {
      '/api/courses': 300,        // 5 minutes - course list
      '/api/courses/': 600,       // 10 minutes - specific course
      '/api/progress/': 60,       // 1 minute - progress data
      '/api/assessments/': 1800,  // 30 minutes - assessment content
      '/api/users/profile': 300   // 5 minutes - user profile
    };

    for (const [route, ttl] of Object.entries(cacheRules)) {
      if (path.startsWith(route)) return ttl;
    }
    
    return 0; // No cache by default
  }

  setupRoutes() {
    // Optimized course routes
    this.app.get('/api/courses', this.getCourses.bind(this));
    this.app.get('/api/courses/:id', this.getCourse.bind(this));
    this.app.post('/api/courses/generate', this.generateCourse.bind(this));
    
    // Optimized progress routes
    this.app.get('/api/progress/:userId/:courseId', this.getProgress.bind(this));
    this.app.post('/api/progress/update', this.updateProgress.bind(this));
  }

  async getCourses(req, res) {
    try {
      const startTime = Date.now();
      
      // Optimized query with pagination
      const { page = 1, limit = 20, category, difficulty } = req.query;
      const offset = (page - 1) * limit;

      const courses = await this.courseService.findMany({
        where: {
          ...(category && { category }),
          ...(difficulty && { difficulty }),
          published: true
        },
        include: {
          instructor: { select: { id: true, name: true } },
          _count: { select: { enrollments: true } }
        },
        take: parseInt(limit),
        skip: offset,
        orderBy: { createdAt: 'desc' }
      });

      const responseTime = Date.now() - startTime;
      
      // Log performance metrics
      this.logMetric('GET /api/courses', responseTime, {
        courseCount: courses.length,
        filters: { category, difficulty },
        pagination: { page, limit }
      });

      res.json({
        courses,
        pagination: {
          page: parseInt(page),
          limit: parseInt(limit),
          total: await this.courseService.count()
        },
        meta: {
          responseTime,
          cached: false
        }
      });
    } catch (error) {
      this.handleError(error, req, res);
    }
  }

  async generateCourse(req, res) {
    try {
      const startTime = Date.now();
      const { title, difficulty, topics } = req.body;

      // Validate input
      if (!title || !difficulty || !topics) {
        return res.status(400).json({ error: 'Missing required fields' });
      }

      // Start course generation asynchronously
      const jobId = this.generateUniqueId();
      
      // Return immediately with job ID
      res.status(202).json({
        jobId,
        status: 'processing',
        estimatedTime: 30000, // 30 seconds
        statusUrl: `/api/courses/generate/status/${jobId}`
      });

      // Process in background
      this.processCourseGeneration(jobId, { title, difficulty, topics })
        .catch(error => {
          console.error('Course generation failed:', error);
          this.updateJobStatus(jobId, 'failed', error.message);
        });

    } catch (error) {
      this.handleError(error, req, res);
    }
  }

  logMetric(operation, duration, metadata = {}) {
    const metric = {
      operation,
      duration,
      timestamp: new Date(),
      metadata
    };

    // Send to monitoring system
    this.performanceTracker.trackMetric(operation.replace(/\s+/g, '_'), duration, metadata);
    
    // Log slow operations
    if (duration > 2000) {
      console.warn('Slow operation detected:', metric);
    }
  }
}

// ✅ Good: Cluster setup for production
if (cluster.isMaster) {
  console.log(`Master ${process.pid} is running`);

  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork(); // Restart worker
  });
} else {
  const server = new LMSServer();
  const PORT = process.env.PORT || 3000;
  
  server.app.listen(PORT, () => {
    console.log(`Worker ${process.pid} started on port ${PORT}`);
  });
}
```

### 2. Database Query Optimization

**✅ Good: Educational Data Query Optimization**
```javascript
// database/optimizations/queries.js
class LMSQueryOptimizer {
  constructor(prisma) {
    this.prisma = prisma;
    this.queryCache = new Map();
  }

  // ✅ Good: Optimized course loading with selective includes
  async getCourseWithProgress(courseId, userId) {
    const cacheKey = `course:${courseId}:user:${userId}`;
    
    if (this.queryCache.has(cacheKey)) {
      return this.queryCache.get(cacheKey);
    }

    const result = await this.prisma.course.findUnique({
      where: { id: courseId },
      include: {
        lessons: {
          select: {
            id: true,
            title: true,
            duration: true,
            orderIndex: true,
            // Only load progress for current user
            progress: {
              where: { userId },
              select: {
                completed: true,
                completionDate: true,
                timeSpent: true
              }
            }
          },
          orderBy: { orderIndex: 'asc' }
        },
        assessments: {
          select: {
            id: true,
            title: true,
            passingScore: true,
            // Only load attempts for current user
            attempts: {
              where: { userId },
              select: {
                score: true,
                completedAt: true,
                passed: true
              },
              orderBy: { completedAt: 'desc' },
              take: 1 // Only latest attempt
            }
          }
        },
        instructor: {
          select: {
            id: true,
            name: true,
            avatar: true
          }
        },
        // Aggregate counts without loading all records
        _count: {
          select: {
            enrollments: true,
            lessons: true,
            assessments: true
          }
        }
      }
    });

    // Cache for 5 minutes
    this.queryCache.set(cacheKey, result);
    setTimeout(() => this.queryCache.delete(cacheKey), 300000);

    return result;
  }

  // ✅ Good: Batch progress updates for performance
  async batchUpdateProgress(updates) {
    const transaction = await this.prisma.$transaction(async (tx) => {
      const results = [];

      // Group updates by user and course for efficiency
      const groupedUpdates = this.groupProgressUpdates(updates);

      for (const [key, userUpdates] of groupedUpdates) {
        const [userId, courseId] = key.split(':');

        // Upsert progress records in batch
        const progressUpserts = userUpdates.map(update => 
          tx.lessonProgress.upsert({
            where: {
              userId_lessonId: {
                userId,
                lessonId: update.lessonId
              }
            },
            update: {
              progress: update.progress,
              timeSpent: { increment: update.timeSpent || 0 },
              lastAccessed: new Date(),
              completed: update.progress >= 1.0
            },
            create: {
              userId,
              lessonId: update.lessonId,
              progress: update.progress,
              timeSpent: update.timeSpent || 0,
              lastAccessed: new Date(),
              completed: update.progress >= 1.0
            }
          })
        );

        const batchResults = await Promise.all(progressUpserts);
        results.push(...batchResults);

        // Update course-level progress
        await this.updateCourseProgress(tx, userId, courseId);
      }

      return results;
    });

    return transaction;
  }

  // ✅ Good: Efficient analytics queries
  async getCourseAnalytics(courseId, timeRange = '30d') {
    const dateFilter = this.getDateFilter(timeRange);

    // Use raw SQL for complex analytics to avoid ORM overhead
    const analytics = await this.prisma.$queryRaw`
      SELECT 
        DATE(created_at) as date,
        COUNT(DISTINCT user_id) as active_users,
        AVG(session_duration) as avg_session_duration,
        SUM(CASE WHEN completed = true THEN 1 ELSE 0 END) as completions,
        COUNT(*) as total_sessions
      FROM user_sessions 
      WHERE course_id = ${courseId}
        AND created_at >= ${dateFilter}
      GROUP BY DATE(created_at)
      ORDER BY date DESC
    `;

    // Get progress distribution
    const progressDistribution = await this.prisma.$queryRaw`
      SELECT 
        FLOOR(overall_progress * 10) * 10 as progress_range,
        COUNT(*) as user_count
      FROM course_progress 
      WHERE course_id = ${courseId}
      GROUP BY FLOOR(overall_progress * 10)
      ORDER BY progress_range
    `;

    return {
      dailyActivity: analytics,
      progressDistribution,
      cacheTimestamp: new Date()
    };
  }

  // ✅ Good: Optimized search with full-text search
  async searchCourses(query, filters = {}) {
    const { category, difficulty, minRating, maxPrice } = filters;

    // Use full-text search for better performance
    const courses = await this.prisma.$queryRaw`
      SELECT 
        c.*,
        ts_rank_cd(search_vector, plainto_tsquery(${query})) as relevance,
        i.name as instructor_name,
        COUNT(e.id) as enrollment_count,
        AVG(r.rating) as average_rating
      FROM courses c
      LEFT JOIN instructors i ON c.instructor_id = i.id
      LEFT JOIN enrollments e ON c.id = e.course_id
      LEFT JOIN reviews r ON c.id = r.course_id
      WHERE c.search_vector @@ plainto_tsquery(${query})
        AND c.published = true
        ${category ? `AND c.category = ${category}` : ''}
        ${difficulty ? `AND c.difficulty = ${difficulty}` : ''}
      GROUP BY c.id, i.name
      HAVING ${maxPrice ? `c.price <= ${maxPrice}` : 'true'}
        AND ${minRating ? `AVG(r.rating) >= ${minRating}` : 'true'}
      ORDER BY relevance DESC, average_rating DESC
      LIMIT 50
    `;

    return courses;
  }

  groupProgressUpdates(updates) {
    const grouped = new Map();
    
    updates.forEach(update => {
      const key = `${update.userId}:${update.courseId}`;
      if (!grouped.has(key)) {
        grouped.set(key, []);
      }
      grouped.get(key).push(update);
    });

    return grouped;
  }

  async updateCourseProgress(tx, userId, courseId) {
    // Calculate overall course progress efficiently
    const progressSummary = await tx.lessonProgress.aggregate({
      where: {
        userId,
        lesson: { courseId }
      },
      _avg: { progress: true },
      _count: { completed: true }
    });

    const totalLessons = await tx.lesson.count({
      where: { courseId }
    });

    const overallProgress = progressSummary._avg.progress || 0;
    const completedLessons = progressSummary._count.completed || 0;

    await tx.courseProgress.upsert({
      where: {
        userId_courseId: { userId, courseId }
      },
      update: {
        overallProgress,
        completedLessons,
        totalLessons,
        lastAccessed: new Date(),
        completed: completedLessons === totalLessons
      },
      create: {
        userId,
        courseId,
        overallProgress,
        completedLessons,
        totalLessons,
        lastAccessed: new Date(),
        completed: completedLessons === totalLessons
      }
    });
  }

  getDateFilter(timeRange) {
    const now = new Date();
    const ranges = {
      '7d': 7,
      '30d': 30,
      '90d': 90,
      '1y': 365
    };

    const days = ranges[timeRange] || 30;
    return new Date(now.getTime() - days * 24 * 60 * 60 * 1000);
  }
}
```

This performance optimization guide provides comprehensive strategies for scaling LMS platforms, with specific focus on educational workflows, real-time learning analytics, and user experience optimization. The examples demonstrate production-ready patterns for handling large-scale educational technology systems. 