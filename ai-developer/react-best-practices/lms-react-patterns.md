# React Best Practices for LMS Development

## Table of Contents
1. [React Fundamentals for Educational Platforms](#react-fundamentals-for-educational-platforms)
2. [State Management in LMS Applications](#state-management-in-lms-applications)
3. [Custom Hooks for Educational Features](#custom-hooks-for-educational-features)
4. [Performance Optimization for Learning Platforms](#performance-optimization-for-learning-platforms)
5. [LMS-Specific Component Patterns](#lms-specific-component-patterns)
6. [Testing Educational Components](#testing-educational-components)
7. [Accessibility in Educational Interfaces](#accessibility-in-educational-interfaces)

## React Fundamentals for Educational Platforms

### 1. Component Architecture for Learning Systems

**Lesson Container Pattern:**
```jsx
// ✅ Good: Structured lesson container with clear data flow
const LessonContainer = ({ courseId, lessonId }) => {
  const [lesson, setLesson] = useState(null);
  const [progress, setProgress] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const loadLessonData = async () => {
      try {
        const [lessonData, progressData] = await Promise.all([
          fetchLesson(courseId, lessonId),
          fetchProgress(courseId, lessonId)
        ]);
        setLesson(lessonData);
        setProgress(progressData);
      } catch (error) {
        console.error('Failed to load lesson:', error);
      } finally {
        setLoading(false);
      }
    };

    loadLessonData();
  }, [courseId, lessonId]);

  if (loading) return <LessonSkeleton />;
  if (!lesson) return <LessonNotFound />;

  return (
    <div className="lesson-container">
      <LessonHeader lesson={lesson} progress={progress} />
      <LessonContent lesson={lesson} onProgress={updateProgress} />
      <LessonNavigation courseId={courseId} currentLesson={lessonId} />
    </div>
  );
};

// ❌ Poor: Monolithic component with mixed concerns
const BadLessonPage = () => {
  // Hundreds of lines mixing UI, data fetching, and business logic
};
```

**Educational Component Composition:**
```jsx
// ✅ Good: Composable educational components
const CourseViewer = ({ course }) => (
  <CourseProvider course={course}>
    <CourseHeader />
    <CourseProgress />
    <LessonGrid />
    <CourseFooter />
  </CourseProvider>
);

const LessonGrid = () => {
  const { lessons, progress } = useCourse();
  
  return (
    <div className="lesson-grid">
      {lessons.map(lesson => (
        <LessonCard 
          key={lesson.id}
          lesson={lesson}
          progress={progress[lesson.id]}
          isLocked={!isLessonUnlocked(lesson, progress)}
        />
      ))}
    </div>
  );
};
```

### 2. Props and Data Flow Patterns

**Educational Data Structures:**
```jsx
// ✅ Good: Well-defined educational props
interface LessonProps {
  lesson: {
    id: string;
    title: string;
    description: string;
    sections: Section[];
    estimatedDuration: number;
    difficulty: 'beginner' | 'intermediate' | 'advanced';
  };
  progress: {
    completedSections: string[];
    timeSpent: number;
    lastAccessed: Date;
    completionPercentage: number;
  };
  onSectionComplete: (sectionId: string) => void;
  onLessonExit: () => void;
}

const LessonViewer: React.FC<LessonProps> = ({
  lesson,
  progress,
  onSectionComplete,
  onLessonExit
}) => {
  // Implementation
};
```

**Progress Tracking Props Pattern:**
```jsx
// ✅ Good: Consistent progress tracking interface
interface ProgressProps {
  completed: number;
  total: number;
  showPercentage?: boolean;
  showTime?: boolean;
  onProgressUpdate?: (progress: number) => void;
}

const ProgressBar: React.FC<ProgressProps> = ({
  completed,
  total,
  showPercentage = true,
  showTime = false,
  onProgressUpdate
}) => {
  const percentage = Math.round((completed / total) * 100);
  
  useEffect(() => {
    onProgressUpdate?.(percentage);
  }, [percentage, onProgressUpdate]);

  return (
    <div className="progress-container">
      <div className="progress-bar">
        <div 
          className="progress-fill" 
          style={{ width: `${percentage}%` }}
        />
      </div>
      {showPercentage && <span>{percentage}%</span>}
      {showTime && <TimeDisplay />}
    </div>
  );
};
```

## State Management in LMS Applications

### 1. Course State with Context API

**Course Context Provider:**
```jsx
// ✅ Good: Comprehensive course state management
const CourseContext = createContext();

export const CourseProvider = ({ children, courseId }) => {
  const [courseState, setCourseState] = useReducer(courseReducer, {
    course: null,
    lessons: [],
    progress: {},
    enrollment: null,
    loading: true,
    error: null
  });

  const courseActions = useMemo(() => ({
    updateProgress: (lessonId, sectionId) => {
      setCourseState({
        type: 'UPDATE_PROGRESS',
        payload: { lessonId, sectionId }
      });
    },
    
    markLessonComplete: (lessonId) => {
      setCourseState({
        type: 'COMPLETE_LESSON',
        payload: { lessonId }
      });
    },
    
    enrollInCourse: async () => {
      try {
        setCourseState({ type: 'SET_LOADING', payload: true });
        const enrollment = await enrollStudent(courseId);
        setCourseState({ 
          type: 'SET_ENROLLMENT', 
          payload: enrollment 
        });
      } catch (error) {
        setCourseState({ 
          type: 'SET_ERROR', 
          payload: error.message 
        });
      }
    }
  }), [courseId]);

  return (
    <CourseContext.Provider value={{ ...courseState, ...courseActions }}>
      {children}
    </CourseContext.Provider>
  );
};

// Course reducer for complex state updates
const courseReducer = (state, action) => {
  switch (action.type) {
    case 'UPDATE_PROGRESS':
      const { lessonId, sectionId } = action.payload;
      return {
        ...state,
        progress: {
          ...state.progress,
          [lessonId]: {
            ...state.progress[lessonId],
            completedSections: [
              ...(state.progress[lessonId]?.completedSections || []),
              sectionId
            ]
          }
        }
      };
    
    case 'COMPLETE_LESSON':
      return {
        ...state,
        progress: {
          ...state.progress,
          [action.payload.lessonId]: {
            ...state.progress[action.payload.lessonId],
            completed: true,
            completedAt: new Date()
          }
        }
      };
    
    default:
      return state;
  }
};
```

### 2. Global Learning State with Zustand

**Learning Progress Store:**
```jsx
// ✅ Good: Zustand store for learning progress
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

export const useLearningStore = create(
  persist(
    (set, get) => ({
      // State
      courses: {},
      currentCourse: null,
      learningStreak: 0,
      totalTimeSpent: 0,
      achievements: [],
      
      // Actions
      updateCourseProgress: (courseId, lessonId, progress) => 
        set((state) => ({
          courses: {
            ...state.courses,
            [courseId]: {
              ...state.courses[courseId],
              lessons: {
                ...state.courses[courseId]?.lessons,
                [lessonId]: progress
              }
            }
          }
        })),
      
      addTimeSpent: (minutes) =>
        set((state) => ({
          totalTimeSpent: state.totalTimeSpent + minutes
        })),
      
      updateStreak: () =>
        set((state) => ({
          learningStreak: state.learningStreak + 1
        })),
      
      unlockAchievement: (achievement) =>
        set((state) => ({
          achievements: [...state.achievements, achievement]
        })),
      
      // Computed values
      getCourseProgress: (courseId) => {
        const course = get().courses[courseId];
        if (!course) return 0;
        
        const lessons = Object.values(course.lessons);
        const completed = lessons.filter(l => l.completed).length;
        return (completed / lessons.length) * 100;
      }
    }),
    {
      name: 'learning-progress',
      getStorage: () => localStorage,
    }
  )
);
```

### 3. Server State with React Query

**Educational Data Queries:**
```jsx
// ✅ Good: React Query for server state management
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// Course queries
export const useCourse = (courseId) => {
  return useQuery({
    queryKey: ['course', courseId],
    queryFn: () => fetchCourse(courseId),
    staleTime: 5 * 60 * 1000, // 5 minutes
    cacheTime: 10 * 60 * 1000, // 10 minutes
  });
};

export const useLessons = (courseId) => {
  return useQuery({
    queryKey: ['lessons', courseId],
    queryFn: () => fetchLessons(courseId),
    enabled: !!courseId,
  });
};

// Progress mutations
export const useUpdateProgress = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: ({ courseId, lessonId, progress }) =>
      updateLessonProgress(courseId, lessonId, progress),
    
    onSuccess: (data, variables) => {
      // Optimistic updates
      queryClient.setQueryData(
        ['progress', variables.courseId],
        (old) => ({ ...old, ...data })
      );
      
      // Invalidate related queries
      queryClient.invalidateQueries(['course', variables.courseId]);
    },
    
    onError: (error) => {
      console.error('Failed to update progress:', error);
      // Show user-friendly error
    }
  });
};

// Usage in components
const LessonProgress = ({ courseId, lessonId }) => {
  const updateProgress = useUpdateProgress();
  
  const handleSectionComplete = (sectionId) => {
    updateProgress.mutate({
      courseId,
      lessonId,
      progress: { completedSections: [sectionId] }
    });
  };

  return (
    // Component JSX
  );
};
```

## Custom Hooks for Educational Features

### 1. Progress Tracking Hook

```jsx
// ✅ Good: Custom hook for progress tracking
export const useProgressTracking = (courseId, lessonId) => {
  const [progress, setProgress] = useState({
    currentSection: 0,
    completedSections: [],
    timeSpent: 0,
    lastAccessed: null
  });
  
  const [timeStart, setTimeStart] = useState(null);
  
  // Start timer when lesson begins
  useEffect(() => {
    setTimeStart(Date.now());
    return () => {
      if (timeStart) {
        const sessionTime = Math.floor((Date.now() - timeStart) / 1000 / 60);
        updateTimeSpent(sessionTime);
      }
    };
  }, [lessonId]);
  
  const markSectionComplete = useCallback((sectionId) => {
    setProgress(prev => ({
      ...prev,
      completedSections: [...prev.completedSections, sectionId],
      currentSection: prev.currentSection + 1
    }));
    
    // Persist to server
    updateProgressOnServer(courseId, lessonId, sectionId);
  }, [courseId, lessonId]);
  
  const updateTimeSpent = useCallback((minutes) => {
    setProgress(prev => ({
      ...prev,
      timeSpent: prev.timeSpent + minutes
    }));
  }, []);
  
  const getCompletionPercentage = useCallback(() => {
    return Math.round(
      (progress.completedSections.length / totalSections) * 100
    );
  }, [progress.completedSections.length]);
  
  return {
    progress,
    markSectionComplete,
    getCompletionPercentage,
    isComplete: progress.completedSections.length === totalSections
  };
};
```

### 2. Video Player Hook

```jsx
// ✅ Good: Educational video player hook
export const useVideoPlayer = (videoUrl, onProgress) => {
  const videoRef = useRef(null);
  const [playerState, setPlayerState] = useState({
    playing: false,
    currentTime: 0,
    duration: 0,
    muted: false,
    volume: 1,
    playbackRate: 1,
    buffered: 0
  });
  
  const [watchedSegments, setWatchedSegments] = useState([]);
  
  // Track watched segments for progress
  useEffect(() => {
    const interval = setInterval(() => {
      if (playerState.playing && videoRef.current) {
        const currentTime = videoRef.current.currentTime;
        const newSegment = Math.floor(currentTime / 10); // 10-second segments
        
        setWatchedSegments(prev => {
          if (!prev.includes(newSegment)) {
            const updated = [...prev, newSegment];
            onProgress?.(updated.length / Math.ceil(playerState.duration / 10));
            return updated;
          }
          return prev;
        });
      }
    }, 1000);
    
    return () => clearInterval(interval);
  }, [playerState.playing, playerState.duration, onProgress]);
  
  const controls = {
    play: () => {
      videoRef.current?.play();
      setPlayerState(prev => ({ ...prev, playing: true }));
    },
    
    pause: () => {
      videoRef.current?.pause();
      setPlayerState(prev => ({ ...prev, playing: false }));
    },
    
    seek: (time) => {
      if (videoRef.current) {
        videoRef.current.currentTime = time;
        setPlayerState(prev => ({ ...prev, currentTime: time }));
      }
    },
    
    setPlaybackRate: (rate) => {
      if (videoRef.current) {
        videoRef.current.playbackRate = rate;
        setPlayerState(prev => ({ ...prev, playbackRate: rate }));
      }
    },
    
    toggleMute: () => {
      if (videoRef.current) {
        videoRef.current.muted = !videoRef.current.muted;
        setPlayerState(prev => ({ ...prev, muted: !prev.muted }));
      }
    }
  };
  
  return {
    videoRef,
    playerState,
    controls,
    watchedSegments,
    completionPercentage: watchedSegments.length / Math.ceil(playerState.duration / 10)
  };
};
```

### 3. Quiz State Hook

```jsx
// ✅ Good: Quiz management hook
export const useQuizState = (quizData) => {
  const [currentQuestion, setCurrentQuestion] = useState(0);
  const [answers, setAnswers] = useState({});
  const [timeRemaining, setTimeRemaining] = useState(quizData.timeLimit);
  const [quizCompleted, setQuizCompleted] = useState(false);
  
  // Timer logic
  useEffect(() => {
    if (timeRemaining > 0 && !quizCompleted) {
      const timer = setTimeout(() => {
        setTimeRemaining(prev => prev - 1);
      }, 1000);
      return () => clearTimeout(timer);
    } else if (timeRemaining === 0) {
      submitQuiz();
    }
  }, [timeRemaining, quizCompleted]);
  
  const selectAnswer = useCallback((questionId, answer) => {
    setAnswers(prev => ({
      ...prev,
      [questionId]: answer
    }));
  }, []);
  
  const nextQuestion = useCallback(() => {
    if (currentQuestion < quizData.questions.length - 1) {
      setCurrentQuestion(prev => prev + 1);
    }
  }, [currentQuestion, quizData.questions.length]);
  
  const previousQuestion = useCallback(() => {
    if (currentQuestion > 0) {
      setCurrentQuestion(prev => prev - 1);
    }
  }, [currentQuestion]);
  
  const submitQuiz = useCallback(() => {
    setQuizCompleted(true);
    
    // Calculate score
    const score = quizData.questions.reduce((total, question) => {
      return answers[question.id] === question.correctAnswer ? total + 1 : total;
    }, 0);
    
    // Submit to server
    submitQuizResults({
      quizId: quizData.id,
      answers,
      score,
      timeSpent: quizData.timeLimit - timeRemaining
    });
  }, [answers, quizData, timeRemaining]);
  
  const getQuizSummary = useCallback(() => {
    const totalQuestions = quizData.questions.length;
    const answeredQuestions = Object.keys(answers).length;
    const score = quizData.questions.reduce((total, question) => {
      return answers[question.id] === question.correctAnswer ? total + 1 : total;
    }, 0);
    
    return {
      totalQuestions,
      answeredQuestions,
      score,
      percentage: Math.round((score / totalQuestions) * 100),
      timeSpent: quizData.timeLimit - timeRemaining
    };
  }, [answers, quizData, timeRemaining]);
  
  return {
    currentQuestion,
    answers,
    timeRemaining,
    quizCompleted,
    selectAnswer,
    nextQuestion,
    previousQuestion,
    submitQuiz,
    getQuizSummary,
    canSubmit: Object.keys(answers).length === quizData.questions.length
  };
};
```

## Performance Optimization for Learning Platforms

### 1. Component Optimization

**Memoization for Educational Components:**
```jsx
// ✅ Good: Proper memoization for lesson components
const LessonCard = React.memo(({ lesson, progress, onEnroll }) => {
  return (
    <div className="lesson-card">
      <h3>{lesson.title}</h3>
      <ProgressBar 
        completed={progress.completed} 
        total={progress.total} 
      />
      <button onClick={() => onEnroll(lesson.id)}>
        Start Lesson
      </button>
    </div>
  );
}, (prevProps, nextProps) => {
  // Custom comparison for complex progress objects
  return (
    prevProps.lesson.id === nextProps.lesson.id &&
    prevProps.progress.completed === nextProps.progress.completed &&
    prevProps.progress.total === nextProps.progress.total
  );
});

// ✅ Good: Optimized course list with virtualization
import { FixedSizeList as List } from 'react-window';

const CourseList = ({ courses }) => {
  const CourseItem = useCallback(({ index, style }) => (
    <div style={style}>
      <CourseCard course={courses[index]} />
    </div>
  ), [courses]);

  return (
    <List
      height={600}
      itemCount={courses.length}
      itemSize={200}
      itemData={courses}
    >
      {CourseItem}
    </List>
  );
};
```

### 2. Data Loading Optimization

**Lazy Loading and Code Splitting:**
```jsx
// ✅ Good: Lazy loading for heavy educational components
const VideoPlayer = lazy(() => import('./VideoPlayer'));
const InteractiveQuiz = lazy(() => import('./InteractiveQuiz'));
const CodeEditor = lazy(() => import('./CodeEditor'));

const LessonContent = ({ lesson }) => {
  const renderContent = (section) => {
    switch (section.type) {
      case 'video':
        return (
          <Suspense fallback={<VideoSkeleton />}>
            <VideoPlayer src={section.videoUrl} />
          </Suspense>
        );
      case 'quiz':
        return (
          <Suspense fallback={<QuizSkeleton />}>
            <InteractiveQuiz questions={section.questions} />
          </Suspense>
        );
      case 'code':
        return (
          <Suspense fallback={<CodeSkeleton />}>
            <CodeEditor initialCode={section.code} />
          </Suspense>
        );
      default:
        return <TextContent content={section.content} />;
    }
  };

  return (
    <div className="lesson-content">
      {lesson.sections.map((section, index) => (
        <section key={section.id} className="lesson-section">
          {renderContent(section)}
        </section>
      ))}
    </div>
  );
};
```

**Intelligent Prefetching:**
```jsx
// ✅ Good: Prefetch next lesson content
export const usePrefetchLesson = (courseId, currentLessonIndex) => {
  const queryClient = useQueryClient();
  
  useEffect(() => {
    // Prefetch next lesson when user is 80% through current lesson
    const prefetchNext = () => {
      const nextLessonIndex = currentLessonIndex + 1;
      queryClient.prefetchQuery({
        queryKey: ['lesson', courseId, nextLessonIndex],
        queryFn: () => fetchLesson(courseId, nextLessonIndex),
        staleTime: 5 * 60 * 1000,
      });
    };
    
    // Trigger prefetch based on progress
    const progressListener = (progress) => {
      if (progress > 0.8) {
        prefetchNext();
      }
    };
    
    // Add progress listener
    return () => {
      // Cleanup
    };
  }, [courseId, currentLessonIndex, queryClient]);
};
```

### 3. Image and Media Optimization

**Progressive Image Loading:**
```jsx
// ✅ Good: Progressive image loading for course thumbnails
const CourseImage = ({ src, alt, className }) => {
  const [imageLoaded, setImageLoaded] = useState(false);
  const [imageSrc, setImageSrc] = useState(null);
  
  useEffect(() => {
    const img = new Image();
    img.onload = () => {
      setImageSrc(src);
      setImageLoaded(true);
    };
    img.src = src;
  }, [src]);
  
  return (
    <div className={`image-container ${className}`}>
      {!imageLoaded && (
        <div className="image-skeleton">
          <div className="pulse-animation" />
        </div>
      )}
      {imageSrc && (
        <img
          src={imageSrc}
          alt={alt}
          className={`course-image ${imageLoaded ? 'loaded' : 'loading'}`}
          loading="lazy"
        />
      )}
    </div>
  );
};
```

## LMS-Specific Component Patterns

### 1. Learning Path Components

**Adaptive Learning Path:**
```jsx
// ✅ Good: Adaptive learning path component
const LearningPath = ({ pathId, studentLevel }) => {
  const { data: path, isLoading } = useLearningPath(pathId);
  const { progress } = useStudentProgress(pathId);
  
  const getAdaptiveContent = useCallback((lesson, studentData) => {
    // Adaptive logic based on student performance
    if (studentData.strugglingTopics.includes(lesson.topic)) {
      return {
        ...lesson,
        additionalResources: true,
        practiceExercises: lesson.practiceExercises * 1.5
      };
    }
    
    if (studentData.masteredTopics.includes(lesson.topic)) {
      return {
        ...lesson,
        skipBasics: true,
        advancedChallenges: true
      };
    }
    
    return lesson;
  }, []);
  
  if (isLoading) return <PathSkeleton />;
  
  return (
    <div className="learning-path">
      <PathHeader path={path} progress={progress} />
      <div className="path-timeline">
        {path.lessons.map((lesson, index) => {
          const adaptiveLesson = getAdaptiveContent(lesson, progress);
          const isUnlocked = index === 0 || progress.completedLessons.includes(path.lessons[index - 1].id);
          
          return (
            <LessonNode
              key={lesson.id}
              lesson={adaptiveLesson}
              isUnlocked={isUnlocked}
              isCompleted={progress.completedLessons.includes(lesson.id)}
              position={index}
            />
          );
        })}
      </div>
    </div>
  );
};
```

### 2. Assessment Components

**Multi-Type Assessment Handler:**
```jsx
// ✅ Good: Flexible assessment component
const AssessmentRenderer = ({ assessment, onSubmit }) => {
  const [responses, setResponses] = useState({});
  const [timeTracking, setTimeTracking] = useState({});
  
  const renderQuestion = (question) => {
    const commonProps = {
      question,
      value: responses[question.id],
      onChange: (value) => setResponses(prev => ({
        ...prev,
        [question.id]: value
      })),
      onTimeUpdate: (time) => setTimeTracking(prev => ({
        ...prev,
        [question.id]: time
      }))
    };
    
    switch (question.type) {
      case 'multiple-choice':
        return <MultipleChoiceQuestion {...commonProps} />;
      case 'true-false':
        return <TrueFalseQuestion {...commonProps} />;
      case 'short-answer':
        return <ShortAnswerQuestion {...commonProps} />;
      case 'essay':
        return <EssayQuestion {...commonProps} />;
      case 'code':
        return <CodeQuestion {...commonProps} />;
      case 'drag-drop':
        return <DragDropQuestion {...commonProps} />;
      default:
        return <div>Unsupported question type</div>;
    }
  };
  
  const handleSubmit = () => {
    const submissionData = {
      assessmentId: assessment.id,
      responses,
      timeSpent: timeTracking,
      submittedAt: new Date(),
      metadata: {
        userAgent: navigator.userAgent,
        viewport: `${window.innerWidth}x${window.innerHeight}`
      }
    };
    
    onSubmit(submissionData);
  };
  
  return (
    <div className="assessment-container">
      <AssessmentHeader assessment={assessment} />
      <div className="questions-container">
        {assessment.questions.map((question, index) => (
          <div key={question.id} className="question-wrapper">
            <div className="question-number">
              Question {index + 1} of {assessment.questions.length}
            </div>
            {renderQuestion(question)}
          </div>
        ))}
      </div>
      <AssessmentFooter 
        onSubmit={handleSubmit}
        canSubmit={Object.keys(responses).length === assessment.questions.length}
      />
    </div>
  );
};
```

### 3. Progress Visualization Components

**Comprehensive Progress Dashboard:**
```jsx
// ✅ Good: Multi-level progress visualization
const ProgressDashboard = ({ studentId, timeRange = 'week' }) => {
  const { data: progressData } = useProgressData(studentId, timeRange);
  
  const ProgressMetrics = () => (
    <div className="progress-metrics">
      <MetricCard
        title="Courses Completed"
        value={progressData.completedCourses}
        total={progressData.enrolledCourses}
        icon="📚"
      />
      <MetricCard
        title="Learning Streak"
        value={progressData.learningStreak}
        unit="days"
        icon="🔥"
      />
      <MetricCard
        title="Time Studied"
        value={progressData.totalTimeMinutes}
        unit="minutes"
        icon="⏱️"
      />
      <MetricCard
        title="Skill Points"
        value={progressData.skillPoints}
        icon="⭐"
      />
    </div>
  );
  
  const LearningChart = () => (
    <div className="learning-chart">
      <h3>Learning Activity</h3>
      <ResponsiveContainer width="100%" height={300}>
        <LineChart data={progressData.dailyActivity}>
          <XAxis dataKey="date" />
          <YAxis />
          <CartesianGrid strokeDasharray="3 3" />
          <Tooltip />
          <Legend />
          <Line 
            type="monotone" 
            dataKey="timeSpent" 
            stroke="#8884d8" 
            name="Time Spent (min)"
          />
          <Line 
            type="monotone" 
            dataKey="lessonsCompleted" 
            stroke="#82ca9d" 
            name="Lessons Completed"
          />
        </LineChart>
      </ResponsiveContainer>
    </div>
  );
  
  const SkillRadar = () => (
    <div className="skill-radar">
      <h3>Skill Development</h3>
      <ResponsiveContainer width="100%" height={300}>
        <RadarChart data={progressData.skillLevels}>
          <PolarGrid />
          <PolarAngleAxis dataKey="skill" />
          <PolarRadiusAxis angle={90} domain={[0, 100]} />
          <Radar
            name="Current Level"
            dataKey="current"
            stroke="#8884d8"
            fill="#8884d8"
            fillOpacity={0.6}
          />
          <Radar
            name="Target Level"
            dataKey="target"
            stroke="#82ca9d"
            fill="transparent"
            strokeDasharray="5 5"
          />
          <Legend />
        </RadarChart>
      </ResponsiveContainer>
    </div>
  );
  
  return (
    <div className="progress-dashboard">
      <ProgressMetrics />
      <div className="charts-container">
        <LearningChart />
        <SkillRadar />
      </div>
      <AchievementsList achievements={progressData.recentAchievements} />
    </div>
  );
};
```

## Testing Educational Components

### 1. Unit Testing Learning Components

**Testing Progress Components:**
```jsx
// ✅ Good: Comprehensive testing for educational components
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { LessonViewer } from '../LessonViewer';

describe('LessonViewer', () => {
  let queryClient;
  
  beforeEach(() => {
    queryClient = new QueryClient({
      defaultOptions: {
        queries: { retry: false },
        mutations: { retry: false },
      },
    });
  });
  
  const renderWithProviders = (component) => {
    return render(
      <QueryClientProvider client={queryClient}>
        {component}
      </QueryClientProvider>
    );
  };
  
  test('should track progress correctly when sections are completed', async () => {
    const mockLesson = {
      id: 'lesson-1',
      title: 'Introduction to React',
      sections: [
        { id: 'section-1', title: 'What is React?', type: 'text' },
        { id: 'section-2', title: 'JSX Basics', type: 'text' }
      ]
    };
    
    const mockProgress = {
      completedSections: [],
      currentSection: 0,
      timeSpent: 0
    };
    
    const onProgressUpdate = jest.fn();
    
    renderWithProviders(
      <LessonViewer 
        lesson={mockLesson}
        progress={mockProgress}
        onProgressUpdate={onProgressUpdate}
      />
    );
    
    // Complete first section
    fireEvent.click(screen.getByText('Complete Section'));
    
    await waitFor(() => {
      expect(onProgressUpdate).toHaveBeenCalledWith({
        completedSections: ['section-1'],
        currentSection: 1,
        timeSpent: expect.any(Number)
      });
    });
    
    // Verify progress bar updates
    expect(screen.getByText('50%')).toBeInTheDocument();
  });
  
  test('should handle quiz submission correctly', async () => {
    const mockQuiz = {
      id: 'quiz-1',
      questions: [
        {
          id: 'q1',
          text: 'What is JSX?',
          type: 'multiple-choice',
          options: ['A markup language', 'A programming language'],
          correctAnswer: 'A markup language'
        }
      ]
    };
    
    const onQuizSubmit = jest.fn();
    
    renderWithProviders(
      <QuizComponent quiz={mockQuiz} onSubmit={onQuizSubmit} />
    );
    
    // Select answer
    fireEvent.click(screen.getByText('A markup language'));
    
    // Submit quiz
    fireEvent.click(screen.getByText('Submit Quiz'));
    
    await waitFor(() => {
      expect(onQuizSubmit).toHaveBeenCalledWith({
        quizId: 'quiz-1',
        answers: { q1: 'A markup language' },
        score: 1,
        timeSpent: expect.any(Number)
      });
    });
  });
});
```

### 2. Integration Testing Learning Flows

**End-to-End Learning Flow Testing:**
```jsx
// ✅ Good: Integration testing for complete learning flows
describe('Complete Learning Flow', () => {
  test('should allow student to complete entire lesson', async () => {
    // Mock API responses
    server.use(
      rest.get('/api/courses/:courseId/lessons/:lessonId', (req, res, ctx) => {
        return res(ctx.json(mockLessonData));
      }),
      rest.post('/api/progress/update', (req, res, ctx) => {
        return res(ctx.json({ success: true }));
      })
    );
    
    render(<App />);
    
    // Navigate to lesson
    fireEvent.click(screen.getByText('Start Lesson'));
    
    // Complete each section
    for (let i = 0; i < mockLessonData.sections.length; i++) {
      await waitFor(() => {
        expect(screen.getByText(`Section ${i + 1}`)).toBeInTheDocument();
      });
      
      fireEvent.click(screen.getByText('Complete Section'));
      
      if (i < mockLessonData.sections.length - 1) {
        fireEvent.click(screen.getByText('Next Section'));
      }
    }
    
    // Verify lesson completion
    await waitFor(() => {
      expect(screen.getByText('Lesson Complete!')).toBeInTheDocument();
    });
    
    // Verify progress was saved
    expect(screen.getByText('100% Complete')).toBeInTheDocument();
  });
});
```

## Accessibility in Educational Interfaces

### 1. Screen Reader Support

**Accessible Progress Indicators:**
```jsx
// ✅ Good: Accessible progress components
const AccessibleProgressBar = ({ 
  completed, 
  total, 
  label = "Lesson Progress",
  announceChanges = true 
}) => {
  const percentage = Math.round((completed / total) * 100);
  const [previousPercentage, setPreviousPercentage] = useState(percentage);
  
  // Announce progress changes to screen readers
  useEffect(() => {
    if (announceChanges && percentage !== previousPercentage) {
      const message = `Progress updated: ${percentage}% complete`;
      announceToScreenReader(message);
      setPreviousPercentage(percentage);
    }
  }, [percentage, previousPercentage, announceChanges]);
  
  return (
    <div className="progress-container">
      <label id="progress-label" className="progress-label">
        {label}
      </label>
      <div
        role="progressbar"
        aria-labelledby="progress-label"
        aria-valuenow={percentage}
        aria-valuemin={0}
        aria-valuemax={100}
        aria-valuetext={`${completed} of ${total} sections completed`}
        className="progress-bar"
      >
        <div 
          className="progress-fill" 
          style={{ width: `${percentage}%` }}
        />
      </div>
      <span className="progress-text" aria-live="polite">
        {completed} of {total} completed ({percentage}%)
      </span>
    </div>
  );
};

// Screen reader announcement utility
const announceToScreenReader = (message) => {
  const announcement = document.createElement('div');
  announcement.setAttribute('aria-live', 'assertive');
  announcement.setAttribute('aria-atomic', 'true');
  announcement.className = 'sr-only';
  announcement.textContent = message;
  
  document.body.appendChild(announcement);
  
  setTimeout(() => {
    document.body.removeChild(announcement);
  }, 1000);
};
```

### 2. Keyboard Navigation

**Accessible Lesson Navigation:**
```jsx
// ✅ Good: Full keyboard navigation support
const KeyboardAccessibleLessonNav = ({ lessons, currentLessonId, onLessonChange }) => {
  const [focusedIndex, setFocusedIndex] = useState(0);
  const navRefs = useRef([]);
  
  const handleKeyDown = (event, index) => {
    switch (event.key) {
      case 'ArrowDown':
        event.preventDefault();
        const nextIndex = Math.min(index + 1, lessons.length - 1);
        setFocusedIndex(nextIndex);
        navRefs.current[nextIndex]?.focus();
        break;
        
      case 'ArrowUp':
        event.preventDefault();
        const prevIndex = Math.max(index - 1, 0);
        setFocusedIndex(prevIndex);
        navRefs.current[prevIndex]?.focus();
        break;
        
      case 'Enter':
      case ' ':
        event.preventDefault();
        onLessonChange(lessons[index].id);
        break;
        
      case 'Home':
        event.preventDefault();
        setFocusedIndex(0);
        navRefs.current[0]?.focus();
        break;
        
      case 'End':
        event.preventDefault();
        const lastIndex = lessons.length - 1;
        setFocusedIndex(lastIndex);
        navRefs.current[lastIndex]?.focus();
        break;
    }
  };
  
  return (
    <nav aria-label="Lesson Navigation" className="lesson-nav">
      <ul role="list">
        {lessons.map((lesson, index) => (
          <li key={lesson.id} role="listitem">
            <button
              ref={el => navRefs.current[index] = el}
              className={`lesson-nav-item ${lesson.id === currentLessonId ? 'current' : ''}`}
              aria-current={lesson.id === currentLessonId ? 'page' : undefined}
              aria-describedby={`lesson-${lesson.id}-description`}
              onKeyDown={(e) => handleKeyDown(e, index)}
              onClick={() => onLessonChange(lesson.id)}
            >
              <span className="lesson-number">Lesson {index + 1}</span>
              <span className="lesson-title">{lesson.title}</span>
              {lesson.completed && (
                <span className="completion-indicator" aria-label="Completed">
                  ✓
                </span>
              )}
            </button>
            <div 
              id={`lesson-${lesson.id}-description`}
              className="lesson-description sr-only"
            >
              {lesson.description}. 
              Duration: {lesson.duration} minutes. 
              {lesson.completed ? 'Completed' : 'Not completed'}.
            </div>
          </li>
        ))}
      </ul>
    </nav>
  );
};
```

### 3. Visual Accessibility

**High Contrast and Responsive Text:**
```jsx
// ✅ Good: Accessible visual design system
const AccessibleLearningCard = ({ 
  course, 
  progress, 
  difficulty,
  onEnroll 
}) => {
  const difficultyColors = {
    beginner: { bg: '#e8f5e8', border: '#4caf50', text: '#2e7d32' },
    intermediate: { bg: '#fff3e0', border: '#ff9800', text: '#ef6c00' },
    advanced: { bg: '#ffebee', border: '#f44336', text: '#c62828' }
  };
  
  const colors = difficultyColors[difficulty] || difficultyColors.beginner;
  
  return (
    <div 
      className="learning-card"
      style={{
        '--difficulty-bg': colors.bg,
        '--difficulty-border': colors.border,
        '--difficulty-text': colors.text
      }}
    >
      <div className="card-header">
        <h3 className="course-title">
          {course.title}
          <span 
            className="difficulty-badge"
            aria-label={`Difficulty level: ${difficulty}`}
          >
            {difficulty}
          </span>
        </h3>
      </div>
      
      <div className="card-content">
        <p className="course-description">
          {course.description}
        </p>
        
        <AccessibleProgressBar
          completed={progress.completed}
          total={progress.total}
          label={`${course.title} progress`}
        />
        
        <div className="course-metadata">
          <span className="duration">
            <span aria-label="Estimated duration">⏱️</span>
            {course.estimatedHours} hours
          </span>
          <span className="lessons">
            <span aria-label="Number of lessons">📚</span>
            {course.lessonCount} lessons
          </span>
        </div>
      </div>
      
      <div className="card-actions">
        <button
          className="enroll-button"
          onClick={() => onEnroll(course.id)}
          aria-describedby={`course-${course.id}-description`}
        >
          {progress.enrolled ? 'Continue Learning' : 'Start Course'}
        </button>
      </div>
      
      <div 
        id={`course-${course.id}-description`}
        className="sr-only"
      >
        {course.title}. {difficulty} level course. 
        {course.estimatedHours} hours long with {course.lessonCount} lessons.
        {progress.enrolled 
          ? `${progress.completed} of ${progress.total} lessons completed.`
          : 'Not yet enrolled.'
        }
      </div>
    </div>
  );
};
```

---

## Summary

This comprehensive React guide covers the essential patterns and practices for building robust, performant, and accessible LMS applications. Key takeaways:

**✅ Do:**
- Use proper state management patterns for educational data
- Implement custom hooks for reusable learning functionality
- Optimize performance with memoization and lazy loading
- Design accessible interfaces with proper ARIA labels
- Test educational workflows comprehensively
- Follow component composition principles

**❌ Don't:**
- Create monolithic components mixing UI and business logic
- Ignore performance implications of large course datasets
- Skip accessibility considerations for educational interfaces
- Forget to test learning progression and assessment flows
- Neglect proper error handling for educational content
- Use inadequate state management for complex learning data

This guide serves as the foundation for building professional-grade LMS applications with React. 