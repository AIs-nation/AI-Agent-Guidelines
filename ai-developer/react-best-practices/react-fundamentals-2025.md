# React Best Practices 2025 - LMS Project Focus

## Core React Fundamentals for LMS Development

### Functional Components with Hooks
**Modern React development uses functional components exclusively**

✅ **Good Example: Course Detail Component**
```jsx
// Proper functional component with hooks
function CourseDetailPage({ courseId }) {
  const [course, setCourse] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchCourse = async () => {
      try {
        setLoading(true);
        const response = await api.getCourse(courseId);
        setCourse(response.data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchCourse();
  }, [courseId]);

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!course) return <NotFound />;

  return (
    <div className="course-detail">
      <CourseHeader course={course} />
      <LessonGrid lessons={course.lessons} />
    </div>
  );
}
```

❌ **Bad Example: Class Component (Outdated)**
```jsx
// Avoid class components - they're legacy
class CourseDetailPage extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      course: null,
      loading: true
    };
  }
  // This pattern is outdated for new development
}
```

### State Management Patterns

✅ **Good Example: Custom Hook for Course Progress**
```jsx
// Custom hook for progress tracking
function useProgressTracking(courseId) {
  const [progress, setProgress] = useState(null);
  const [updating, setUpdating] = useState(false);

  const updateProgress = useCallback(async (lessonId, sectionId) => {
    setUpdating(true);
    try {
      const newProgress = await api.updateProgress(courseId, lessonId, sectionId);
      setProgress(newProgress);
    } catch (error) {
      console.error('Progress update failed:', error);
    } finally {
      setUpdating(false);
    }
  }, [courseId]);

  useEffect(() => {
    const fetchProgress = async () => {
      try {
        const response = await api.getProgress(courseId);
        setProgress(response.data);
      } catch (error) {
        console.error('Failed to fetch progress:', error);
      }
    };

    if (courseId) {
      fetchProgress();
    }
  }, [courseId]);

  return { progress, updateProgress, updating };
}
```

❌ **Bad Example: Prop Drilling**
```jsx
// Avoid passing props through multiple levels
function App() {
  const [user, setUser] = useState(null);
  return (
    <Dashboard user={user}>
      <CourseList user={user}>
        <CourseCard user={user}>
          <ProgressBar user={user} /> {/* Deep prop drilling */}
        </CourseCard>
      </CourseList>
    </Dashboard>
  );
}
```

### Context API for Global State

✅ **Good Example: Course Context**
```jsx
// Create context for course-related state
const CourseContext = createContext();

export function CourseProvider({ children }) {
  const [courses, setCourses] = useState([]);
  const [selectedCourse, setSelectedCourse] = useState(null);
  const [progress, setProgress] = useState({});

  const value = {
    courses,
    setCourses,
    selectedCourse,
    setSelectedCourse,
    progress,
    setProgress
  };

  return (
    <CourseContext.Provider value={value}>
      {children}
    </CourseContext.Provider>
  );
}

// Custom hook to use course context
export function useCourse() {
  const context = useContext(CourseContext);
  if (!context) {
    throw new Error('useCourse must be used within CourseProvider');
  }
  return context;
}
```

### Performance Optimization

✅ **Good Example: Memoization for Expensive Operations**
```jsx
function LessonGrid({ lessons, progress }) {
  // Memoize expensive calculations
  const sortedLessons = useMemo(() => {
    return lessons
      .map(lesson => ({
        ...lesson,
        completionRate: calculateCompletionRate(lesson, progress)
      }))
      .sort((a, b) => a.order - b.order);
  }, [lessons, progress]);

  // Memoize callback to prevent unnecessary re-renders
  const handleLessonClick = useCallback((lessonId) => {
    navigate(`/lessons/${lessonId}`);
  }, [navigate]);

  return (
    <div className="lesson-grid">
      {sortedLessons.map(lesson => (
        <LessonCard 
          key={lesson.id}
          lesson={lesson}
          onClick={handleLessonClick}
        />
      ))}
    </div>
  );
}

// Memoize component to prevent unnecessary re-renders
const LessonCard = memo(({ lesson, onClick }) => {
  return (
    <div 
      className="lesson-card"
      onClick={() => onClick(lesson.id)}
    >
      <h3>{lesson.title}</h3>
      <ProgressBar progress={lesson.completionRate} />
    </div>
  );
});
```

❌ **Bad Example: Performance Issues**
```jsx
function LessonGrid({ lessons, progress }) {
  // Expensive calculation runs on every render
  const sortedLessons = lessons
    .map(lesson => ({
      ...lesson,
      completionRate: calculateExpensiveRate(lesson, progress) // Runs every render
    }))
    .sort((a, b) => a.order - b.order);

  return (
    <div className="lesson-grid">
      {sortedLessons.map(lesson => (
        <LessonCard 
          key={lesson.id}
          lesson={lesson}
          onClick={(lessonId) => navigate(`/lessons/${lessonId}`)} // New function every render
        />
      ))}
    </div>
  );
}
```

### Error Boundaries

✅ **Good Example: Course Error Boundary**
```jsx
class CourseErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Course error:', error, errorInfo);
    // Log to error tracking service
    logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-fallback">
          <h2>Something went wrong with this course</h2>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage in course components
function App() {
  return (
    <CourseErrorBoundary>
      <CourseDetailPage />
    </CourseErrorBoundary>
  );
}
```

### Async Operations with Suspense

✅ **Good Example: Suspense for Course Loading**
```jsx
// Suspense-enabled course fetching
const CourseDetail = lazy(() => import('./CourseDetail'));

function App() {
  return (
    <Suspense fallback={<CourseLoadingSkeleton />}>
      <Routes>
        <Route path="/courses/:id" element={<CourseDetail />} />
      </Routes>
    </Suspense>
  );
}

// Loading skeleton component
function CourseLoadingSkeleton() {
  return (
    <div className="course-skeleton">
      <div className="skeleton-header" />
      <div className="skeleton-content">
        {Array.from({ length: 6 }).map((_, i) => (
          <div key={i} className="skeleton-lesson-card" />
        ))}
      </div>
    </div>
  );
}
```

## React 18 Specific Features

### Concurrent Features

✅ **Good Example: Concurrent Rendering for Course Generation**
```jsx
function CourseGenerator() {
  const [isPending, startTransition] = useTransition();
  const [courses, setCourses] = useState([]);
  const [filter, setFilter] = useState('');

  const handleFilterChange = (newFilter) => {
    setFilter(newFilter);
    
    // Mark filtering as non-urgent
    startTransition(() => {
      const filtered = filterCourses(courses, newFilter);
      setCourses(filtered);
    });
  };

  return (
    <div>
      <SearchInput 
        value={filter}
        onChange={handleFilterChange}
        disabled={isPending}
      />
      {isPending && <LoadingSpinner />}
      <CourseList courses={courses} />
    </div>
  );
}
```

### Automatic Batching

✅ **Good Example: Leveraging Automatic Batching**
```jsx
function ProgressTracker({ courseId }) {
  const [progress, setProgress] = useState(0);
  const [status, setStatus] = useState('idle');
  const [lastUpdated, setLastUpdated] = useState(null);

  const updateProgress = async (sectionId) => {
    // React 18 automatically batches these updates
    setStatus('updating');
    setProgress(prev => prev + 1);
    setLastUpdated(new Date());
    
    try {
      await api.updateProgress(courseId, sectionId);
      setStatus('success');
    } catch (error) {
      setStatus('error');
      setProgress(prev => prev - 1); // Rollback
    }
  };

  return (
    <div className="progress-tracker">
      <ProgressBar value={progress} />
      <StatusIndicator status={status} />
      <LastUpdated time={lastUpdated} />
    </div>
  );
}
```

## Testing Best Practices

✅ **Good Example: Testing Course Components**
```jsx
// Course component test
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { CourseDetailPage } from './CourseDetailPage';

// Mock API calls
jest.mock('../api/courses', () => ({
  getCourse: jest.fn(),
  updateProgress: jest.fn()
}));

describe('CourseDetailPage', () => {
  it('displays course content when loaded', async () => {
    const mockCourse = {
      id: '1',
      title: 'React Fundamentals',
      lessons: [
        { id: '1', title: 'Introduction', sections: [] }
      ]
    };

    api.getCourse.mockResolvedValue({ data: mockCourse });

    render(<CourseDetailPage courseId="1" />);

    // Wait for loading to complete
    await waitFor(() => {
      expect(screen.getByText('React Fundamentals')).toBeInTheDocument();
    });

    expect(screen.getByText('Introduction')).toBeInTheDocument();
  });

  it('handles progress updates correctly', async () => {
    const mockCourse = { /* course data */ };
    api.getCourse.mockResolvedValue({ data: mockCourse });
    api.updateProgress.mockResolvedValue({ success: true });

    render(<CourseDetailPage courseId="1" />);

    const completeButton = await screen.findByText('Mark Complete');
    fireEvent.click(completeButton);

    await waitFor(() => {
      expect(api.updateProgress).toHaveBeenCalledWith('1', expect.any(String));
    });
  });
});
```

## LMS-Specific React Patterns

### Course Navigation Component
✅ **Good Example: Reusable Navigation Hook**
```jsx
function useCourseNavigation(courseId) {
  const navigate = useNavigate();
  const { progress } = useCourse();

  const goToLesson = useCallback((lessonId) => {
    navigate(`/courses/${courseId}/lessons/${lessonId}`);
  }, [courseId, navigate]);

  const goToNextSection = useCallback((currentLessonId, currentSectionId) => {
    const nextSection = findNextSection(progress, currentLessonId, currentSectionId);
    if (nextSection) {
      navigate(`/courses/${courseId}/lessons/${nextSection.lessonId}/sections/${nextSection.sectionId}`);
    }
  }, [courseId, navigate, progress]);

  const canAccessLesson = useCallback((lessonId) => {
    return isLessonUnlocked(progress, lessonId);
  }, [progress]);

  return {
    goToLesson,
    goToNextSection,
    canAccessLesson
  };
}
```

### Progress Tracking Components
✅ **Good Example: Compound Component Pattern**
```jsx
// Compound component for progress display
function ProgressDisplay({ children, courseId }) {
  const { progress } = useProgressTracking(courseId);
  
  return (
    <ProgressContext.Provider value={progress}>
      <div className="progress-display">
        {children}
      </div>
    </ProgressContext.Provider>
  );
}

ProgressDisplay.Overall = function({ className }) {
  const progress = useContext(ProgressContext);
  return (
    <div className={className}>
      Overall: {progress.completionPercentage}%
    </div>
  );
};

ProgressDisplay.ByLesson = function({ lessonId }) {
  const progress = useContext(ProgressContext);
  const lessonProgress = progress.lessons[lessonId];
  
  return (
    <div>
      Lesson Progress: {lessonProgress?.completedSections || 0}/{lessonProgress?.totalSections || 0}
    </div>
  );
};

// Usage
function CoursePage({ courseId }) {
  return (
    <ProgressDisplay courseId={courseId}>
      <ProgressDisplay.Overall className="overall-progress" />
      <ProgressDisplay.ByLesson lessonId="lesson-1" />
    </ProgressDisplay>
  );
}
```

## Key Takeaways for LMS Development

1. **Use functional components exclusively** - Better performance and modern React features
2. **Implement custom hooks** - Share logic between components efficiently
3. **Use Context API judiciously** - Only for truly global state (user auth, course selection)
4. **Optimize with React.memo and useMemo** - Critical for course listings and progress tracking
5. **Implement proper error boundaries** - Essential for course content that might fail to load
6. **Test components thoroughly** - Use React Testing Library for user-centric tests
7. **Leverage React 18 features** - Concurrent rendering for better UX during course generation
8. **Create reusable patterns** - Course navigation, progress tracking, content display components

These patterns ensure scalable, maintainable, and performant React code specifically tailored for learning management system requirements. 