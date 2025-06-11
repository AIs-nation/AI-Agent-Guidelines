# Performance Optimization and Caching Strategies for LMS Systems

## Multi-Layer Caching Architecture

### Strategic Caching Hierarchy
**Implement multi-level caching for optimal LMS performance**

✅ **Good Example: LMS Caching Strategy**
```javascript
// cacheManager.js - Comprehensive caching solution
class LMSCacheManager {
  constructor() {
    this.redis = new Redis({
      host: process.env.REDIS_HOST,
      port: process.env.REDIS_PORT,
      password: process.env.REDIS_PASSWORD,
      retryDelayOnFailover: 100,
      maxRetriesPerRequest: 3
    });
    
    this.memoryCache = new Map(); // In-memory cache for hot data
    this.cacheLevels = {
      MEMORY: 'memory',
      REDIS: 'redis',
      DATABASE: 'database'
    };
  }

  // Course content caching with TTL management
  async getCourse(courseId, includeContent = false) {
    const cacheKey = `course:${courseId}:${includeContent ? 'full' : 'meta'}`;
    
    try {
      // Level 1: Memory cache (fastest)
      if (this.memoryCache.has(cacheKey)) {
        return this.memoryCache.get(cacheKey);
      }

      // Level 2: Redis cache
      const cached = await this.redis.get(cacheKey);
      if (cached) {
        const course = JSON.parse(cached);
        
        // Populate memory cache for frequently accessed courses
        if (course.popularity > 80) {
          this.memoryCache.set(cacheKey, course);
          setTimeout(() => this.memoryCache.delete(cacheKey), 300000); // 5 min TTL
        }
        
        return course;
      }

      // Level 3: Database (slowest)
      const course = await this.fetchCourseFromDatabase(courseId, includeContent);
      
      if (course) {
        // Cache with appropriate TTL based on content type
        const ttl = includeContent ? 3600 : 1800; // 1 hour vs 30 minutes
        await this.redis.setex(cacheKey, ttl, JSON.stringify(course));
        
        // Track course popularity for memory cache decisions
        await this.incrementCoursePopularity(courseId);
      }

      return course;
    } catch (error) {
      console.error('Cache error:', error);
      return await this.fetchCourseFromDatabase(courseId, includeContent);
    }
  }

  // Progress tracking with real-time updates
  async getUserProgress(userId, courseId) {
    const cacheKey = `progress:${userId}:${courseId}`;
    
    try {
      // Always check Redis first for progress (real-time data)
      const cached = await this.redis.get(cacheKey);
      if (cached) {
        return JSON.parse(cached);
      }

      const progress = await this.fetchProgressFromDatabase(userId, courseId);
      
      if (progress) {
        // Short TTL for progress data (5 minutes)
        await this.redis.setex(cacheKey, 300, JSON.stringify(progress));
      }

      return progress;
    } catch (error) {
      return await this.fetchProgressFromDatabase(userId, courseId);
    }
  }

  // Course listing with pagination and filtering
  async getCourseList(filters, page = 1, limit = 20) {
    const filterHash = this.generateFilterHash(filters);
    const cacheKey = `courses:list:${filterHash}:${page}:${limit}`;
    
    try {
      const cached = await this.redis.get(cacheKey);
      if (cached) {
        return JSON.parse(cached);
      }

      const courseList = await this.fetchCourseListFromDatabase(filters, page, limit);
      
      // Cache course listings for 10 minutes
      await this.redis.setex(cacheKey, 600, JSON.stringify(courseList));
      
      return courseList;
    } catch (error) {
      return await this.fetchCourseListFromDatabase(filters, page, limit);
    }
  }

  // Cache invalidation strategies
  async invalidateCourseCache(courseId) {
    // Remove from all cache levels
    const patterns = [
      `course:${courseId}:*`,
      `courses:list:*`,
      `course:meta:${courseId}`,
      `course:full:${courseId}`
    ];

    for (const pattern of patterns) {
      const keys = await this.redis.keys(pattern);
      if (keys.length > 0) {
        await this.redis.del(...keys);
      }
    }

    // Clear memory cache
    for (const key of this.memoryCache.keys()) {
      if (key.includes(courseId)) {
        this.memoryCache.delete(key);
      }
    }
  }

  async invalidateUserProgressCache(userId, courseId = null) {
    if (courseId) {
      await this.redis.del(`progress:${userId}:${courseId}`);
    } else {
      // Invalidate all progress for user
      const keys = await this.redis.keys(`progress:${userId}:*`);
      if (keys.length > 0) {
        await this.redis.del(...keys);
      }
    }
  }

  // Cache warming for popular content
  async warmCache() {
    try {
      // Get popular courses
      const popularCourses = await this.getPopularCourses(50);
      
      for (const course of popularCourses) {
        // Pre-load course metadata
        await this.getCourse(course.id, false);
        
        // Pre-load course content for top 10 courses
        if (course.rank <= 10) {
          await this.getCourse(course.id, true);
        }
      }

      console.log(`Cache warmed with ${popularCourses.length} courses`);
    } catch (error) {
      console.error('Cache warming failed:', error);
    }
  }
}

module.exports = LMSCacheManager;
```

❌ **Bad Example: Poor Caching Strategy**
```javascript
// Poor caching - no strategy, no invalidation
class BadCacheManager {
  async getCourse(courseId) {
    // No cache levels, no TTL consideration
    const cached = await redis.get(`course_${courseId}`);
    if (cached) return JSON.parse(cached);
    
    const course = await db.course.findById(courseId);
    
    // No TTL, no invalidation strategy
    await redis.set(`course_${courseId}`, JSON.stringify(course));
    
    return course;
  }
}
```

## Database Performance Optimization

### Query Optimization Patterns
✅ **Good Example: Optimized LMS Database Queries**
```javascript
// optimizedQueries.js
class OptimizedLMSQueries {
  // Efficient course dashboard query
  async getUserDashboard(userId) {
    // Single optimized query instead of multiple round trips
    const query = `
      WITH user_enrollments AS (
        SELECT DISTINCT e.course_id, e.status, e.enrolled_at
        FROM enrollments e 
        WHERE e.user_id = $1 AND e.status = 'ACTIVE'
      ),
      course_progress AS (
        SELECT 
          p.course_id,
          COUNT(p.section_id) as completed_sections,
          SUM(p.time_spent) as total_time,
          MAX(p.completed_at) as last_activity
        FROM progress p 
        WHERE p.user_id = $1
        GROUP BY p.course_id
      ),
      course_totals AS (
        SELECT 
          c.id as course_id,
          COUNT(s.id) as total_sections
        FROM courses c
        JOIN lessons l ON l.course_id = c.id
        JOIN sections s ON s.lesson_id = l.id
        GROUP BY c.id
      )
      SELECT 
        c.id,
        c.title,
        c.image_url,
        c.estimated_hours,
        ue.status,
        ue.enrolled_at,
        COALESCE(cp.completed_sections, 0) as completed_sections,
        COALESCE(ct.total_sections, 0) as total_sections,
        COALESCE(cp.total_time, 0) as time_spent,
        cp.last_activity,
        CASE 
          WHEN ct.total_sections > 0 
          THEN ROUND((cp.completed_sections * 100.0 / ct.total_sections), 2)
          ELSE 0 
        END as completion_percentage
      FROM user_enrollments ue
      JOIN courses c ON c.id = ue.course_id
      LEFT JOIN course_progress cp ON cp.course_id = c.id
      LEFT JOIN course_totals ct ON ct.course_id = c.id
      ORDER BY cp.last_activity DESC NULLS LAST, ue.enrolled_at DESC
      LIMIT 10
    `;

    return await this.db.query(query, [userId]);
  }

  // Optimized course content loading with selective fields
  async getCourseWithSections(courseId, userId = null) {
    const query = `
      SELECT 
        c.id as course_id,
        c.title as course_title,
        c.description as course_description,
        c.difficulty,
        c.estimated_hours,
        
        l.id as lesson_id,
        l.title as lesson_title,
        l.order as lesson_order,
        
        s.id as section_id,
        s.title as section_title,
        s.type as section_type,
        s.order as section_order,
        s.duration,
        
        -- Only include progress if userId provided
        CASE WHEN $2 IS NOT NULL 
          THEN (SELECT TRUE FROM progress p WHERE p.user_id = $2 AND p.section_id = s.id LIMIT 1)
          ELSE NULL 
        END as is_completed,
        
        CASE WHEN $2 IS NOT NULL 
          THEN (SELECT p.completed_at FROM progress p WHERE p.user_id = $2 AND p.section_id = s.id LIMIT 1)
          ELSE NULL 
        END as completed_at
        
      FROM courses c
      JOIN lessons l ON l.course_id = c.id
      JOIN sections s ON s.lesson_id = l.id
      WHERE c.id = $1 
        AND c.is_published = true 
        AND c.deleted_at IS NULL
      ORDER BY l.order, s.order
    `;

    const rows = await this.db.query(query, [courseId, userId]);
    return this.formatCourseStructure(rows);
  }

  // Efficient search with full-text search and filters
  async searchCourses(searchTerm, filters = {}, page = 1, limit = 20) {
    const offset = (page - 1) * limit;
    
    const query = `
      WITH search_results AS (
        SELECT 
          c.id,
          c.title,
          c.description,
          c.difficulty,
          c.estimated_hours,
          c.image_url,
          c.created_at,
          
          -- Full-text search ranking
          ts_rank(
            to_tsvector('english', c.title || ' ' || COALESCE(c.description, '')),
            plainto_tsquery('english', $1)
          ) as search_rank,
          
          -- Course statistics
          (SELECT COUNT(*) FROM enrollments e WHERE e.course_id = c.id) as enrollment_count,
          (SELECT AVG(rating) FROM course_ratings cr WHERE cr.course_id = c.id) as avg_rating,
          (SELECT COUNT(*) FROM course_ratings cr WHERE cr.course_id = c.id) as rating_count
          
        FROM courses c
        WHERE c.is_published = true 
          AND c.deleted_at IS NULL
          AND ($1 = '' OR to_tsvector('english', c.title || ' ' || COALESCE(c.description, '')) @@ plainto_tsquery('english', $1))
          AND ($2 IS NULL OR c.difficulty = $2)
          AND ($3 IS NULL OR c.estimated_hours <= $3)
        ORDER BY 
          CASE WHEN $1 != '' THEN search_rank ELSE 0 END DESC,
          enrollment_count DESC,
          c.created_at DESC
      ),
      total_count AS (
        SELECT COUNT(*) as total FROM search_results
      )
      SELECT sr.*, tc.total
      FROM search_results sr
      CROSS JOIN total_count tc
      LIMIT $4 OFFSET $5
    `;

    return await this.db.query(query, [
      searchTerm || '',
      filters.difficulty || null,
      filters.maxHours || null,
      limit,
      offset
    ]);
  }

  formatCourseStructure(rows) {
    if (!rows.length) return null;

    const course = {
      id: rows[0].course_id,
      title: rows[0].course_title,
      description: rows[0].course_description,
      difficulty: rows[0].difficulty,
      estimatedHours: rows[0].estimated_hours,
      lessons: []
    };

    const lessonsMap = new Map();

    rows.forEach(row => {
      if (!lessonsMap.has(row.lesson_id)) {
        lessonsMap.set(row.lesson_id, {
          id: row.lesson_id,
          title: row.lesson_title,
          order: row.lesson_order,
          sections: []
        });
      }

      const lesson = lessonsMap.get(row.lesson_id);
      lesson.sections.push({
        id: row.section_id,
        title: row.section_title,
        type: row.section_type,
        order: row.section_order,
        duration: row.duration,
        isCompleted: row.is_completed || false,
        completedAt: row.completed_at
      });
    });

    course.lessons = Array.from(lessonsMap.values());
    return course;
  }
}
```

### Connection Pooling and Database Configuration
✅ **Good Example: Optimized Database Configuration**
```javascript
// databaseConfig.js
class DatabaseConfig {
  static createOptimizedPool() {
    return new Pool({
      host: process.env.DB_HOST,
      port: process.env.DB_PORT,
      database: process.env.DB_NAME,
      user: process.env.DB_USER,
      password: process.env.DB_PASSWORD,
      
      // Connection pool optimization
      min: 5,                    // Minimum connections in pool
      max: 30,                   // Maximum connections in pool
      acquireTimeoutMillis: 5000, // Max wait time for connection
      createTimeoutMillis: 3000,  // Max time to create connection
      destroyTimeoutMillis: 5000, // Max time to destroy connection
      idleTimeoutMillis: 30000,   // Connection idle timeout
      createRetryIntervalMillis: 200,
      
      // Performance optimizations
      statement_timeout: 10000,   // 10 second query timeout
      query_timeout: 8000,        // 8 second query timeout
      connectionTimeoutMillis: 2000,
      
      // SSL and connection options
      ssl: process.env.NODE_ENV === 'production' ? { rejectUnauthorized: false } : false,
      
      // Connection configuration
      options: [
        '--lock_timeout=5000',
        '--statement_timeout=10000'
      ].join(' ')
    });
  }

  // Database monitoring and health checks
  static async monitorConnectionHealth(pool) {
    setInterval(async () => {
      try {
        const client = await pool.connect();
        const result = await client.query('SELECT 1');
        client.release();
        
        console.log('Database health check: OK');
      } catch (error) {
        console.error('Database health check failed:', error);
        // Implement alerting here
      }
    }, 30000); // Check every 30 seconds
  }

  // Query performance monitoring
  static setupQueryMonitoring(pool) {
    pool.on('connect', client => {
      client.on('notice', msg => console.log('DB Notice:', msg));
      
      // Log slow queries
      const originalQuery = client.query;
      client.query = function(...args) {
        const start = Date.now();
        const result = originalQuery.apply(this, args);
        
        if (result && result.then) {
          return result.then(res => {
            const duration = Date.now() - start;
            if (duration > 1000) { // Log queries over 1 second
              console.warn(`Slow query (${duration}ms):`, args[0]);
            }
            return res;
          });
        }
        
        return result;
      };
    });
  }
}
```

## React Performance Optimization

### Component Optimization Strategies
✅ **Good Example: Optimized React Components for LMS**
```jsx
// OptimizedCourseList.jsx
import React, { memo, useMemo, useCallback, useState, useEffect } from 'react';
import { FixedSizeList as List } from 'react-window';

// Memoized course card component
const CourseCard = memo(({ course, onEnroll, onView, style }) => {
  const handleEnroll = useCallback(() => {
    onEnroll(course.id);
  }, [course.id, onEnroll]);

  const handleView = useCallback(() => {
    onView(course.id);
  }, [course.id, onView]);

  const progressPercentage = useMemo(() => {
    if (!course.progress) return 0;
    return Math.round((course.progress.completed / course.progress.total) * 100);
  }, [course.progress]);

  return (
    <div style={style} className="course-card">
      <div className="course-image">
        <img 
          src={course.imageUrl} 
          alt={course.title}
          loading="lazy"
          onError={(e) => {
            e.target.src = '/default-course-image.jpg';
          }}
        />
      </div>
      
      <div className="course-content">
        <h3>{course.title}</h3>
        <p className="course-description">{course.description}</p>
        
        <div className="course-meta">
          <span className="difficulty">{course.difficulty}</span>
          <span className="duration">{course.estimatedHours}h</span>
          <span className="rating">★ {course.rating || 'N/A'}</span>
        </div>

        {course.progress && (
          <div className="progress-bar">
            <div 
              className="progress-fill" 
              style={{ width: `${progressPercentage}%` }}
            />
            <span className="progress-text">{progressPercentage}% Complete</span>
          </div>
        )}

        <div className="course-actions">
          <button onClick={handleView} className="btn-secondary">
            View Course
          </button>
          {!course.isEnrolled && (
            <button onClick={handleEnroll} className="btn-primary">
              Enroll Now
            </button>
          )}
        </div>
      </div>
    </div>
  );
}, (prevProps, nextProps) => {
  // Custom comparison for optimization
  return (
    prevProps.course.id === nextProps.course.id &&
    prevProps.course.progress?.completed === nextProps.course.progress?.completed &&
    prevProps.course.isEnrolled === nextProps.course.isEnrolled
  );
});

// Virtualized course list for large datasets
const OptimizedCourseList = ({ courses, loading, onEnroll, onView }) => {
  const [searchTerm, setSearchTerm] = useState('');
  const [filteredCourses, setFilteredCourses] = useState(courses);

  // Debounced search
  useEffect(() => {
    const timer = setTimeout(() => {
      if (!searchTerm.trim()) {
        setFilteredCourses(courses);
      } else {
        const filtered = courses.filter(course =>
          course.title.toLowerCase().includes(searchTerm.toLowerCase()) ||
          course.description.toLowerCase().includes(searchTerm.toLowerCase())
        );
        setFilteredCourses(filtered);
      }
    }, 300);

    return () => clearTimeout(timer);
  }, [searchTerm, courses]);

  // Memoized callbacks to prevent unnecessary re-renders
  const handleEnroll = useCallback((courseId) => {
    onEnroll(courseId);
  }, [onEnroll]);

  const handleView = useCallback((courseId) => {
    onView(courseId);
  }, [onView]);

  // Virtualized list item renderer
  const CourseRow = useCallback(({ index, style }) => {
    const course = filteredCourses[index];
    return (
      <CourseCard
        course={course}
        onEnroll={handleEnroll}
        onView={handleView}
        style={style}
      />
    );
  }, [filteredCourses, handleEnroll, handleView]);

  if (loading) {
    return <CourseSkeleton count={6} />;
  }

  return (
    <div className="course-list-container">
      <div className="course-search">
        <input
          type="text"
          placeholder="Search courses..."
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
          className="search-input"
        />
      </div>

      {filteredCourses.length > 50 ? (
        // Use virtualization for large lists
        <List
          height={600}
          itemCount={filteredCourses.length}
          itemSize={250}
          width="100%"
        >
          {CourseRow}
        </List>
      ) : (
        // Regular rendering for smaller lists
        <div className="course-grid">
          {filteredCourses.map(course => (
            <CourseCard
              key={course.id}
              course={course}
              onEnroll={handleEnroll}
              onView={handleView}
            />
          ))}
        </div>
      )}
    </div>
  );
};

// Loading skeleton component
const CourseSkeleton = memo(({ count }) => {
  return (
    <div className="course-grid">
      {Array.from({ length: count }).map((_, index) => (
        <div key={index} className="course-card skeleton">
          <div className="skeleton-image" />
          <div className="skeleton-content">
            <div className="skeleton-title" />
            <div className="skeleton-description" />
            <div className="skeleton-meta" />
          </div>
        </div>
      ))}
    </div>
  );
});

export default memo(OptimizedCourseList);
```

### Custom Hooks for Performance
✅ **Good Example: Performance-Optimized Custom Hooks**
```jsx
// useOptimizedCourseData.js
import { useState, useEffect, useCallback, useRef } from 'react';

export function useOptimizedCourseData(userId) {
  const [courses, setCourses] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const cacheRef = useRef(new Map());
  const abortControllerRef = useRef(null);

  // Fetch courses with caching and request deduplication
  const fetchCourses = useCallback(async (forceRefresh = false) => {
    const cacheKey = `courses_${userId}`;
    
    // Return cached data if available and not forcing refresh
    if (!forceRefresh && cacheRef.current.has(cacheKey)) {
      const cached = cacheRef.current.get(cacheKey);
      if (Date.now() - cached.timestamp < 300000) { // 5 minutes
        setCourses(cached.data);
        setLoading(false);
        return;
      }
    }

    // Cancel previous request if still pending
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }

    abortControllerRef.current = new AbortController();
    setLoading(true);
    setError(null);

    try {
      const response = await fetch(`/api/users/${userId}/courses`, {
        signal: abortControllerRef.current.signal,
        headers: {
          'Cache-Control': forceRefresh ? 'no-cache' : 'max-age=300'
        }
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const data = await response.json();
      
      // Cache the result
      cacheRef.current.set(cacheKey, {
        data: data.courses,
        timestamp: Date.now()
      });

      setCourses(data.courses);
    } catch (err) {
      if (err.name !== 'AbortError') {
        setError(err.message);
        console.error('Failed to fetch courses:', err);
      }
    } finally {
      setLoading(false);
    }
  }, [userId]);

  // Optimized course enrollment
  const enrollInCourse = useCallback(async (courseId) => {
    try {
      const response = await fetch(`/api/courses/${courseId}/enroll`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ userId })
      });

      if (!response.ok) {
        throw new Error('Enrollment failed');
      }

      // Optimistically update the UI
      setCourses(prevCourses => 
        prevCourses.map(course => 
          course.id === courseId 
            ? { ...course, isEnrolled: true, enrollmentDate: new Date().toISOString() }
            : course
        )
      );

      // Invalidate cache
      const cacheKey = `courses_${userId}`;
      cacheRef.current.delete(cacheKey);

    } catch (err) {
      setError(err.message);
      // Revert optimistic update
      fetchCourses(true);
    }
  }, [userId, fetchCourses]);

  // Update course progress
  const updateProgress = useCallback(async (courseId, sectionId) => {
    try {
      const response = await fetch(`/api/progress`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          userId,
          courseId,
          sectionId
        })
      });

      if (!response.ok) {
        throw new Error('Progress update failed');
      }

      const updatedProgress = await response.json();

      // Update local state
      setCourses(prevCourses =>
        prevCourses.map(course =>
          course.id === courseId
            ? { ...course, progress: updatedProgress }
            : course
        )
      );

    } catch (err) {
      setError(err.message);
    }
  }, [userId]);

  // Cleanup on unmount
  useEffect(() => {
    return () => {
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, []);

  // Initial data fetch
  useEffect(() => {
    fetchCourses();
  }, [fetchCourses]);

  return {
    courses,
    loading,
    error,
    refetch: () => fetchCourses(true),
    enrollInCourse,
    updateProgress
  };
}

// Hook for optimized search with debouncing
export function useOptimizedCourseSearch() {
  const [searchTerm, setSearchTerm] = useState('');
  const [searchResults, setSearchResults] = useState([]);
  const [searching, setSearching] = useState(false);
  const searchTimeoutRef = useRef(null);
  const abortControllerRef = useRef(null);

  const performSearch = useCallback(async (term) => {
    if (!term.trim()) {
      setSearchResults([]);
      return;
    }

    // Cancel previous search
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }

    abortControllerRef.current = new AbortController();
    setSearching(true);

    try {
      const response = await fetch(`/api/courses/search?q=${encodeURIComponent(term)}`, {
        signal: abortControllerRef.current.signal
      });

      if (!response.ok) {
        throw new Error('Search failed');
      }

      const data = await response.json();
      setSearchResults(data.courses);
    } catch (err) {
      if (err.name !== 'AbortError') {
        console.error('Search error:', err);
      }
    } finally {
      setSearching(false);
    }
  }, []);

  const debouncedSearch = useCallback((term) => {
    // Clear previous timeout
    if (searchTimeoutRef.current) {
      clearTimeout(searchTimeoutRef.current);
    }

    // Set new timeout
    searchTimeoutRef.current = setTimeout(() => {
      performSearch(term);
    }, 300);
  }, [performSearch]);

  const updateSearchTerm = useCallback((term) => {
    setSearchTerm(term);
    debouncedSearch(term);
  }, [debouncedSearch]);

  useEffect(() => {
    return () => {
      if (searchTimeoutRef.current) {
        clearTimeout(searchTimeoutRef.current);
      }
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, []);

  return {
    searchTerm,
    searchResults,
    searching,
    updateSearchTerm,
    clearSearch: () => {
      setSearchTerm('');
      setSearchResults([]);
    }
  };
}
```

## CDN and Asset Optimization

### Static Asset Optimization
✅ **Good Example: Comprehensive Asset Optimization**
```javascript
// assetOptimization.js
class AssetOptimizationManager {
  constructor() {
    this.cdnBaseUrl = process.env.CDN_BASE_URL;
    this.imageOptimizationApi = process.env.IMAGE_OPTIMIZATION_API;
  }

  // Optimize and serve course images
  generateOptimizedImageUrl(originalUrl, options = {}) {
    const {
      width = 'auto',
      height = 'auto',
      quality = 80,
      format = 'webp',
      resize = 'cover'
    } = options;

    // Use CDN transformation parameters
    const params = new URLSearchParams({
      w: width,
      h: height,
      q: quality,
      f: format,
      fit: resize
    });

    return `${this.cdnBaseUrl}/images/${originalUrl}?${params.toString()}`;
  }

  // Generate responsive image srcSet
  generateResponsiveImageSrcSet(originalUrl, formats = ['webp', 'jpg']) {
    const breakpoints = [320, 640, 768, 1024, 1280, 1920];
    const srcSets = {};

    formats.forEach(format => {
      srcSets[format] = breakpoints
        .map(width => {
          const url = this.generateOptimizedImageUrl(originalUrl, {
            width,
            format,
            quality: format === 'webp' ? 75 : 80
          });
          return `${url} ${width}w`;
        })
        .join(', ');
    });

    return srcSets;
  }

  // Preload critical course content
  async preloadCriticalAssets(courseId) {
    try {
      // Preload course thumbnail
      const courseData = await this.getCourseMetadata(courseId);
      if (courseData.imageUrl) {
        this.preloadImage(
          this.generateOptimizedImageUrl(courseData.imageUrl, {
            width: 400,
            height: 300
          })
        );
      }

      // Preload first lesson video thumbnail
      if (courseData.firstVideoThumbnail) {
        this.preloadImage(
          this.generateOptimizedImageUrl(courseData.firstVideoThumbnail, {
            width: 640,
            height: 360
          })
        );
      }

      // Preload critical CSS for course page
      this.preloadResource('/css/course-page.css', 'style');
      
      // Preload course content JSON
      this.preloadResource(`/api/courses/${courseId}/content`, 'fetch');

    } catch (error) {
      console.error('Failed to preload critical assets:', error);
    }
  }

  preloadImage(src) {
    const link = document.createElement('link');
    link.rel = 'preload';
    link.as = 'image';
    link.href = src;
    document.head.appendChild(link);
  }

  preloadResource(href, as, crossorigin = false) {
    const link = document.createElement('link');
    link.rel = 'preload';
    link.as = as;
    link.href = href;
    if (crossorigin) link.crossOrigin = 'anonymous';
    document.head.appendChild(link);
  }

  // Lazy loading with intersection observer
  setupLazyLoading() {
    const imageObserver = new IntersectionObserver(
      (entries, observer) => {
        entries.forEach(entry => {
          if (entry.isIntersecting) {
            const img = entry.target;
            const src = img.dataset.src;
            const srcset = img.dataset.srcset;

            if (src) img.src = src;
            if (srcset) img.srcset = srcset;
            
            img.classList.add('loaded');
            observer.unobserve(img);
          }
        });
      },
      {
        rootMargin: '50px 0px',
        threshold: 0.01
      }
    );

    // Observe all lazy images
    document.querySelectorAll('img[data-src]').forEach(img => {
      imageObserver.observe(img);
    });

    return imageObserver;
  }

  // Progressive image loading
  createProgressiveImage(src, placeholder = null) {
    const container = document.createElement('div');
    container.className = 'progressive-image';

    // Show placeholder first
    if (placeholder) {
      const placeholderImg = document.createElement('img');
      placeholderImg.src = placeholder;
      placeholderImg.className = 'placeholder';
      container.appendChild(placeholderImg);
    }

    // Load full image
    const fullImg = document.createElement('img');
    fullImg.onload = () => {
      container.classList.add('loaded');
      if (placeholder) {
        setTimeout(() => {
          container.removeChild(container.querySelector('.placeholder'));
        }, 300);
      }
    };
    fullImg.src = src;
    fullImg.className = 'full-image';
    container.appendChild(fullImg);

    return container;
  }
}

// React component for optimized images
const OptimizedImage = ({ src, alt, width, height, className, loading = 'lazy' }) => {
  const [imageLoaded, setImageLoaded] = useState(false);
  const [imageError, setImageError] = useState(false);
  const assetManager = new AssetOptimizationManager();

  const optimizedSrc = useMemo(() => {
    return assetManager.generateOptimizedImageUrl(src, {
      width,
      height,
      quality: 80,
      format: 'webp'
    });
  }, [src, width, height]);

  const fallbackSrc = useMemo(() => {
    return assetManager.generateOptimizedImageUrl(src, {
      width,
      height,
      quality: 85,
      format: 'jpg'
    });
  }, [src, width, height]);

  return (
    <div className={`optimized-image-container ${className || ''}`}>
      {!imageLoaded && !imageError && (
        <div className="image-placeholder">
          <div className="placeholder-content">Loading...</div>
        </div>
      )}
      
      <picture>
        <source srcSet={optimizedSrc} type="image/webp" />
        <img
          src={fallbackSrc}
          alt={alt}
          loading={loading}
          className={`optimized-image ${imageLoaded ? 'loaded' : ''}`}
          onLoad={() => setImageLoaded(true)}
          onError={() => setImageError(true)}
        />
      </picture>
      
      {imageError && (
        <div className="image-error">
          <span>Failed to load image</span>
        </div>
      )}
    </div>
  );
};

export { AssetOptimizationManager, OptimizedImage };
```

## Key Performance Takeaways for LMS Systems

### Critical Performance Optimizations ✅

1. **Multi-Layer Caching Strategy**
   - Memory cache for hot data (frequently accessed courses)
   - Redis cache for user sessions and course metadata
   - Database query optimization with proper indexing
   - CDN for static assets and course materials

2. **Database Performance**
   - Connection pooling with optimal configuration
   - Query optimization with selective field loading
   - Proper indexing strategy for search and filtering
   - Progress tracking optimization for real-time updates

3. **React Frontend Optimization**
   - Component memoization and virtualization for large lists
   - Custom hooks with caching and request deduplication
   - Lazy loading and code splitting for course content
   - Optimistic updates for better user experience

4. **Asset Optimization**
   - Image optimization with multiple formats (WebP/JPEG)
   - Responsive images with appropriate breakpoints
   - Progressive loading for better perceived performance
   - Critical resource preloading for course pages

5. **Monitoring and Metrics**
   - Performance monitoring for database queries
   - Cache hit rate tracking and optimization
   - User experience metrics (loading times, interaction delays)
   - Error tracking and performance degradation alerts

### Performance Anti-Patterns to Avoid ❌

1. **No Caching Strategy** - Loading course data from database on every request
2. **N+1 Query Problems** - Fetching related data in loops instead of joins
3. **Unoptimized Images** - Serving full-resolution images for all screen sizes
4. **Blocking Operations** - Synchronous operations that freeze the UI
5. **Memory Leaks** - Not cleaning up event listeners, timers, and subscriptions

The LMS platform handles intensive operations like course generation, progress tracking, and content delivery. These performance optimizations ensure the system scales effectively with increasing users and course content. 