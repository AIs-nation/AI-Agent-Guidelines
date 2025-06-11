# React Production Performance Patterns for LMS Platforms

## Core Performance Optimization Strategies

### React.memo and Memoization Patterns
**Essential for LMS course listings and lesson components**

✅ **Good Example: Course Card Memoization**
```jsx
// CourseCard.jsx
import { memo, useMemo } from 'react';

const CourseCard = memo(({ course, userProgress, onEnroll }) => {
  // Memoize expensive calculations
  const completionPercentage = useMemo(() => {
    if (!userProgress?.lessonProgress) return 0;
    const totalSections = course.lessons.reduce((acc, lesson) => acc + lesson.sections.length, 0);
    const completedSections = userProgress.lessonProgress.reduce(
      (acc, lesson) => acc + lesson.sections.filter(s => s.completed).length, 0
    );
    return Math.round((completedSections / totalSections) * 100);
  }, [course.lessons, userProgress?.lessonProgress]);

  const courseMetadata = useMemo(() => ({
    totalLessons: course.lessons.length,
    estimatedHours: course.lessons.reduce((acc, lesson) => 
      acc + lesson.sections.reduce((sAcc, section) => sAcc + (section.duration || 0), 0), 0
    ) / 3600,
    difficulty: course.difficulty
  }), [course.lessons, course.difficulty]);

  return (
    <div className="course-card" data-testid="course-card">
      <img src={course.imageUrl} alt={course.title} loading="lazy" />
      <div className="course-content">
        <h3>{course.title}</h3>
        <p>{course.description}</p>
        <div className="course-meta">
          <span>{courseMetadata.totalLessons} lessons</span>
          <span>{courseMetadata.estimatedHours}h</span>
          <span>{courseMetadata.difficulty}</span>
        </div>
        {userProgress && (
          <div className="progress-bar">
            <div 
              className="progress-fill" 
              style={{ width: `${completionPercentage}%` }}
            />
            <span>{completionPercentage}% complete</span>
          </div>
        )}
        <button onClick={() => onEnroll(course.id)}>
          {completionPercentage > 0 ? 'Continue Learning' : 'Start Course'}
        </button>
      </div>
    </div>
  );
});

// Comparison function for memo optimization
CourseCard.displayName = 'CourseCard';
export default CourseCard;
```

❌ **Bad Example: Unoptimized Course Card**
```jsx
// Poor performance - no memoization, recalculates on every render
const CourseCard = ({ course, userProgress, onEnroll }) => {
  // Expensive calculation on every render
  const totalSections = course.lessons.reduce((acc, lesson) => acc + lesson.sections.length, 0);
  const completedSections = userProgress?.lessonProgress?.reduce(
    (acc, lesson) => acc + lesson.sections.filter(s => s.completed).length, 0
  ) || 0;
  const completionPercentage = Math.round((completedSections / totalSections) * 100);

  return (
    <div className="course-card">
      {/* Component content with no memoization */}
    </div>
  );
};
```

### useCallback for Event Handler Optimization
✅ **Good Example: Optimized Event Handlers**
```jsx
// LessonView.jsx
import { useCallback, useMemo, useState } from 'react';

const LessonView = ({ lesson, courseId, onProgressUpdate }) => {
  const [currentSectionIndex, setCurrentSectionIndex] = useState(0);
  const [completedSections, setCompletedSections] = useState(new Set());

  // Memoize event handlers to prevent child re-renders
  const handleSectionComplete = useCallback((sectionId) => {
    setCompletedSections(prev => new Set([...prev, sectionId]));
    
    // Report progress to parent
    onProgressUpdate({
      lessonId: lesson.id,
      sectionId,
      completed: true,
      timeSpent: Date.now() - startTime
    });
  }, [lesson.id, onProgressUpdate]);

  const handleNextSection = useCallback(() => {
    if (currentSectionIndex < lesson.sections.length - 1) {
      setCurrentSectionIndex(prev => prev + 1);
    }
  }, [currentSectionIndex, lesson.sections.length]);

  const handlePreviousSection = useCallback(() => {
    if (currentSectionIndex > 0) {
      setCurrentSectionIndex(prev => prev - 1);
    }
  }, [currentSectionIndex]);

  // Memoize current section to prevent unnecessary re-renders
  const currentSection = useMemo(() => 
    lesson.sections[currentSectionIndex], 
    [lesson.sections, currentSectionIndex]
  );

  const progressData = useMemo(() => ({
    currentIndex: currentSectionIndex + 1,
    totalSections: lesson.sections.length,
    completedCount: completedSections.size,
    percentage: Math.round((completedSections.size / lesson.sections.length) * 100)
  }), [currentSectionIndex, lesson.sections.length, completedSections.size]);

  return (
    <div className="lesson-view">
      <LessonHeader 
        title={lesson.title}
        progress={progressData}
      />
      <SectionContent
        section={currentSection}
        isCompleted={completedSections.has(currentSection.id)}
        onComplete={handleSectionComplete}
      />
      <LessonNavigation
        canGoPrevious={currentSectionIndex > 0}
        canGoNext={currentSectionIndex < lesson.sections.length - 1}
        onPrevious={handlePreviousSection}
        onNext={handleNextSection}
      />
    </div>
  );
};
```

### Virtual Scrolling for Large Datasets
✅ **Good Example: Virtualized Course List**
```jsx
// VirtualizedCourseList.jsx
import { FixedSizeList as List } from 'react-window';
import { memo, useMemo } from 'react';

const VirtualizedCourseList = memo(({ courses, userProgress, onCourseSelect }) => {
  // Memoize course data with progress
  const coursesWithProgress = useMemo(() => 
    courses.map(course => ({
      ...course,
      progress: userProgress[course.id] || null
    })), 
    [courses, userProgress]
  );

  const CourseListItem = memo(({ index, style }) => {
    const course = coursesWithProgress[index];
    
    return (
      <div style={style}>
        <CourseCard
          course={course}
          userProgress={course.progress}
          onEnroll={onCourseSelect}
        />
      </div>
    );
  });

  const itemCount = coursesWithProgress.length;
  const itemSize = 320; // Height of each course card
  const listHeight = Math.min(window.innerHeight - 200, itemCount * itemSize);

  return (
    <div className="virtualized-course-list" data-testid="virtualized-course-list">
      <List
        height={listHeight}
        itemCount={itemCount}
        itemSize={itemSize}
        itemData={coursesWithProgress}
        overscanCount={5} // Render 5 extra items for smooth scrolling
      >
        {CourseListItem}
      </List>
    </div>
  );
});

VirtualizedCourseList.displayName = 'VirtualizedCourseList';
export default VirtualizedCourseList;
```

### State Management Optimization Patterns
✅ **Good Example: Optimized Context and State Slicing**
```jsx
// contexts/LMSContext.jsx
import { createContext, useContext, useMemo, useReducer } from 'react';

// Separate contexts to prevent unnecessary re-renders
const CourseDataContext = createContext();
const UserProgressContext = createContext();
const UIStateContext = createContext();

const courseDataReducer = (state, action) => {
  switch (action.type) {
    case 'SET_COURSES':
      return { ...state, courses: action.payload };
    case 'ADD_COURSE':
      return { ...state, courses: [...state.courses, action.payload] };
    case 'UPDATE_COURSE':
      return {
        ...state,
        courses: state.courses.map(course =>
          course.id === action.payload.id ? { ...course, ...action.payload } : course
        )
      };
    default:
      return state;
  }
};

const userProgressReducer = (state, action) => {
  switch (action.type) {
    case 'UPDATE_PROGRESS':
      return {
        ...state,
        [action.payload.courseId]: {
          ...state[action.payload.courseId],
          lessonProgress: updateLessonProgress(
            state[action.payload.courseId]?.lessonProgress || [],
            action.payload
          )
        }
      };
    case 'SET_USER_PROGRESS':
      return action.payload;
    default:
      return state;
  }
};

export const LMSProvider = ({ children }) => {
  const [courseData, dispatchCourseData] = useReducer(courseDataReducer, {
    courses: [],
    loading: false,
    error: null
  });

  const [userProgress, dispatchUserProgress] = useReducer(userProgressReducer, {});

  const [uiState, setUIState] = useState({
    currentView: 'dashboard',
    sidebarOpen: false,
    notifications: []
  });

  // Memoize context values to prevent unnecessary re-renders
  const courseDataValue = useMemo(() => ({
    ...courseData,
    dispatch: dispatchCourseData
  }), [courseData]);

  const userProgressValue = useMemo(() => ({
    progress: userProgress,
    dispatch: dispatchUserProgress
  }), [userProgress]);

  const uiStateValue = useMemo(() => ({
    ...uiState,
    setUIState
  }), [uiState]);

  return (
    <CourseDataContext.Provider value={courseDataValue}>
      <UserProgressContext.Provider value={userProgressValue}>
        <UIStateContext.Provider value={uiStateValue}>
          {children}
        </UIStateContext.Provider>
      </UserProgressContext.Provider>
    </CourseDataContext.Provider>
  );
};

// Custom hooks for context consumption
export const useCourseData = () => {
  const context = useContext(CourseDataContext);
  if (!context) {
    throw new Error('useCourseData must be used within LMSProvider');
  }
  return context;
};

export const useUserProgress = () => {
  const context = useContext(UserProgressContext);
  if (!context) {
    throw new Error('useUserProgress must be used within LMSProvider');
  }
  return context;
};
```

### Code Splitting and Lazy Loading
✅ **Good Example: Route-based Code Splitting**
```jsx
// App.jsx
import { lazy, Suspense } from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';

// Lazy load route components
const Dashboard = lazy(() => import('./pages/Dashboard'));
const CourseList = lazy(() => import('./pages/CourseList'));
const CourseDetail = lazy(() => import('./pages/CourseDetail'));
const LessonView = lazy(() => import('./pages/LessonView'));
const Analytics = lazy(() => import('./pages/Analytics'));
const CourseEditor = lazy(() => import('./pages/CourseEditor'));

// Loading component for suspense fallback
const PageLoader = () => (
  <div className="page-loader">
    <div className="loader-spinner" />
    <p>Loading...</p>
  </div>
);

function App() {
  return (
    <Router>
      <div className="app">
        <Navigation />
        <main className="main-content">
          <Suspense fallback={<PageLoader />}>
            <Routes>
              <Route path="/" element={<Dashboard />} />
              <Route path="/courses" element={<CourseList />} />
              <Route path="/courses/:courseId" element={<CourseDetail />} />
              <Route path="/courses/:courseId/lessons/:lessonId" element={<LessonView />} />
              <Route path="/analytics" element={<Analytics />} />
              <Route path="/create" element={<CourseEditor />} />
            </Routes>
          </Suspense>
        </main>
      </div>
    </Router>
  );
}

export default App;
```

### Component-level Code Splitting
✅ **Good Example: Dynamic Component Loading**
```jsx
// pages/CourseDetail.jsx
import { lazy, Suspense, useState, useEffect } from 'react';

// Dynamically import heavy components
const AITeacher = lazy(() => import('../components/AITeacher'));
const AdvancedAnalytics = lazy(() => import('../components/AdvancedAnalytics'));
const CourseReviews = lazy(() => import('../components/CourseReviews'));

const CourseDetail = ({ courseId }) => {
  const [activeTab, setActiveTab] = useState('overview');
  const [course, setCourse] = useState(null);

  useEffect(() => {
    // Load course data
    fetchCourse(courseId).then(setCourse);
  }, [courseId]);

  const ComponentLoader = ({ children }) => (
    <Suspense fallback={<div className="component-loader">Loading...</div>}>
      {children}
    </Suspense>
  );

  if (!course) return <div>Loading course...</div>;

  return (
    <div className="course-detail">
      <CourseHeader course={course} />
      
      <div className="course-tabs">
        <button 
          className={activeTab === 'overview' ? 'active' : ''}
          onClick={() => setActiveTab('overview')}
        >
          Overview
        </button>
        <button 
          className={activeTab === 'ai-teacher' ? 'active' : ''}
          onClick={() => setActiveTab('ai-teacher')}
        >
          AI Teacher
        </button>
        <button 
          className={activeTab === 'analytics' ? 'active' : ''}
          onClick={() => setActiveTab('analytics')}
        >
          Analytics
        </button>
        <button 
          className={activeTab === 'reviews' ? 'active' : ''}
          onClick={() => setActiveTab('reviews')}
        >
          Reviews
        </button>
      </div>

      <div className="course-content">
        {activeTab === 'overview' && <CourseOverview course={course} />}
        
        {activeTab === 'ai-teacher' && (
          <ComponentLoader>
            <AITeacher courseId={courseId} />
          </ComponentLoader>
        )}
        
        {activeTab === 'analytics' && (
          <ComponentLoader>
            <AdvancedAnalytics courseId={courseId} />
          </ComponentLoader>
        )}
        
        {activeTab === 'reviews' && (
          <ComponentLoader>
            <CourseReviews courseId={courseId} />
          </ComponentLoader>
        )}
      </div>
    </div>
  );
};

export default CourseDetail;
```

### Image Optimization for LMS Content
✅ **Good Example: Optimized Image Loading**
```jsx
// components/OptimizedImage.jsx
import { useState, useRef, useEffect } from 'react';

const OptimizedImage = ({ 
  src, 
  alt, 
  width, 
  height, 
  className = '',
  loading = 'lazy',
  priority = false 
}) => {
  const [imageLoaded, setImageLoaded] = useState(false);
  const [imageError, setImageError] = useState(false);
  const imgRef = useRef();

  // Generate responsive image URLs
  const generateSrcSet = (baseSrc) => {
    const sizes = [320, 480, 768, 1024, 1280];
    return sizes
      .map(size => `${baseSrc}?w=${size}&q=75 ${size}w`)
      .join(', ');
  };

  const handleLoad = () => {
    setImageLoaded(true);
  };

  const handleError = () => {
    setImageError(true);
  };

  // Intersection Observer for lazy loading
  useEffect(() => {
    if (loading === 'lazy' && !priority) {
      const observer = new IntersectionObserver(
        (entries) => {
          entries.forEach(entry => {
            if (entry.isIntersecting) {
              const img = entry.target;
              if (img.dataset.src) {
                img.src = img.dataset.src;
                img.srcset = img.dataset.srcset;
                observer.unobserve(img);
              }
            }
          });
        },
        { threshold: 0.1 }
      );

      if (imgRef.current) {
        observer.observe(imgRef.current);
      }

      return () => observer.disconnect();
    }
  }, [loading, priority]);

  if (imageError) {
    return (
      <div className={`image-error ${className}`} style={{ width, height }}>
        <div className="error-placeholder">
          <span>Image not available</span>
        </div>
      </div>
    );
  }

  return (
    <div className={`image-container ${className}`}>
      {!imageLoaded && (
        <div className="image-placeholder" style={{ width, height }}>
          <div className="placeholder-shimmer" />
        </div>
      )}
      
      <img
        ref={imgRef}
        src={priority ? src : undefined}
        data-src={!priority ? src : undefined}
        data-srcset={!priority ? generateSrcSet(src) : undefined}
        srcSet={priority ? generateSrcSet(src) : undefined}
        alt={alt}
        width={width}
        height={height}
        loading={priority ? 'eager' : 'lazy'}
        onLoad={handleLoad}
        onError={handleError}
        className={`optimized-image ${imageLoaded ? 'loaded' : 'loading'}`}
        sizes="(max-width: 768px) 100vw, (max-width: 1024px) 50vw, 33vw"
      />
    </div>
  );
};

export default OptimizedImage;
```

## Key Performance Optimization Takeaways

### Critical Performance Patterns ✅

1. **Component Memoization**
   - Use React.memo for components that receive props frequently
   - Implement useMemo for expensive calculations
   - Apply useCallback for event handlers passed to children

2. **State Management Optimization**
   - Separate contexts to prevent cascading re-renders
   - Use state slicing for large applications
   - Implement proper reducer patterns for complex state

3. **Code Splitting Strategies**
   - Route-based splitting for page components
   - Component-level splitting for heavy features
   - Dynamic imports with proper loading states

4. **Virtual Scrolling**
   - Essential for large course lists (1000+ items)
   - Implement with react-window for optimal performance
   - Use proper overscan for smooth scrolling

5. **Image Optimization**
   - Implement lazy loading with Intersection Observer
   - Use responsive images with srcset
   - Provide proper loading states and error handling

### Performance Anti-Patterns to Avoid ❌

1. **Excessive Re-renders** - Not memoizing expensive calculations in course/progress components
2. **Prop Drilling** - Passing course data through multiple component layers without context
3. **Unoptimized Lists** - Rendering large course lists without virtualization
4. **Blocking Operations** - Loading all course content synchronously without code splitting  
5. **Memory Leaks** - Not cleaning up event listeners and timers in learning components

React performance optimization is crucial for LMS platforms handling large datasets of courses, lessons, and user progress. These patterns ensure smooth user experience even with hundreds of courses and thousands of lessons. 