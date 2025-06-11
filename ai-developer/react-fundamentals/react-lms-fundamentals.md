# React Fundamentals for LMS Development

## Core React State Management for Educational Platforms

### Educational State Architecture
✅ **Good Example: React State for Learning Management**
```jsx
// hooks/useLearningState.js - Centralized learning state management
import { useState, useEffect, useCallback, useReducer } from 'react';

// Learning state reducer for complex educational workflows
const learningReducer = (state, action) => {
  switch (action.type) {
    case 'SET_CURRENT_COURSE':
      return {
        ...state,
        currentCourse: action.payload,
        currentLesson: null,
        currentSection: null,
        loadingState: 'idle'
      };
    
    case 'SET_CURRENT_LESSON':
      return {
        ...state,
        currentLesson: action.payload,
        currentSection: action.payload.sections?.[0] || null
      };
    
    case 'COMPLETE_SECTION':
      const updatedLesson = {
        ...state.currentLesson,
        sections: state.currentLesson.sections.map(section =>
          section.id === action.sectionId
            ? { ...section, completed: true, progress: 100 }
            : section
        )
      };
      return {
        ...state,
        currentLesson: updatedLesson,
        progress: {
          ...state.progress,
          sectionsCompleted: state.progress.sectionsCompleted + 1
        }
      };
    
    case 'SET_LOADING_STATE':
      return {
        ...state,
        loadingState: action.payload
      };
    
    default:
      return state;
  }
};

// Custom hook for learning management
export const useLearningState = () => {
  const [state, dispatch] = useReducer(learningReducer, {
    currentCourse: null,
    currentLesson: null,
    currentSection: null,
    progress: {
      sectionsCompleted: 0,
      totalSections: 0,
      timeSpent: 0
    },
    loadingState: 'idle'
  });

  // Course navigation with progress validation
  const navigateToCourse = useCallback(async (courseId) => {
    dispatch({ type: 'SET_LOADING_STATE', payload: 'loading' });
    
    try {
      const response = await fetch(`/api/courses/${courseId}`);
      if (!response.ok) throw new Error('Course not found');
      
      const courseData = await response.json();
      dispatch({ type: 'SET_CURRENT_COURSE', payload: courseData });
      dispatch({ type: 'SET_LOADING_STATE', payload: 'success' });
    } catch (error) {
      console.error('Failed to load course:', error);
      dispatch({ type: 'SET_LOADING_STATE', payload: 'error' });
    }
  }, []);

  // Section completion with progress tracking
  const completeSection = useCallback(async (sectionId, timeSpent) => {
    try {
      const response = await fetch('/api/progress/section', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          sectionId,
          courseId: state.currentCourse?.id,
          lessonId: state.currentLesson?.id,
          timeSpent,
          completed: true
        })
      });
      
      if (response.ok) {
        dispatch({ type: 'COMPLETE_SECTION', sectionId });
      }
    } catch (error) {
      console.error('Failed to save progress:', error);
    }
  }, [state.currentCourse, state.currentLesson]);

  return {
    ...state,
    navigateToCourse,
    completeSection,
    dispatch
  };
};
```

❌ **Bad Example: Scattered State Without Educational Context**
```jsx
// ❌ Poor state management - scattered across components
const CourseViewer = () => {
  const [course, setCourse] = useState(null);
  const [lessons, setLessons] = useState([]);
  const [progress, setProgress] = useState({});
  // ... multiple separate state variables
  
  // ❌ No centralized learning logic
  const handleSectionComplete = (sectionId) => {
    // Directly mutating state without proper progress tracking
    setProgress({ ...progress, [sectionId]: true });
  };
  
  // ❌ No error handling or loading states
  const loadCourse = async (id) => {
    const data = await fetch(`/api/courses/${id}`);
    setCourse(data);
  };
};
```

## Educational Component Architecture Patterns

### Course Progression Components
✅ **Good Example: Proper LMS Component Hierarchy**
```jsx
// components/learning/CoursePlayerContainer.jsx
import React, { useState, useEffect } from 'react';
import { useLearningState } from '../../hooks/useLearningState';

const CoursePlayerContainer = ({ courseId }) => {
  const {
    currentCourse,
    currentLesson,
    currentSection,
    loadingState,
    navigateToCourse,
    completeSection
  } = useLearningState();

  useEffect(() => {
    if (courseId) {
      navigateToCourse(courseId);
    }
  }, [courseId, navigateToCourse]);

  if (loadingState === 'loading') {
    return <CourseLoadingState />;
  }

  if (loadingState === 'error') {
    return <CourseErrorState onRetry={() => navigateToCourse(courseId)} />;
  }

  if (!currentCourse) {
    return <NoCourseSelectedState />;
  }

  return (
    <div className="course-player-container">
      <CourseHeader course={currentCourse} />
      <div className="course-content">
        <CourseSidebar 
          lessons={currentCourse.lessons}
          currentLesson={currentLesson}
          onLessonSelect={(lesson) => setCurrentLesson(lesson)}
        />
        <LessonPlayer
          lesson={currentLesson}
          section={currentSection}
          onSectionComplete={completeSection}
        />
      </div>
      <CourseProgressBar course={currentCourse} />
    </div>
  );
};

// components/learning/LessonPlayer.jsx
const LessonPlayer = ({ lesson, section, onSectionComplete }) => {
  const [currentSectionIndex, setCurrentSectionIndex] = useState(0);
  const [sectionStartTime, setSectionStartTime] = useState(Date.now());

  const handleSectionComplete = useCallback(() => {
    const timeSpent = Math.round((Date.now() - sectionStartTime) / 1000);
    onSectionComplete(section.id, timeSpent);
    
    // Navigate to next section
    if (currentSectionIndex < lesson.sections.length - 1) {
      setCurrentSectionIndex(prev => prev + 1);
      setSectionStartTime(Date.now());
    }
  }, [section, onSectionComplete, currentSectionIndex, lesson.sections.length, sectionStartTime]);

  if (!lesson || !section) {
    return <div>Select a lesson to start learning</div>;
  }

  return (
    <div className="lesson-player">
      <SectionHeader section={section} lesson={lesson} />
      <SectionContent 
        content={section.content}
        type={section.type}
      />
      <SectionNavigation
        isLastSection={currentSectionIndex === lesson.sections.length - 1}
        onComplete={handleSectionComplete}
        onNext={() => setCurrentSectionIndex(prev => prev + 1)}
        canProceed={section.completed || section.canSkip}
      />
    </div>
  );
};

export default CoursePlayerContainer;
```

### Educational Form Handling
✅ **Good Example: Course Generation Form with Validation**
```jsx
// components/course-creation/CourseGenerationForm.jsx
import React, { useState } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// Validation schema for course generation
const courseGenerationSchema = z.object({
  title: z.string()
    .min(10, 'Course title must be at least 10 characters')
    .max(100, 'Course title must be less than 100 characters'),
  description: z.string()
    .min(50, 'Course description must be at least 50 characters')
    .max(500, 'Course description must be less than 500 characters'),
  difficulty: z.enum(['beginner', 'intermediate', 'advanced']),
  learningObjectives: z.array(z.string().min(10))
    .min(3, 'At least 3 learning objectives required')
    .max(10, 'Maximum 10 learning objectives allowed'),
  duration: z.number().min(30).max(480), // 30 minutes to 8 hours
  tags: z.array(z.string()).max(10, 'Maximum 10 tags allowed')
});

const CourseGenerationForm = ({ onSubmit }) => {
  const [isGenerating, setIsGenerating] = useState(false);
  const [generationProgress, setGenerationProgress] = useState(0);

  const {
    register,
    handleSubmit,
    formState: { errors, isValid },
    watch,
    setValue
  } = useForm({
    resolver: zodResolver(courseGenerationSchema),
    defaultValues: {
      difficulty: 'intermediate',
      learningObjectives: [''],
      tags: []
    }
  });

  // Dynamic learning objective management
  const learningObjectives = watch('learningObjectives');
  
  const addLearningObjective = () => {
    if (learningObjectives.length < 10) {
      setValue('learningObjectives', [...learningObjectives, '']);
    }
  };

  const removeLearningObjective = (index) => {
    if (learningObjectives.length > 1) {
      const updated = learningObjectives.filter((_, i) => i !== index);
      setValue('learningObjectives', updated);
    }
  };

  // Course generation with progress tracking
  const handleCourseGeneration = async (formData) => {
    setIsGenerating(true);
    setGenerationProgress(0);

    try {
      // Create EventSource for progress tracking
      const eventSource = new EventSource('/api/courses/generate-stream');
      
      eventSource.onmessage = (event) => {
        const data = JSON.parse(event.data);
        
        if (data.type === 'progress') {
          setGenerationProgress(data.progress);
        } else if (data.type === 'complete') {
          setGenerationProgress(100);
          onSubmit(data.course);
          eventSource.close();
        } else if (data.type === 'error') {
          console.error('Generation error:', data.error);
          eventSource.close();
        }
      };

      // Start course generation
      await fetch('/api/courses/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData)
      });

    } catch (error) {
      console.error('Course generation failed:', error);
    } finally {
      setIsGenerating(false);
    }
  };

  return (
    <form onSubmit={handleSubmit(handleCourseGeneration)} className="course-generation-form">
      <div className="form-section">
        <label htmlFor="title">Course Title</label>
        <input
          id="title"
          {...register('title')}
          placeholder="Enter a descriptive course title"
          className={errors.title ? 'error' : ''}
        />
        {errors.title && <span className="error-message">{errors.title.message}</span>}
      </div>

      <div className="form-section">
        <label htmlFor="description">Course Description</label>
        <textarea
          id="description"
          {...register('description')}
          placeholder="Describe what students will learn in this course"
          className={errors.description ? 'error' : ''}
          rows={4}
        />
        {errors.description && <span className="error-message">{errors.description.message}</span>}
      </div>

      <div className="form-section">
        <label htmlFor="difficulty">Difficulty Level</label>
        <select id="difficulty" {...register('difficulty')}>
          <option value="beginner">Beginner</option>
          <option value="intermediate">Intermediate</option>
          <option value="advanced">Advanced</option>
        </select>
      </div>

      <div className="form-section">
        <label>Learning Objectives</label>
        {learningObjectives.map((objective, index) => (
          <div key={index} className="objective-input">
            <input
              {...register(`learningObjectives.${index}`)}
              placeholder={`Learning objective ${index + 1}`}
            />
            {learningObjectives.length > 1 && (
              <button 
                type="button" 
                onClick={() => removeLearningObjective(index)}
                className="remove-objective"
              >
                Remove
              </button>
            )}
          </div>
        ))}
        <button 
          type="button" 
          onClick={addLearningObjective}
          disabled={learningObjectives.length >= 10}
        >
          Add Learning Objective
        </button>
        {errors.learningObjectives && (
          <span className="error-message">{errors.learningObjectives.message}</span>
        )}
      </div>

      {isGenerating && (
        <div className="generation-progress">
          <div className="progress-bar">
            <div 
              className="progress-fill" 
              style={{ width: `${generationProgress}%` }}
            />
          </div>
          <span>Generating course... {generationProgress}%</span>
        </div>
      )}

      <button 
        type="submit" 
        disabled={!isValid || isGenerating}
        className="generate-button"
      >
        {isGenerating ? 'Generating...' : 'Generate Course'}
      </button>
    </form>
  );
};

export default CourseGenerationForm;
```

## Performance Optimization for Educational Content

### Efficient Course Data Loading
✅ **Good Example: Optimized Course Loading with Caching**
```jsx
// hooks/useCourseData.js - Optimized course data management
import { useState, useEffect, useMemo } from 'react';
import { useQuery } from '@tanstack/react-query';

const useCourseData = (courseId) => {
  // Query with caching and background updates
  const {
    data: courseData,
    isLoading,
    error,
    refetch
  } = useQuery({
    queryKey: ['course', courseId],
    queryFn: async () => {
      const response = await fetch(`/api/courses/${courseId}`);
      if (!response.ok) throw new Error('Failed to load course');
      return response.json();
    },
    staleTime: 5 * 60 * 1000, // 5 minutes
    cacheTime: 10 * 60 * 1000, // 10 minutes
    enabled: !!courseId
  });

  // Memoized course structure for performance
  const courseStructure = useMemo(() => {
    if (!courseData) return null;

    return {
      ...courseData,
      totalLessons: courseData.lessons?.length || 0,
      totalSections: courseData.lessons?.reduce((sum, lesson) => 
        sum + (lesson.sections?.length || 0), 0) || 0,
      estimatedDuration: courseData.lessons?.reduce((sum, lesson) => 
        sum + (lesson.estimatedDuration || 0), 0) || 0,
      lessonsMap: new Map(courseData.lessons?.map(lesson => [lesson.id, lesson]) || [])
    };
  }, [courseData]);

  return {
    course: courseStructure,
    isLoading,
    error,
    refetch
  };
};

// components/CourseList.jsx - Virtualized course listing
import { FixedSizeList as List } from 'react-window';

const CourseList = ({ courses, onCourseSelect }) => {
  const CourseItem = ({ index, style }) => {
    const course = courses[index];
    
    return (
      <div style={style} className="course-item">
        <CourseCard 
          course={course}
          onClick={() => onCourseSelect(course.id)}
        />
      </div>
    );
  };

  return (
    <List
      height={600}
      itemCount={courses.length}
      itemSize={200}
      className="course-list"
    >
      {CourseItem}
    </List>
  );
};
```

❌ **Bad Example: Inefficient Data Loading**
```jsx
// ❌ Poor performance - loading all data at once
const CourseViewer = ({ courseId }) => {
  const [courseData, setCourseData] = useState(null);
  
  useEffect(() => {
    // ❌ No caching, loads every time
    fetch(`/api/courses/${courseId}`)
      .then(res => res.json())
      .then(data => setCourseData(data));
  }, [courseId]);

  // ❌ No memoization - recalculates on every render
  const totalSections = courseData?.lessons?.reduce((sum, lesson) => 
    sum + lesson.sections.length, 0);
    
  // ❌ Renders all lessons at once regardless of visibility
  return (
    <div>
      {courseData?.lessons?.map(lesson => (
        <LessonComponent key={lesson.id} lesson={lesson} />
      ))}
    </div>
  );
};
```

## Educational Event Handling Patterns

### Progress Tracking Events
✅ **Good Example: Comprehensive Progress Event Handling**
```jsx
// hooks/useProgressTracking.js
import { useCallback, useRef } from 'react';

const useProgressTracking = (courseId, lessonId) => {
  const startTimeRef = useRef(Date.now());
  const progressQueueRef = useRef([]);

  // Batch progress updates for performance
  const batchUpdateProgress = useCallback(async () => {
    if (progressQueueRef.current.length === 0) return;

    const progressUpdates = [...progressQueueRef.current];
    progressQueueRef.current = [];

    try {
      await fetch('/api/progress/batch', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          courseId,
          lessonId,
          updates: progressUpdates
        })
      });
    } catch (error) {
      console.error('Failed to update progress:', error);
      // Re-queue failed updates
      progressQueueRef.current.unshift(...progressUpdates);
    }
  }, [courseId, lessonId]);

  // Track section completion with time measurement
  const trackSectionCompletion = useCallback((sectionId) => {
    const timeSpent = Date.now() - startTimeRef.current;
    
    progressQueueRef.current.push({
      type: 'section_completed',
      sectionId,
      timeSpent,
      timestamp: new Date().toISOString()
    });

    // Batch update after short delay
    setTimeout(batchUpdateProgress, 1000);
  }, [batchUpdateProgress]);

  // Track learning interactions
  const trackInteraction = useCallback((interactionType, data = {}) => {
    progressQueueRef.current.push({
      type: 'interaction',
      interactionType,
      data,
      timestamp: new Date().toISOString()
    });
  }, []);

  return {
    trackSectionCompletion,
    trackInteraction,
    batchUpdateProgress
  };
};
```

## React Error Handling for Educational Platforms

### Comprehensive Error Boundaries
✅ **Good Example: Educational Platform Error Handling**
```jsx
// components/error/LearningErrorBoundary.jsx
import React from 'react';

class LearningErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { 
      hasError: false, 
      error: null,
      errorType: null
    };
  }

  static getDerivedStateFromError(error) {
    // Categorize error types for educational context
    let errorType = 'general';
    
    if (error.message.includes('course')) {
      errorType = 'course_loading';
    } else if (error.message.includes('progress')) {
      errorType = 'progress_saving';
    } else if (error.message.includes('network')) {
      errorType = 'network';
    }

    return { 
      hasError: true, 
      error,
      errorType
    };
  }

  componentDidCatch(error, errorInfo) {
    // Log error with educational context
    console.error('Learning Error:', error, errorInfo);
    
    // Send error to monitoring service
    if (window.analytics) {
      window.analytics.track('Learning Error', {
        error: error.message,
        errorType: this.state.errorType,
        component: this.props.componentName,
        courseId: this.props.courseId,
        lessonId: this.props.lessonId
      });
    }
  }

  render() {
    if (this.state.hasError) {
      return (
        <LearningErrorFallback 
          errorType={this.state.errorType}
          error={this.state.error}
          onRetry={() => this.setState({ hasError: false })}
          courseId={this.props.courseId}
        />
      );
    }

    return this.props.children;
  }
}

// Error fallback with educational context
const LearningErrorFallback = ({ errorType, error, onRetry, courseId }) => {
  const getErrorMessage = () => {
    switch (errorType) {
      case 'course_loading':
        return {
          title: 'Course Loading Error',
          message: 'We encountered an issue loading this course. Your progress has been saved.',
          action: 'Try reloading the course'
        };
      case 'progress_saving':
        return {
          title: 'Progress Saving Error',
          message: 'Your progress might not have been saved. Please check your connection.',
          action: 'Continue learning - we\'ll retry automatically'
        };
      case 'network':
        return {
          title: 'Connection Issue',
          message: 'Please check your internet connection and try again.',
          action: 'Retry connection'
        };
      default:
        return {
          title: 'Something went wrong',
          message: 'We encountered an unexpected error. Your progress is safe.',
          action: 'Continue learning'
        };
    }
  };

  const errorInfo = getErrorMessage();

  return (
    <div className="learning-error-boundary">
      <div className="error-content">
        <h2>{errorInfo.title}</h2>
        <p>{errorInfo.message}</p>
        <div className="error-actions">
          <button onClick={onRetry} className="retry-button">
            {errorInfo.action}
          </button>
          {courseId && (
            <a href={`/courses/${courseId}`} className="course-link">
              Back to Course Overview
            </a>
          )}
        </div>
      </div>
    </div>
  );
};

export default LearningErrorBoundary;
```

## Key React LMS Development Takeaways

### Critical React Patterns for Educational Platforms ✅

1. **Centralized Learning State Management**
   - Use `useReducer` for complex educational workflows
   - Implement custom hooks for learning progress tracking
   - Maintain course, lesson, and section state coherently
   - Handle loading states and error recovery gracefully

2. **Educational Component Architecture**
   - Design hierarchical components reflecting learning structure
   - Implement proper data flow between course/lesson/section components
   - Use compound component patterns for flexible course layouts
   - Create reusable educational UI components

3. **Performance Optimization for Content**
   - Implement virtual scrolling for large course lists
   - Use React Query for caching course data
   - Memoize expensive calculations (progress, duration)
   - Lazy load lesson content and media assets

4. **Educational Form Handling**
   - Use validation schemas appropriate for educational content
   - Implement dynamic form fields for learning objectives
   - Handle complex course generation workflows
   - Provide real-time feedback during content creation

5. **Progress Tracking and Events**
   - Batch progress updates for performance
   - Track time-based learning analytics
   - Handle offline scenarios with queue synchronization
   - Implement comprehensive learning event logging

### React LMS Anti-Patterns to Avoid ❌

1. **Scattered State Management** - Managing course/lesson/progress state across multiple disconnected components
2. **Inefficient Data Loading** - Loading all course content at once without caching or optimization
3. **Poor Error Handling** - Generic error messages without educational context or recovery options
4. **Synchronous Progress Updates** - Blocking UI with individual progress API calls instead of batching
5. **Missing Validation** - Accepting educational content without proper validation or structure checking

React development for educational platforms requires specialized attention to learning workflows, progress tracking, content optimization, and user experience patterns while maintaining performance and accessibility standards for effective learning environments. 