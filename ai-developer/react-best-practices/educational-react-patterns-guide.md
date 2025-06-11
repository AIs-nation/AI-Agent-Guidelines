# Educational React Patterns Guide: Best Practices for LMS Development

## Table of Contents
1. [Educational Component Architecture](#educational-component-architecture)
2. [Learning State Management](#learning-state-management)
3. [Educational Interaction Patterns](#educational-interaction-patterns)
4. [Accessibility-First Educational Design](#accessibility-first-educational-design)
5. [Performance Optimization for Learning](#performance-optimization-for-learning)
6. [Educational Data Visualization](#educational-data-visualization)
7. [Mobile-First Educational Responsive Design](#mobile-first-educational-responsive-design)
8. [Testing Educational Components](#testing-educational-components)

## Educational Component Architecture

### 1. Learning-Focused Component Structure

**✅ Good: Educational Component Hierarchy**
```jsx
// Educational React component architecture optimized for Learning Management Systems
// Designed with educational user experience and learning workflow prioritization

import React, { useState, useEffect, useCallback, useMemo } from 'react';
import { useEducationalContext, useLearningProgress, useAccessibility } from '@/hooks/educational';
import { EducationalAnalytics, LearningTracker } from '@/services/educational';

// Educational Course Component with learning-focused architecture
const EducationalCourse = ({ courseId, studentId, educationalSettings }) => {
  // Educational state management
  const {
    course,
    lessons,
    learningProgress,
    educationalMetrics,
    isLoading,
    error
  } = useLearningProgress(courseId, studentId);

  // Educational accessibility features
  const {
    accessibilityFeatures,
    screenReaderAnnouncements,
    keyboardNavigation,
    highContrastMode
  } = useAccessibility(educationalSettings.accessibility);

  // Educational interaction tracking
  const [educationalInteractions, setEducationalInteractions] = useState({
    sectionStartTime: null,
    engagementScore: 0,
    learningVelocity: 0,
    difficultyRating: null
  });

  // Educational progress tracking
  const trackEducationalProgress = useCallback(async (progressData) => {
    try {
      await LearningTracker.updateProgress({
        studentId,
        courseId,
        progressData: {
          ...progressData,
          interactionMetrics: educationalInteractions,
          accessibilityUsage: accessibilityFeatures.usage
        }
      });

      // Educational achievement detection
      await EducationalAnalytics.checkAchievements(studentId, progressData);
    } catch (error) {
      console.error('Educational progress tracking failed:', error);
    }
  }, [studentId, courseId, educationalInteractions, accessibilityFeatures]);

  // Educational component structure with learning optimization
  return (
    <div 
      className="educational-course"
      role="main"
      aria-label={`Course: ${course?.title}`}
      data-educational-course={courseId}
    >
      {/* Educational Course Header with learning context */}
      <EducationalCourseHeader
        course={course}
        learningProgress={learningProgress}
        educationalMetrics={educationalMetrics}
        onProgressUpdate={trackEducationalProgress}
        accessibilityFeatures={accessibilityFeatures}
      />

      {/* Educational Lesson Navigation with learning pathway visualization */}
      <EducationalLessonNavigation
        lessons={lessons}
        currentProgress={learningProgress}
        onLessonSelect={(lessonId) => trackEducationalProgress({ 
          type: 'lesson_navigation',
          lessonId,
          timestamp: new Date()
        })}
        keyboardNavigation={keyboardNavigation}
      />

      {/* Educational Content Area with adaptive learning */}
      <EducationalContentArea
        course={course}
        lessons={lessons}
        learningProgress={learningProgress}
        onContentInteraction={(interaction) => {
          setEducationalInteractions(prev => ({
            ...prev,
            ...interaction
          }));
        }}
        accessibilityFeatures={accessibilityFeatures}
        screenReaderAnnouncements={screenReaderAnnouncements}
      />

      {/* Educational Progress Sidebar with learning analytics */}
      <EducationalProgressSidebar
        learningProgress={learningProgress}
        educationalMetrics={educationalMetrics}
        achievements={educationalMetrics?.achievements}
        learningRecommendations={educationalMetrics?.recommendations}
        onProgressVisualization={(viewType) => trackEducationalProgress({
          type: 'progress_visualization',
          viewType,
          timestamp: new Date()
        })}
      />

      {/* Educational Assessment Integration */}
      <EducationalAssessmentPanel
        course={course}
        learningProgress={learningProgress}
        onAssessmentStart={(assessmentId) => trackEducationalProgress({
          type: 'assessment_start',
          assessmentId,
          timestamp: new Date()
        })}
        onAssessmentComplete={(assessmentResult) => trackEducationalProgress({
          type: 'assessment_complete',
          result: assessmentResult,
          timestamp: new Date()
        })}
        accessibilityFeatures={accessibilityFeatures}
      />
    </div>
  );
};

// Educational Course Header with learning context
const EducationalCourseHeader = ({ 
  course, 
  learningProgress, 
  educationalMetrics, 
  onProgressUpdate,
  accessibilityFeatures 
}) => {
  // Educational progress calculation
  const progressPercentage = useMemo(() => {
    if (!learningProgress?.completedSections || !course?.totalSections) return 0;
    return Math.round((learningProgress.completedSections / course.totalSections) * 100);
  }, [learningProgress, course]);

  // Educational milestone detection
  const currentMilestone = useMemo(() => {
    const milestones = [
      { percentage: 25, title: 'Getting Started', icon: '🌱' },
      { percentage: 50, title: 'Half Way There', icon: '📈' },
      { percentage: 75, title: 'Almost Done', icon: '🚀' },
      { percentage: 100, title: 'Course Complete', icon: '🎓' }
    ];
    
    return milestones.reverse().find(m => progressPercentage >= m.percentage) || milestones[0];
  }, [progressPercentage]);

  return (
    <header 
      className="educational-course-header"
      role="banner"
      aria-labelledby="course-title"
    >
      {/* Educational Course Information */}
      <div className="educational-course-info">
        <h1 
          id="course-title"
          className="educational-course-title"
          tabIndex={accessibilityFeatures.enhancedTabIndex ? 0 : -1}
        >
          {course?.title}
        </h1>
        
        <div className="educational-course-metadata">
          <span className="educational-difficulty">
            Difficulty: {course?.difficulty_level}
          </span>
          <span className="educational-duration">
            Duration: {course?.estimated_duration_hours}h
          </span>
          <span className="educational-format">
            Format: {course?.learning_format}
          </span>
        </div>

        {/* Educational Learning Objectives */}
        <div className="educational-learning-objectives">
          <h2>Learning Objectives</h2>
          <ul role="list" aria-label="Course learning objectives">
            {course?.learning_objectives?.map((objective, index) => (
              <li key={index} role="listitem">
                <span className="objective-icon">🎯</span>
                {objective}
              </li>
            ))}
          </ul>
        </div>
      </div>

      {/* Educational Progress Visualization */}
      <div className="educational-progress-section">
        <div className="educational-progress-header">
          <h2>Your Learning Progress</h2>
          <span className="educational-milestone">
            {currentMilestone.icon} {currentMilestone.title}
          </span>
        </div>

        {/* Educational Progress Bar with milestone markers */}
        <div 
          className="educational-progress-container"
          role="progressbar"
          aria-valuenow={progressPercentage}
          aria-valuemin="0"
          aria-valuemax="100"
          aria-label={`Course completion: ${progressPercentage}%`}
        >
          <div 
            className="educational-progress-bar"
            style={{ 
              width: `${progressPercentage}%`,
              backgroundColor: progressPercentage >= 75 ? '#10b981' : 
                             progressPercentage >= 50 ? '#f59e0b' : '#6366f1'
            }}
          />
          
          {/* Educational Milestone Markers */}
          <div className="educational-milestone-markers">
            {[25, 50, 75, 100].map(milestone => (
              <div
                key={milestone}
                className={`educational-milestone-marker ${
                  progressPercentage >= milestone ? 'completed' : 'pending'
                }`}
                style={{ left: `${milestone}%` }}
                title={`${milestone}% milestone`}
              />
            ))}
          </div>
        </div>

        {/* Educational Statistics */}
        <div className="educational-stats">
          <div className="educational-stat">
            <span className="stat-value">{learningProgress?.timeSpentHours || 0}h</span>
            <span className="stat-label">Time Invested</span>
          </div>
          <div className="educational-stat">
            <span className="stat-value">{learningProgress?.completedSections || 0}</span>
            <span className="stat-label">Sections Complete</span>
          </div>
          <div className="educational-stat">
            <span className="stat-value">{educationalMetrics?.averageScore || 0}%</span>
            <span className="stat-label">Average Score</span>
          </div>
          <div className="educational-stat">
            <span className="stat-value">{educationalMetrics?.learningVelocity || 0}</span>
            <span className="stat-label">Learning Velocity</span>
          </div>
        </div>
      </div>
    </header>
  );
};

// Educational Lesson Navigation with learning pathway
const EducationalLessonNavigation = ({ 
  lessons, 
  currentProgress, 
  onLessonSelect,
  keyboardNavigation 
}) => {
  const [selectedLessonId, setSelectedLessonId] = useState(null);
  const [expandedLessons, setExpandedLessons] = useState(new Set());

  // Educational lesson status calculation
  const getLessonStatus = useCallback((lesson) => {
    const completedSections = currentProgress?.lessonProgress?.[lesson.id]?.completedSections || 0;
    const totalSections = lesson.sections?.length || 0;
    
    if (completedSections === 0) return 'not_started';
    if (completedSections === totalSections) return 'completed';
    return 'in_progress';
  }, [currentProgress]);

  // Educational keyboard navigation
  useEffect(() => {
    if (!keyboardNavigation.enabled) return;

    const handleKeyDown = (event) => {
      if (event.key === 'ArrowDown' || event.key === 'ArrowUp') {
        event.preventDefault();
        // Navigate between lessons with arrow keys
        const currentIndex = lessons.findIndex(l => l.id === selectedLessonId);
        const nextIndex = event.key === 'ArrowDown' 
          ? Math.min(currentIndex + 1, lessons.length - 1)
          : Math.max(currentIndex - 1, 0);
        
        if (nextIndex !== currentIndex) {
          setSelectedLessonId(lessons[nextIndex].id);
        }
      } else if (event.key === 'Enter' || event.key === ' ') {
        event.preventDefault();
        if (selectedLessonId) {
          onLessonSelect(selectedLessonId);
        }
      }
    };

    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, [lessons, selectedLessonId, onLessonSelect, keyboardNavigation]);

  return (
    <nav 
      className="educational-lesson-navigation"
      role="navigation"
      aria-label="Course lessons navigation"
    >
      <h2>Course Lessons</h2>
      
      <ul className="educational-lesson-list" role="list">
        {lessons?.map((lesson, index) => {
          const lessonStatus = getLessonStatus(lesson);
          const isExpanded = expandedLessons.has(lesson.id);
          const isUnlocked = index === 0 || getLessonStatus(lessons[index - 1]) === 'completed';

          return (
            <li 
              key={lesson.id}
              className={`educational-lesson-item ${lessonStatus} ${!isUnlocked ? 'locked' : ''}`}
              role="listitem"
            >
              {/* Educational Lesson Header */}
              <button
                className="educational-lesson-header"
                onClick={() => {
                  if (isUnlocked) {
                    setExpandedLessons(prev => {
                      const next = new Set(prev);
                      if (isExpanded) {
                        next.delete(lesson.id);
                      } else {
                        next.add(lesson.id);
                      }
                      return next;
                    });
                    onLessonSelect(lesson.id);
                  }
                }}
                disabled={!isUnlocked}
                aria-expanded={isExpanded}
                aria-controls={`lesson-sections-${lesson.id}`}
                tabIndex={keyboardNavigation.enhancedTabIndex ? 0 : -1}
              >
                <div className="educational-lesson-status">
                  {lessonStatus === 'completed' && <span className="status-icon">✅</span>}
                  {lessonStatus === 'in_progress' && <span className="status-icon">📖</span>}
                  {lessonStatus === 'not_started' && isUnlocked && <span className="status-icon">⭕</span>}
                  {!isUnlocked && <span className="status-icon">🔒</span>}
                </div>

                <div className="educational-lesson-info">
                  <h3 className="educational-lesson-title">{lesson.title}</h3>
                  <p className="educational-lesson-description">{lesson.description}</p>
                  
                  <div className="educational-lesson-metadata">
                    <span className="lesson-duration">{lesson.estimated_duration} min</span>
                    <span className="lesson-sections">{lesson.sections?.length} sections</span>
                    <span className="lesson-difficulty">{lesson.difficulty_level}</span>
                  </div>
                </div>

                <div className="educational-lesson-progress">
                  {lessonStatus !== 'not_started' && (
                    <div className="lesson-progress-circle">
                      <svg viewBox="0 0 36 36" className="circular-chart">
                        <path
                          className="circle-bg"
                          d="M18 2.0845 a 15.9155 15.9155 0 0 1 0 31.831 a 15.9155 15.9155 0 0 1 0 -31.831"
                        />
                        <path
                          className="circle"
                          strokeDasharray={`${(currentProgress?.lessonProgress?.[lesson.id]?.completedSections || 0) / (lesson.sections?.length || 1) * 100}, 100`}
                          d="M18 2.0845 a 15.9155 15.9155 0 0 1 0 31.831 a 15.9155 15.9155 0 0 1 0 -31.831"
                        />
                      </svg>
                    </div>
                  )}
                </div>
              </button>

              {/* Educational Section List */}
              {isExpanded && isUnlocked && (
                <div 
                  id={`lesson-sections-${lesson.id}`}
                  className="educational-section-list"
                  role="group"
                  aria-label={`Sections for ${lesson.title}`}
                >
                  {lesson.sections?.map((section, sectionIndex) => {
                    const sectionCompleted = currentProgress?.sectionProgress?.[section.id]?.completed;
                    const sectionUnlocked = sectionIndex === 0 || 
                      currentProgress?.sectionProgress?.[lesson.sections[sectionIndex - 1].id]?.completed;

                    return (
                      <button
                        key={section.id}
                        className={`educational-section-item ${
                          sectionCompleted ? 'completed' : sectionUnlocked ? 'available' : 'locked'
                        }`}
                        onClick={() => {
                          if (sectionUnlocked) {
                            onLessonSelect(lesson.id, section.id);
                          }
                        }}
                        disabled={!sectionUnlocked}
                        tabIndex={keyboardNavigation.enhancedTabIndex ? 0 : -1}
                      >
                        <div className="educational-section-status">
                          {sectionCompleted && <span>✅</span>}
                          {!sectionCompleted && sectionUnlocked && <span>⭕</span>}
                          {!sectionUnlocked && <span>🔒</span>}
                        </div>

                        <div className="educational-section-info">
                          <h4>{section.title}</h4>
                          <span className="section-type">{section.content_type}</span>
                        </div>

                        <div className="educational-section-metadata">
                          <span className="section-duration">{section.estimated_duration} min</span>
                        </div>
                      </button>
                    );
                  })}
                </div>
              )}
            </li>
          );
        })}
      </ul>
    </nav>
  );
};

export default EducationalCourse;
```

### 2. Learning State Management

**✅ Good: Educational Context and State Patterns**
```jsx
// Educational state management optimized for Learning Management Systems
// Designed with educational data patterns and learning workflow optimization

import React, { createContext, useContext, useReducer, useEffect, useCallback } from 'react';
import { EducationalAPI, LearningAnalytics } from '@/services/educational';

// Educational Learning Context
const EducationalLearningContext = createContext();

// Educational state structure
const initialEducationalState = {
  // Student learning data
  student: {
    id: null,
    profile: null,
    learningPreferences: {
      visualLearner: false,
      auditoryLearner: false,
      kinestheticLearner: false,
      readingWritingLearner: false
    },
    accessibilityNeeds: [],
    learningGoals: []
  },

  // Current course state
  currentCourse: {
    course: null,
    lessons: [],
    currentLesson: null,
    currentSection: null,
    learningProgress: {},
    educationalMetrics: {}
  },

  // Learning progress tracking
  progressTracking: {
    sessionStartTime: null,
    totalTimeSpent: 0,
    sectionsCompleted: 0,
    currentEngagementScore: 0,
    learningVelocity: 0,
    difficultyRatings: {}
  },

  // Educational achievements and milestones
  achievements: {
    earned: [],
    available: [],
    progress: {},
    milestones: []
  },

  // Educational recommendations
  recommendations: {
    nextLessons: [],
    skillGaps: [],
    reviewSuggestions: [],
    learningPaths: []
  },

  // UI state for educational features
  ui: {
    accessibilityMode: 'normal',
    fontSizeMultiplier: 1,
    highContrastMode: false,
    reducedMotion: false,
    screenReaderMode: false
  },

  // Loading and error states
  loading: {
    course: false,
    progress: false,
    recommendations: false
  },
  
  error: {
    course: null,
    progress: null,
    recommendations: null
  }
};

// Educational state reducer
const educationalLearningReducer = (state, action) => {
  switch (action.type) {
    // Student management actions
    case 'SET_STUDENT':
      return {
        ...state,
        student: {
          ...state.student,
          ...action.payload
        }
      };

    case 'UPDATE_LEARNING_PREFERENCES':
      return {
        ...state,
        student: {
          ...state.student,
          learningPreferences: {
            ...state.student.learningPreferences,
            ...action.payload
          }
        }
      };

    // Course management actions
    case 'SET_CURRENT_COURSE':
      return {
        ...state,
        currentCourse: {
          ...state.currentCourse,
          course: action.payload.course,
          lessons: action.payload.lessons || []
        }
      };

    case 'SET_CURRENT_LESSON':
      return {
        ...state,
        currentCourse: {
          ...state.currentCourse,
          currentLesson: action.payload.lesson,
          currentSection: action.payload.section || null
        }
      };

    case 'UPDATE_LEARNING_PROGRESS':
      return {
        ...state,
        currentCourse: {
          ...state.currentCourse,
          learningProgress: {
            ...state.currentCourse.learningProgress,
            [action.payload.key]: action.payload.progress
          }
        }
      };

    // Progress tracking actions
    case 'START_LEARNING_SESSION':
      return {
        ...state,
        progressTracking: {
          ...state.progressTracking,
          sessionStartTime: action.payload.startTime,
          currentEngagementScore: 0
        }
      };

    case 'UPDATE_PROGRESS_METRICS':
      return {
        ...state,
        progressTracking: {
          ...state.progressTracking,
          ...action.payload
        }
      };

    case 'COMPLETE_SECTION':
      return {
        ...state,
        progressTracking: {
          ...state.progressTracking,
          sectionsCompleted: state.progressTracking.sectionsCompleted + 1,
          totalTimeSpent: state.progressTracking.totalTimeSpent + action.payload.timeSpent
        },
        currentCourse: {
          ...state.currentCourse,
          learningProgress: {
            ...state.currentCourse.learningProgress,
            [`section_${action.payload.sectionId}`]: {
              completed: true,
              completedAt: action.payload.completedAt,
              timeSpent: action.payload.timeSpent,
              engagementScore: action.payload.engagementScore
            }
          }
        }
      };

    // Achievement actions
    case 'ADD_ACHIEVEMENT':
      return {
        ...state,
        achievements: {
          ...state.achievements,
          earned: [...state.achievements.earned, action.payload]
        }
      };

    case 'UPDATE_ACHIEVEMENT_PROGRESS':
      return {
        ...state,
        achievements: {
          ...state.achievements,
          progress: {
            ...state.achievements.progress,
            [action.payload.achievementId]: action.payload.progress
          }
        }
      };

    // Recommendation actions
    case 'SET_RECOMMENDATIONS':
      return {
        ...state,
        recommendations: {
          ...state.recommendations,
          ...action.payload
        }
      };

    // UI accessibility actions
    case 'UPDATE_ACCESSIBILITY_SETTINGS':
      return {
        ...state,
        ui: {
          ...state.ui,
          ...action.payload
        }
      };

    // Loading state actions
    case 'SET_LOADING':
      return {
        ...state,
        loading: {
          ...state.loading,
          [action.payload.key]: action.payload.loading
        }
      };

    // Error state actions
    case 'SET_ERROR':
      return {
        ...state,
        error: {
          ...state.error,
          [action.payload.key]: action.payload.error
        }
      };

    default:
      return state;
  }
};

// Educational Learning Provider
export const EducationalLearningProvider = ({ children }) => {
  const [state, dispatch] = useReducer(educationalLearningReducer, initialEducationalState);

  // Educational learning session management
  const startLearningSession = useCallback(async (courseId, lessonId, sectionId) => {
    try {
      dispatch({
        type: 'START_LEARNING_SESSION',
        payload: { startTime: new Date() }
      });

      // Track educational session start
      await LearningAnalytics.trackSessionStart({
        courseId,
        lessonId,
        sectionId,
        studentId: state.student.id,
        sessionMetadata: {
          accessibilityMode: state.ui.accessibilityMode,
          learningPreferences: state.student.learningPreferences
        }
      });
    } catch (error) {
      dispatch({
        type: 'SET_ERROR',
        payload: { key: 'progress', error: error.message }
      });
    }
  }, [state.student.id, state.ui.accessibilityMode, state.student.learningPreferences]);

  // Educational progress update
  const updateEducationalProgress = useCallback(async (progressData) => {
    try {
      dispatch({ type: 'SET_LOADING', payload: { key: 'progress', loading: true } });

      // Update local state
      dispatch({
        type: 'UPDATE_PROGRESS_METRICS',
        payload: progressData
      });

      // Send to educational backend
      const response = await EducationalAPI.updateProgress({
        studentId: state.student.id,
        ...progressData,
        educationalContext: {
          learningPreferences: state.student.learningPreferences,
          accessibilitySettings: state.ui
        }
      });

      // Update educational metrics
      if (response.educationalMetrics) {
        dispatch({
          type: 'SET_CURRENT_COURSE',
          payload: {
            ...state.currentCourse,
            educationalMetrics: response.educationalMetrics
          }
        });
      }

      // Check for new achievements
      if (response.achievements) {
        response.achievements.forEach(achievement => {
          dispatch({
            type: 'ADD_ACHIEVEMENT',
            payload: achievement
          });
        });
      }

      // Update recommendations
      if (response.recommendations) {
        dispatch({
          type: 'SET_RECOMMENDATIONS',
          payload: response.recommendations
        });
      }

    } catch (error) {
      dispatch({
        type: 'SET_ERROR',
        payload: { key: 'progress', error: error.message }
      });
    } finally {
      dispatch({ type: 'SET_LOADING', payload: { key: 'progress', loading: false } });
    }
  }, [state.student.id, state.student.learningPreferences, state.ui, state.currentCourse]);

  // Educational section completion
  const completeEducationalSection = useCallback(async (sectionId, completionData) => {
    try {
      const completionTime = new Date();
      const timeSpent = completionTime - state.progressTracking.sessionStartTime;

      // Update local completion state
      dispatch({
        type: 'COMPLETE_SECTION',
        payload: {
          sectionId,
          completedAt: completionTime,
          timeSpent: Math.floor(timeSpent / 1000), // Convert to seconds
          engagementScore: completionData.engagementScore || 0
        }
      });

      // Educational completion analytics
      await updateEducationalProgress({
        type: 'section_completion',
        sectionId,
        timeSpent: Math.floor(timeSpent / 1000),
        engagementScore: completionData.engagementScore,
        difficultyRating: completionData.difficultyRating,
        learningObjectivesAchieved: completionData.learningObjectivesAchieved,
        educationalFeedback: completionData.feedback
      });

    } catch (error) {
      dispatch({
        type: 'SET_ERROR',
        payload: { key: 'progress', error: error.message }
      });
    }
  }, [state.progressTracking.sessionStartTime, updateEducationalProgress]);

  // Educational accessibility settings
  const updateEducationalAccessibility = useCallback((accessibilitySettings) => {
    dispatch({
      type: 'UPDATE_ACCESSIBILITY_SETTINGS',
      payload: accessibilitySettings
    });

    // Persist accessibility preferences
    localStorage.setItem(
      'educational_accessibility_preferences',
      JSON.stringify(accessibilitySettings)
    );
  }, []);

  // Educational learning preferences
  const updateLearningPreferences = useCallback(async (preferences) => {
    try {
      dispatch({
        type: 'UPDATE_LEARNING_PREFERENCES',
        payload: preferences
      });

      // Update backend with learning preferences
      await EducationalAPI.updateStudentPreferences(state.student.id, preferences);

      // Trigger recommendation refresh
      const recommendations = await EducationalAPI.getRecommendations(state.student.id);
      dispatch({
        type: 'SET_RECOMMENDATIONS',
        payload: recommendations
      });

    } catch (error) {
      dispatch({
        type: 'SET_ERROR',
        payload: { key: 'preferences', error: error.message }
      });
    }
  }, [state.student.id]);

  // Context value with educational methods
  const contextValue = {
    // Educational state
    ...state,

    // Educational learning methods
    startLearningSession,
    updateEducationalProgress,
    completeEducationalSection,
    updateEducationalAccessibility,
    updateLearningPreferences,

    // Educational utility methods
    dispatch
  };

  return (
    <EducationalLearningContext.Provider value={contextValue}>
      {children}
    </EducationalLearningContext.Provider>
  );
};

// Educational learning hook
export const useEducationalLearning = () => {
  const context = useContext(EducationalLearningContext);
  if (!context) {
    throw new Error('useEducationalLearning must be used within EducationalLearningProvider');
  }
  return context;
};

// Educational progress hook
export const useLearningProgress = (courseId, studentId) => {
  const { 
    currentCourse, 
    progressTracking, 
    loading, 
    error,
    dispatch
  } = useEducationalLearning();

  // Load educational course data
  useEffect(() => {
    if (!courseId || !studentId) return;

    const loadEducationalCourse = async () => {
      try {
        dispatch({ type: 'SET_LOADING', payload: { key: 'course', loading: true } });

        const [courseData, progressData] = await Promise.all([
          EducationalAPI.getCourse(courseId),
          EducationalAPI.getStudentProgress(courseId, studentId)
        ]);

        dispatch({
          type: 'SET_CURRENT_COURSE',
          payload: {
            course: courseData.course,
            lessons: courseData.lessons
          }
        });

        dispatch({
          type: 'UPDATE_LEARNING_PROGRESS',
          payload: {
            key: 'course_progress',
            progress: progressData
          }
        });

      } catch (error) {
        dispatch({
          type: 'SET_ERROR',
          payload: { key: 'course', error: error.message }
        });
      } finally {
        dispatch({ type: 'SET_LOADING', payload: { key: 'course', loading: false } });
      }
    };

    loadEducationalCourse();
  }, [courseId, studentId, dispatch]);

  return {
    course: currentCourse.course,
    lessons: currentCourse.lessons,
    learningProgress: currentCourse.learningProgress,
    educationalMetrics: currentCourse.educationalMetrics,
    progressTracking,
    isLoading: loading.course,
    error: error.course
  };
};

export default EducationalLearningProvider;
```

This comprehensive educational React patterns guide provides learning-focused component architecture, educational state management, and LMS-optimized UI patterns specifically designed for Learning Management System development. The patterns prioritize educational effectiveness, accessibility, and student learning outcomes over generic web application development approaches. 