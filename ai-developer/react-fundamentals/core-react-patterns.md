# React Fundamentals for LMS Development

## Component Architecture Patterns

### LMS Component Hierarchy Design
**Critical for scalable educational platform structure**

✅ **Good Example: Well-Structured LMS Component Architecture**
```jsx
// App.jsx - Top-level application structure
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import { LMSProvider } from './contexts/LMSContext';
import { ErrorBoundary } from './components/ErrorBoundary';
import { Navigation } from './components/Navigation';
import { Suspense, lazy } from 'react';

// Lazy load major route components
const Dashboard = lazy(() => import('./pages/Dashboard'));
const CourseList = lazy(() => import('./pages/CourseList'));
const CourseDetail = lazy(() => import('./pages/CourseDetail'));
const LessonView = lazy(() => import('./pages/LessonView'));

function App() {
  return (
    <ErrorBoundary>
      <LMSProvider>
        <Router>
          <div className="app">
            <Navigation />
            <main className="main-content">
              <Suspense fallback={<div className="loading">Loading...</div>}>
                <Routes>
                  <Route path="/" element={<Dashboard />} />
                  <Route path="/courses" element={<CourseList />} />
                  <Route path="/courses/:courseId" element={<CourseDetail />} />
                  <Route path="/courses/:courseId/lessons/:lessonId" element={<LessonView />} />
                </Routes>
              </Suspense>
            </main>
          </div>
        </Router>
      </LMSProvider>
    </ErrorBoundary>
  );
}

export default App;
```

### Component Composition Patterns
✅ **Good Example: Reusable LMS Components**
```jsx
// components/CourseCard.jsx - Compositional design
import { memo, useState } from 'react';
import { ProgressBar } from './ProgressBar';
import { DifficultyBadge } from './DifficultyBadge';
import { CourseThumbnail } from './CourseThumbnail';

const CourseCard = memo(({ 
  course, 
  userProgress, 
  onEnroll, 
  onContinue,
  showProgress = true,
  size = 'medium' 
}) => {
  const [isLoading, setIsLoading] = useState(false);

  const handleAction = async () => {
    setIsLoading(true);
    try {
      if (userProgress) {
        await onContinue(course.id);
      } else {
        await onEnroll(course.id);
      }
    } finally {
      setIsLoading(false);
    }
  };

  const cardClasses = `course-card course-card--${size} ${
    isLoading ? 'course-card--loading' : ''
  }`;

  return (
    <article className={cardClasses} data-testid="course-card">
      <CourseCardHeader course={course} />
      <CourseCardBody course={course} showProgress={showProgress} userProgress={userProgress} />
      <CourseCardFooter 
        course={course}
        userProgress={userProgress}
        onAction={handleAction}
        isLoading={isLoading}
      />
    </article>
  );
});

// Smaller, focused sub-components
const CourseCardHeader = ({ course }) => (
  <header className="course-card__header">
    <CourseThumbnail 
      src={course.imageUrl} 
      alt={course.title}
      width={300}
      height={200}
    />
    <DifficultyBadge difficulty={course.difficulty} />
  </header>
);

const CourseCardBody = ({ course, showProgress, userProgress }) => (
  <div className="course-card__body">
    <h3 className="course-card__title">{course.title}</h3>
    <p className="course-card__description">{course.description}</p>
    
    <div className="course-card__meta">
      <span className="meta-item">
        {course.lessonCount} lessons
      </span>
      <span className="meta-item">
        {course.estimatedHours}h
      </span>
    </div>

    {showProgress && userProgress && (
      <ProgressBar 
        current={userProgress.completedLessons}
        total={course.lessonCount}
        showPercentage
      />
    )}
  </div>
);

const CourseCardFooter = ({ course, userProgress, onAction, isLoading }) => (
  <footer className="course-card__footer">
    <button 
      className="course-card__action-btn"
      onClick={onAction}
      disabled={isLoading}
      data-testid="course-action-button"
    >
      {isLoading ? (
        <span>Loading...</span>
      ) : userProgress ? (
        'Continue Learning'
      ) : (
        'Start Course'
      )}
    </button>
  </footer>
);

CourseCard.displayName = 'CourseCard';
export { CourseCard };
```

❌ **Bad Example: Monolithic Component**
```jsx
// Poor component design - everything in one large component
const BadCourseCard = ({ course, userProgress, onEnroll }) => {
  return (
    <div className="course-card">
      {/* 200+ lines of mixed concerns */}
      <img src={course.imageUrl} alt={course.title} />
      <div>{course.title}</div>
      <div>{course.description}</div>
      {/* Complex progress calculation logic inline */}
      {/* Multiple unrelated event handlers */}
      {/* No composition or reusability */}
    </div>
  );
};
```

## State Management Patterns

### Context-Based State Management for LMS
✅ **Good Example: Structured Context Architecture**
```jsx
// contexts/LMSContext.jsx - Separated concerns with multiple contexts
import { createContext, useContext, useReducer, useMemo } from 'react';

// Separate contexts for different data domains
const CourseDataContext = createContext();
const UserProgressContext = createContext();
const UIStateContext = createContext();

// Course data reducer
const courseDataReducer = (state, action) => {
  switch (action.type) {
    case 'LOAD_COURSES_START':
      return { ...state, loading: true, error: null };
    
    case 'LOAD_COURSES_SUCCESS':
      return { 
        ...state, 
        courses: action.payload, 
        loading: false,
        lastUpdated: Date.now()
      };
    
    case 'LOAD_COURSES_ERROR':
      return { 
        ...state, 
        loading: false, 
        error: action.payload 
      };
    
    case 'ADD_COURSE':
      return {
        ...state,
        courses: [...state.courses, action.payload]
      };
    
    case 'UPDATE_COURSE':
      return {
        ...state,
        courses: state.courses.map(course =>
          course.id === action.payload.id 
            ? { ...course, ...action.payload }
            : course
        )
      };
    
    default:
      return state;
  }
};

// User progress reducer
const userProgressReducer = (state, action) => {
  switch (action.type) {
    case 'UPDATE_SECTION_PROGRESS':
      const { courseId, lessonId, sectionId, completed, timeSpent } = action.payload;
      
      return {
        ...state,
        [courseId]: {
          ...state[courseId],
          lessons: {
            ...state[courseId]?.lessons,
            [lessonId]: {
              ...state[courseId]?.lessons?.[lessonId],
              sections: {
                ...state[courseId]?.lessons?.[lessonId]?.sections,
                [sectionId]: {
                  completed,
                  timeSpent: (state[courseId]?.lessons?.[lessonId]?.sections?.[sectionId]?.timeSpent || 0) + timeSpent,
                  lastUpdated: Date.now()
                }
              }
            }
          }
        }
      };
    
    case 'RESET_COURSE_PROGRESS':
      const updatedState = { ...state };
      delete updatedState[action.payload.courseId];
      return updatedState;
    
    default:
      return state;
  }
};

// Main LMS Provider
export const LMSProvider = ({ children }) => {
  const [courseData, dispatchCourseData] = useReducer(courseDataReducer, {
    courses: [],
    loading: false,
    error: null,
    lastUpdated: null
  });

  const [userProgress, dispatchUserProgress] = useReducer(userProgressReducer, {});

  const [uiState, setUIState] = useState({
    currentView: 'dashboard',
    sidebarOpen: false,
    notifications: [],
    theme: 'light'
  });

  // Memoized context values
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

export const useUIState = () => {
  const context = useContext(UIStateContext);
  if (!context) {
    throw new Error('useUIState must be used within LMSProvider');
  }
  return context;
};
```

### Custom Hooks for LMS Logic
✅ **Good Example: Reusable Custom Hooks**
```jsx
// hooks/useCourseProgress.js - Business logic abstraction
import { useCallback, useMemo } from 'react';
import { useUserProgress, useCourseData } from '../contexts/LMSContext';

export const useCourseProgress = (courseId) => {
  const { progress, dispatch } = useUserProgress();
  const { courses } = useCourseData();

  const course = useMemo(() => 
    courses.find(c => c.id === courseId),
    [courses, courseId]
  );

  const courseProgress = useMemo(() => 
    progress[courseId] || {},
    [progress, courseId]
  );

  // Calculate completion statistics
  const statistics = useMemo(() => {
    if (!course || !courseProgress.lessons) {
      return {
        totalSections: 0,
        completedSections: 0,
        completionPercentage: 0,
        totalTimeSpent: 0
      };
    }

    let totalSections = 0;
    let completedSections = 0;
    let totalTimeSpent = 0;

    course.lessons.forEach(lesson => {
      lesson.sections.forEach(section => {
        totalSections++;
        const sectionProgress = courseProgress.lessons[lesson.id]?.sections?.[section.id];
        if (sectionProgress?.completed) {
          completedSections++;
        }
        totalTimeSpent += sectionProgress?.timeSpent || 0;
      });
    });

    return {
      totalSections,
      completedSections,
      completionPercentage: totalSections > 0 ? Math.round((completedSections / totalSections) * 100) : 0,
      totalTimeSpent
    };
  }, [course, courseProgress]);

  // Update section progress
  const updateSectionProgress = useCallback(async (lessonId, sectionId, completed, timeSpent = 0) => {
    dispatch({
      type: 'UPDATE_SECTION_PROGRESS',
      payload: {
        courseId,
        lessonId,
        sectionId,
        completed,
        timeSpent
      }
    });

    // Persist to backend
    try {
      await fetch('/api/progress', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          courseId,
          lessonId,
          sectionId,
          completed,
          timeSpent
        })
      });
    } catch (error) {
      console.error('Failed to save progress:', error);
      // Could implement retry logic or show user notification
    }
  }, [courseId, dispatch]);

  // Get next incomplete section
  const getNextSection = useCallback(() => {
    if (!course || !course.lessons) return null;

    for (const lesson of course.lessons) {
      for (const section of lesson.sections) {
        const sectionProgress = courseProgress.lessons?.[lesson.id]?.sections?.[section.id];
        if (!sectionProgress?.completed) {
          return {
            courseId,
            lessonId: lesson.id,
            sectionId: section.id,
            section,
            lesson
          };
        }
      }
    }
    return null; // Course is complete
  }, [course, courseProgress, courseId]);

  return {
    statistics,
    updateSectionProgress,
    getNextSection,
    courseProgress,
    course
  };
};
```

### Local State Management Patterns
✅ **Good Example: Effective useState and useReducer Patterns**
```jsx
// components/LessonView.jsx - Local state for UI interactions
import { useState, useReducer, useCallback, useEffect } from 'react';
import { useCourseProgress } from '../hooks/useCourseProgress';

// Reducer for complex local state
const lessonViewReducer = (state, action) => {
  switch (action.type) {
    case 'SET_CURRENT_SECTION':
      return {
        ...state,
        currentSectionIndex: action.payload,
        isTransitioning: false
      };
    
    case 'START_TRANSITION':
      return {
        ...state,
        isTransitioning: true
      };
    
    case 'TOGGLE_SIDEBAR':
      return {
        ...state,
        sidebarOpen: !state.sidebarOpen
      };
    
    case 'SET_FULLSCREEN':
      return {
        ...state,
        isFullscreen: action.payload
      };
    
    default:
      return state;
  }
};

const LessonView = ({ courseId, lessonId }) => {
  const { course, updateSectionProgress } = useCourseProgress(courseId);
  
  // Complex local state with useReducer
  const [viewState, dispatch] = useReducer(lessonViewReducer, {
    currentSectionIndex: 0,
    isTransitioning: false,
    sidebarOpen: true,
    isFullscreen: false
  });

  // Simple local state with useState
  const [timeSpent, setTimeSpent] = useState(0);
  const [startTime, setStartTime] = useState(Date.now());

  const lesson = course?.lessons.find(l => l.id === lessonId);
  const currentSection = lesson?.sections[viewState.currentSectionIndex];

  // Navigation handlers
  const goToNextSection = useCallback(() => {
    if (viewState.currentSectionIndex < lesson.sections.length - 1) {
      dispatch({ type: 'START_TRANSITION' });
      
      setTimeout(() => {
        dispatch({ 
          type: 'SET_CURRENT_SECTION', 
          payload: viewState.currentSectionIndex + 1 
        });
      }, 150);
    }
  }, [viewState.currentSectionIndex, lesson]);

  const goToPreviousSection = useCallback(() => {
    if (viewState.currentSectionIndex > 0) {
      dispatch({ type: 'START_TRANSITION' });
      
      setTimeout(() => {
        dispatch({ 
          type: 'SET_CURRENT_SECTION', 
          payload: viewState.currentSectionIndex - 1 
        });
      }, 150);
    }
  }, [viewState.currentSectionIndex]);

  // Mark section complete
  const markSectionComplete = useCallback(async () => {
    const sessionTimeSpent = Date.now() - startTime;
    await updateSectionProgress(lessonId, currentSection.id, true, sessionTimeSpent);
    
    // Auto-advance to next section
    if (viewState.currentSectionIndex < lesson.sections.length - 1) {
      setTimeout(goToNextSection, 1000);
    }
  }, [lessonId, currentSection?.id, startTime, updateSectionProgress, goToNextSection, viewState.currentSectionIndex, lesson]);

  // Time tracking effect
  useEffect(() => {
    const interval = setInterval(() => {
      setTimeSpent(Date.now() - startTime);
    }, 1000);

    return () => clearInterval(interval);
  }, [startTime]);

  // Reset timer when section changes
  useEffect(() => {
    setStartTime(Date.now());
    setTimeSpent(0);
  }, [viewState.currentSectionIndex]);

  if (!lesson || !currentSection) {
    return <div>Lesson not found</div>;
  }

  return (
    <div className={`lesson-view ${viewState.isFullscreen ? 'lesson-view--fullscreen' : ''}`}>
      <LessonSidebar 
        lesson={lesson}
        currentSectionIndex={viewState.currentSectionIndex}
        isOpen={viewState.sidebarOpen}
        onSectionSelect={(index) => dispatch({ type: 'SET_CURRENT_SECTION', payload: index })}
        onToggle={() => dispatch({ type: 'TOGGLE_SIDEBAR' })}
      />
      
      <main className="lesson-content">
        <LessonHeader 
          lesson={lesson}
          currentSection={currentSection}
          timeSpent={timeSpent}
          onToggleFullscreen={() => dispatch({ 
            type: 'SET_FULLSCREEN', 
            payload: !viewState.isFullscreen 
          })}
        />
        
        <SectionContent 
          section={currentSection}
          isTransitioning={viewState.isTransitioning}
        />
        
        <LessonNavigation
          canGoPrevious={viewState.currentSectionIndex > 0}
          canGoNext={viewState.currentSectionIndex < lesson.sections.length - 1}
          onPrevious={goToPreviousSection}
          onNext={goToNextSection}
          onMarkComplete={markSectionComplete}
        />
      </main>
    </div>
  );
};

export { LessonView };
```

## Lifecycle and Effects Patterns

### useEffect Best Practices for LMS
✅ **Good Example: Proper Effect Management**
```jsx
// hooks/useCourseData.js - Data fetching with proper cleanup
import { useState, useEffect, useRef, useCallback } from 'react';

export const useCourseData = (courseId) => {
  const [course, setCourse] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const abortControllerRef = useRef(null);

  const fetchCourse = useCallback(async (id) => {
    // Cancel previous request
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }

    // Create new abort controller
    abortControllerRef.current = new AbortController();

    try {
      setLoading(true);
      setError(null);

      const response = await fetch(`/api/courses/${id}`, {
        signal: abortControllerRef.current.signal
      });

      if (!response.ok) {
        throw new Error(`Failed to fetch course: ${response.statusText}`);
      }

      const courseData = await response.json();
      setCourse(courseData);
    } catch (err) {
      // Don't set error if request was aborted
      if (err.name !== 'AbortError') {
        setError(err.message);
        setCourse(null);
      }
    } finally {
      setLoading(false);
    }
  }, []);

  // Effect for fetching course data
  useEffect(() => {
    if (!courseId) {
      setCourse(null);
      setLoading(false);
      return;
    }

    fetchCourse(courseId);

    // Cleanup function
    return () => {
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, [courseId, fetchCourse]);

  // Cleanup on unmount
  useEffect(() => {
    return () => {
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, []);

  return { course, loading, error, refetch: () => fetchCourse(courseId) };
};

// components/CourseDetail.jsx - Using the custom hook
const CourseDetail = ({ courseId }) => {
  const { course, loading, error, refetch } = useCourseData(courseId);
  const [activeTab, setActiveTab] = useState('overview');

  // Effect for updating document title
  useEffect(() => {
    if (course) {
      document.title = `${course.title} - LMS Platform`;
    }

    return () => {
      document.title = 'LMS Platform';
    };
  }, [course]);

  // Effect for tracking page views
  useEffect(() => {
    if (course) {
      // Analytics tracking
      window.analytics?.track('Course Viewed', {
        courseId: course.id,
        courseName: course.title,
        difficulty: course.difficulty
      });
    }
  }, [course]);

  if (loading) return <CourseDetailSkeleton />;
  if (error) return <ErrorMessage message={error} onRetry={refetch} />;
  if (!course) return <NotFound />;

  return (
    <div className="course-detail">
      <CourseHeader course={course} />
      <CourseTabs activeTab={activeTab} onTabChange={setActiveTab} />
      <CourseContent course={course} activeTab={activeTab} />
    </div>
  );
};
```

## Event Handling Patterns

### Optimized Event Handlers for LMS
✅ **Good Example: Efficient Event Handling**
```jsx
// components/CourseList.jsx - Optimized event handling
import { useCallback, useMemo, useState } from 'react';
import { debounce } from '../utils/debounce';

const CourseList = ({ courses, onCourseSelect, onCourseEnroll }) => {
  const [searchTerm, setSearchTerm] = useState('');
  const [selectedDifficulty, setSelectedDifficulty] = useState('all');

  // Memoized filtered courses
  const filteredCourses = useMemo(() => {
    return courses.filter(course => {
      const matchesSearch = course.title.toLowerCase().includes(searchTerm.toLowerCase()) ||
                           course.description.toLowerCase().includes(searchTerm.toLowerCase());
      const matchesDifficulty = selectedDifficulty === 'all' || course.difficulty === selectedDifficulty;
      
      return matchesSearch && matchesDifficulty;
    });
  }, [courses, searchTerm, selectedDifficulty]);

  // Debounced search handler
  const debouncedSearch = useMemo(
    () => debounce((term) => setSearchTerm(term), 300),
    []
  );

  const handleSearchChange = useCallback((event) => {
    debouncedSearch(event.target.value);
  }, [debouncedSearch]);

  // Memoized event handlers to prevent child re-renders
  const handleCourseClick = useCallback((courseId) => {
    onCourseSelect(courseId);
  }, [onCourseSelect]);

  const handleEnrollClick = useCallback((event, courseId) => {
    event.stopPropagation(); // Prevent course selection
    onCourseEnroll(courseId);
  }, [onCourseEnroll]);

  const handleDifficultyFilter = useCallback((difficulty) => {
    setSelectedDifficulty(difficulty);
  }, []);

  return (
    <div className="course-list">
      <CourseListHeader 
        onSearchChange={handleSearchChange}
        selectedDifficulty={selectedDifficulty}
        onDifficultyChange={handleDifficultyFilter}
      />
      
      <div className="course-grid">
        {filteredCourses.map(course => (
          <CourseCard
            key={course.id}
            course={course}
            onClick={() => handleCourseClick(course.id)}
            onEnroll={(event) => handleEnrollClick(event, course.id)}
          />
        ))}
      </div>
      
      {filteredCourses.length === 0 && (
        <EmptyState message="No courses found matching your criteria" />
      )}
    </div>
  );
};
```

## Error Boundaries and Error Handling

### Comprehensive Error Boundary Implementation
✅ **Good Example: LMS-Specific Error Boundary**
```jsx
// components/ErrorBoundary.jsx - Production-ready error handling
import React from 'react';

class LMSErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      hasError: false,
      error: null,
      errorInfo: null,
      errorId: null
    };
  }

  static getDerivedStateFromError(error) {
    return {
      hasError: true,
      errorId: Date.now().toString()
    };
  }

  componentDidCatch(error, errorInfo) {
    this.setState({
      error,
      errorInfo
    });

    // Log error to monitoring service
    this.logError(error, errorInfo);
  }

  logError = (error, errorInfo) => {
    const errorData = {
      message: error.message,
      stack: error.stack,
      componentStack: errorInfo.componentStack,
      url: window.location.href,
      timestamp: new Date().toISOString(),
      userAgent: navigator.userAgent,
      errorId: this.state.errorId
    };

    // Send to error monitoring service
    if (window.errorLogger) {
      window.errorLogger.captureException(error, {
        extra: errorData,
        tags: {
          component: 'LMSErrorBoundary',
          section: this.props.section || 'unknown'
        }
      });
    }

    console.error('LMS Error Boundary caught an error:', errorData);
  };

  handleRetry = () => {
    this.setState({
      hasError: false,
      error: null,
      errorInfo: null,
      errorId: null
    });
  };

  render() {
    if (this.state.hasError) {
      const { fallback: Fallback, section = 'application' } = this.props;

      if (Fallback) {
        return (
          <Fallback
            error={this.state.error}
            errorInfo={this.state.errorInfo}
            onRetry={this.handleRetry}
            errorId={this.state.errorId}
          />
        );
      }

      return (
        <LMSErrorFallback
          error={this.state.error}
          section={section}
          onRetry={this.handleRetry}
          errorId={this.state.errorId}
        />
      );
    }

    return this.props.children;
  }
}

// Default error fallback component
const LMSErrorFallback = ({ error, section, onRetry, errorId }) => (
  <div className="error-boundary">
    <div className="error-boundary__content">
      <h2>Something went wrong in the {section}</h2>
      <p>We apologize for the inconvenience. The error has been logged and our team will investigate.</p>
      
      <details className="error-boundary__details">
        <summary>Error Details (for support)</summary>
        <pre>
          Error ID: {errorId}
          {error && error.toString()}
        </pre>
      </details>
      
      <div className="error-boundary__actions">
        <button onClick={onRetry} className="btn btn--primary">
          Try Again
        </button>
        <button onClick={() => window.location.reload()} className="btn btn--secondary">
          Reload Page
        </button>
      </div>
    </div>
  </div>
);

export { LMSErrorBoundary };

// Usage in app sections
const App = () => (
  <LMSErrorBoundary section="application">
    <Router>
      <Routes>
        <Route path="/courses" element={
          <LMSErrorBoundary section="course-listing">
            <CourseList />
          </LMSErrorBoundary>
        } />
        <Route path="/courses/:id" element={
          <LMSErrorBoundary section="course-detail">
            <CourseDetail />
          </LMSErrorBoundary>
        } />
      </Routes>
    </Router>
  </LMSErrorBoundary>
);
```

## Key React Patterns Takeaways

### Critical React Development Patterns ✅

1. **Component Architecture**
   - Use composition over inheritance for reusable components
   - Separate container (logic) and presentational (UI) components
   - Implement proper component hierarchy with clear data flow
   - Use React.memo and useMemo for performance optimization

2. **State Management**
   - Choose appropriate state management based on scope (local vs global)
   - Use multiple contexts to separate concerns and prevent unnecessary re-renders
   - Implement proper reducers for complex state transitions
   - Create custom hooks to encapsulate business logic

3. **Effect Management**
   - Always provide cleanup functions in useEffect
   - Use AbortController for cancelling fetch requests
   - Implement proper dependency arrays to prevent infinite loops
   - Separate effects by concern and responsibility

4. **Event Handling**
   - Use useCallback to memoize event handlers
   - Implement debouncing for search and input handlers
   - Prevent unnecessary re-renders with proper event handler memoization
   - Handle async operations safely with loading states

5. **Error Handling**
   - Implement error boundaries at appropriate levels
   - Provide meaningful error messages and recovery options
   - Log errors to monitoring services with proper context
   - Create fallback UI components for different error scenarios

### React Anti-Patterns to Avoid ❌

1. **Massive Components** - Creating components with too many responsibilities
2. **Prop Drilling** - Passing props through multiple levels without context
3. **Missing Dependencies** - Incorrect useEffect dependency arrays causing bugs
4. **Direct State Mutation** - Modifying state objects directly instead of using immutable updates
5. **Unmemoized Expensive Operations** - Not optimizing performance-critical calculations

React fundamentals are essential for building maintainable, performant LMS applications that can handle complex educational workflows and large datasets efficiently. 