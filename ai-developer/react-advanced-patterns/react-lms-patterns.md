# React Advanced Patterns for LMS Development

## Context Management Patterns for Educational Platforms

### Educational Context Architecture
✅ **Good Example: LMS Context Provider Pattern**
```jsx
// contexts/LMSContext.jsx - Comprehensive educational platform state management
import React, { createContext, useContext, useReducer, useEffect } from 'react';

// Educational platform state structure
const initialLMSState = {
  user: {
    id: null,
    profile: null,
    preferences: {
      difficulty: 'intermediate',
      learningStyle: 'visual',
      notifications: true
    }
  },
  courses: {
    available: [],
    enrolled: [],
    completed: [],
    loading: false,
    error: null
  },
  currentLearning: {
    courseId: null,
    lessonId: null,
    sectionId: null,
    progress: {},
    timeTracking: {
      sessionStart: null,
      totalTime: 0,
      activeTime: 0
    }
  },
  aiTeacher: {
    isActive: false,
    conversation: [],
    context: null,
    loading: false
  },
  progress: {
    courses: {},
    lessons: {},
    sections: {},
    analytics: {
      streaks: 0,
      weeklyGoals: null,
      achievements: []
    }
  }
};

// Educational platform actions
const lmsActions = {
  // Course management
  SET_COURSES: 'SET_COURSES',
  ADD_COURSE: 'ADD_COURSE',
  UPDATE_COURSE: 'UPDATE_COURSE',
  ENROLL_COURSE: 'ENROLL_COURSE',
  
  // Learning progression
  START_LESSON: 'START_LESSON',
  COMPLETE_SECTION: 'COMPLETE_SECTION',
  UPDATE_PROGRESS: 'UPDATE_PROGRESS',
  TRACK_TIME: 'TRACK_TIME',
  
  // AI Teacher integration
  ACTIVATE_AI_TEACHER: 'ACTIVATE_AI_TEACHER',
  ADD_AI_MESSAGE: 'ADD_AI_MESSAGE',
  SET_AI_CONTEXT: 'SET_AI_CONTEXT',
  
  // User interaction
  SET_USER: 'SET_USER',
  UPDATE_PREFERENCES: 'UPDATE_PREFERENCES',
  SET_LOADING: 'SET_LOADING',
  SET_ERROR: 'SET_ERROR'
};

// Educational platform state reducer
const lmsReducer = (state, action) => {
  switch (action.type) {
    case lmsActions.SET_COURSES:
      return {
        ...state,
        courses: {
          ...state.courses,
          available: action.payload,
          loading: false,
          error: null
        }
      };

    case lmsActions.ENROLL_COURSE:
      const { courseId, courseData } = action.payload;
      return {
        ...state,
        courses: {
          ...state.courses,
          enrolled: [...state.courses.enrolled, courseData]
        },
        progress: {
          ...state.progress,
          courses: {
            ...state.progress.courses,
            [courseId]: {
              enrolled: true,
              startDate: new Date().toISOString(),
              progress: 0,
              lessonsCompleted: 0,
              sectionsCompleted: 0
            }
          }
        }
      };

    case lmsActions.START_LESSON:
      return {
        ...state,
        currentLearning: {
          ...state.currentLearning,
          courseId: action.payload.courseId,
          lessonId: action.payload.lessonId,
          sectionId: action.payload.sectionId,
          timeTracking: {
            sessionStart: new Date(),
            totalTime: state.currentLearning.timeTracking.totalTime,
            activeTime: 0
          }
        }
      };

    case lmsActions.COMPLETE_SECTION:
      const { courseId: cId, lessonId: lId, sectionId: sId, timeSpent } = action.payload;
      return {
        ...state,
        progress: {
          ...state.progress,
          sections: {
            ...state.progress.sections,
            [`${cId}-${lId}-${sId}`]: {
              completed: true,
              completedAt: new Date().toISOString(),
              timeSpent: timeSpent,
              score: action.payload.score || null
            }
          }
        }
      };

    case lmsActions.ACTIVATE_AI_TEACHER:
      return {
        ...state,
        aiTeacher: {
          ...state.aiTeacher,
          isActive: true,
          context: action.payload.context
        }
      };

    case lmsActions.ADD_AI_MESSAGE:
      return {
        ...state,
        aiTeacher: {
          ...state.aiTeacher,
          conversation: [
            ...state.aiTeacher.conversation,
            action.payload.message
          ],
          loading: false
        }
      };

    default:
      return state;
  }
};

// LMS Context creation
const LMSContext = createContext();

// LMS Provider component
export const LMSProvider = ({ children }) => {
  const [state, dispatch] = useReducer(lmsReducer, initialLMSState);

  // Auto-save progress to localStorage
  useEffect(() => {
    const progressData = {
      progress: state.progress,
      currentLearning: state.currentLearning
    };
    localStorage.setItem('lms-progress', JSON.stringify(progressData));
  }, [state.progress, state.currentLearning]);

  // Load saved progress on mount
  useEffect(() => {
    const savedProgress = localStorage.getItem('lms-progress');
    if (savedProgress) {
      const parsed = JSON.parse(savedProgress);
      dispatch({ type: 'RESTORE_PROGRESS', payload: parsed });
    }
  }, []);

  const contextValue = {
    state,
    dispatch,
    actions: lmsActions
  };

  return (
    <LMSContext.Provider value={contextValue}>
      {children}
    </LMSContext.Provider>
  );
};

// Custom hook for LMS context
export const useLMS = () => {
  const context = useContext(LMSContext);
  if (!context) {
    throw new Error('useLMS must be used within an LMSProvider');
  }
  return context;
};

export default LMSContext;
```

## Learning Progress Hooks for Educational Platforms

### Advanced Progress Tracking Hook
✅ **Good Example: Comprehensive Learning Progress Management**
```jsx
// hooks/useLearningProgress.js - Advanced educational progress tracking
import { useState, useEffect, useCallback, useRef } from 'react';
import { useLMS } from '../contexts/LMSContext';

export const useLearningProgress = (courseId, lessonId) => {
  const { state, dispatch, actions } = useLMS();
  const [timeTracker, setTimeTracker] = useState({
    sessionStart: null,
    sectionStart: null,
    totalSessionTime: 0,
    currentSectionTime: 0
  });
  const intervalRef = useRef(null);

  // Get current progress for specific course/lesson
  const currentProgress = state.progress.courses[courseId] || {
    progress: 0,
    lessonsCompleted: 0,
    sectionsCompleted: 0
  };

  // Start time tracking for learning session
  const startTimeTracking = useCallback(() => {
    const now = new Date();
    setTimeTracker(prev => ({
      ...prev,
      sessionStart: now,
      sectionStart: now
    }));

    // Update time every second
    intervalRef.current = setInterval(() => {
      setTimeTracker(prev => {
        const now = new Date();
        const sessionTime = Math.floor((now - prev.sessionStart) / 1000);
        const sectionTime = Math.floor((now - prev.sectionStart) / 1000);
        
        return {
          ...prev,
          totalSessionTime: sessionTime,
          currentSectionTime: sectionTime
        };
      });
    }, 1000);
  }, []);

  // Stop time tracking
  const stopTimeTracking = useCallback(() => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  }, []);

  // Mark section as completed with progress persistence
  const completeSection = useCallback(async (sectionId, sectionData = {}) => {
    const timeSpent = timeTracker.currentSectionTime;
    
    try {
      // Update local state immediately for responsive UI
      dispatch({
        type: actions.COMPLETE_SECTION,
        payload: {
          courseId,
          lessonId,
          sectionId,
          timeSpent,
          score: sectionData.score,
          completedAt: new Date().toISOString()
        }
      });

      // Persist to backend
      const progressUpdate = {
        courseId,
        lessonId,
        sectionId,
        completed: true,
        timeSpent,
        score: sectionData.score
      };

      const response = await fetch('/api/progress/section', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(progressUpdate)
      });

      if (!response.ok) {
        throw new Error('Failed to save progress');
      }

      const result = await response.json();
      
      // Update progress analytics if lesson or course completed
      if (result.lessonCompleted) {
        dispatch({
          type: actions.COMPLETE_LESSON,
          payload: { courseId, lessonId }
        });
      }

      if (result.courseCompleted) {
        dispatch({
          type: actions.COMPLETE_COURSE,
          payload: { courseId }
        });
      }

      // Reset section timer for next section
      setTimeTracker(prev => ({
        ...prev,
        sectionStart: new Date(),
        currentSectionTime: 0
      }));

      return result;
    } catch (error) {
      console.error('Failed to save progress:', error);
      // Implement retry mechanism or offline queue
      throw error;
    }
  }, [courseId, lessonId, timeTracker.currentSectionTime, dispatch, actions]);

  // Get next section for automatic progression
  const getNextSection = useCallback(() => {
    const courseProgress = state.progress.courses[courseId];
    if (!courseProgress) return null;

    // Logic to determine next incomplete section
    // This would be implemented based on course structure
    // Return next section details for automatic navigation
  }, [courseId, state.progress]);

  // Calculate completion percentages
  const calculateProgress = useCallback(() => {
    const sections = state.progress.sections;
    const courseKey = courseId;
    
    const courseSections = Object.keys(sections).filter(key => 
      key.startsWith(`${courseKey}-`)
    );
    
    const completedSections = courseSections.filter(key => 
      sections[key].completed
    ).length;

    return {
      totalSections: courseSections.length,
      completedSections,
      percentage: courseSections.length > 0 
        ? Math.round((completedSections / courseSections.length) * 100) 
        : 0
    };
  }, [courseId, state.progress.sections]);

  // Initialize time tracking when component mounts
  useEffect(() => {
    startTimeTracking();
    return () => stopTimeTracking();
  }, [startTimeTracking, stopTimeTracking]);

  // Cleanup on unmount
  useEffect(() => {
    return () => {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
    };
  }, []);

  return {
    // Progress data
    currentProgress,
    progressStats: calculateProgress(),
    
    // Time tracking
    timeTracker,
    startTimeTracking,
    stopTimeTracking,
    
    // Progress actions
    completeSection,
    getNextSection,
    
    // Utilities
    isLoading: state.courses.loading,
    error: state.courses.error
  };
};
```

## Course Generation Component Patterns

### AI-Powered Course Generator Component
✅ **Good Example: Production-Ready Course Generation Interface**
```jsx
// components/CourseGenerator/CourseGeneratorWizard.jsx
import React, { useState, useEffect, useRef } from 'react';
import { useLMS } from '../../contexts/LMSContext';

const CourseGeneratorWizard = ({ onCourseGenerated, onClose }) => {
  const { dispatch, actions } = useLMS();
  const [currentStep, setCurrentStep] = useState(1);
  const [isGenerating, setIsGenerating] = useState(false);
  const [generationProgress, setGenerationProgress] = useState({
    stage: '',
    progress: 0,
    message: ''
  });
  const [courseData, setCourseData] = useState({
    title: '',
    description: '',
    difficulty: 'intermediate',
    tags: [],
    duration: 'medium'
  });
  const eventSourceRef = useRef(null);

  // Course generation steps
  const steps = [
    { id: 1, title: 'Course Details', component: CourseDetailsStep },
    { id: 2, title: 'Content Preferences', component: ContentPreferencesStep },
    { id: 3, title: 'Review & Generate', component: ReviewStep },
    { id: 4, title: 'Generation Progress', component: ProgressStep }
  ];

  // Handle course generation with Server-Sent Events
  const generateCourse = async () => {
    setIsGenerating(true);
    setCurrentStep(4);
    
    try {
      // Start Server-Sent Events connection for real-time progress
      eventSourceRef.current = new EventSource('/api/courses/generate-stream', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(courseData)
      });

      eventSourceRef.current.onmessage = (event) => {
        const data = JSON.parse(event.data);
        
        if (event.type === 'progress') {
          setGenerationProgress({
            stage: data.stage,
            progress: data.progress,
            message: data.message
          });
        } else if (event.type === 'complete') {
          setGenerationProgress({
            stage: 'complete',
            progress: 100,
            message: 'Course generated successfully!'
          });
          
          // Add course to context
          dispatch({
            type: actions.ADD_COURSE,
            payload: data.course
          });
          
          // Close SSE connection
          eventSourceRef.current.close();
          setIsGenerating(false);
          
          // Notify parent component
          onCourseGenerated(data.course);
        } else if (event.type === 'error') {
          console.error('Course generation failed:', data);
          setGenerationProgress({
            stage: 'error',
            progress: 0,
            message: data.message || 'Generation failed'
          });
          setIsGenerating(false);
        }
      };

      eventSourceRef.current.onerror = (error) => {
        console.error('SSE connection error:', error);
        setGenerationProgress({
          stage: 'error',
          progress: 0,
          message: 'Connection error during generation'
        });
        setIsGenerating(false);
      };

    } catch (error) {
      console.error('Failed to start course generation:', error);
      setGenerationProgress({
        stage: 'error',
        progress: 0,
        message: 'Failed to start generation'
      });
      setIsGenerating(false);
    }
  };

  // Cleanup SSE connection
  useEffect(() => {
    return () => {
      if (eventSourceRef.current) {
        eventSourceRef.current.close();
      }
    };
  }, []);

  // Navigate between steps
  const nextStep = () => {
    if (currentStep < steps.length - 1) {
      setCurrentStep(currentStep + 1);
    } else if (currentStep === 3) {
      generateCourse();
    }
  };

  const prevStep = () => {
    if (currentStep > 1) {
      setCurrentStep(currentStep - 1);
    }
  };

  const CurrentStepComponent = steps[currentStep - 1].component;

  return (
    <div className="course-generator-wizard">
      <div className="wizard-header">
        <h2>Create New Course with AI</h2>
        <div className="step-indicator">
          {steps.map((step, index) => (
            <div 
              key={step.id}
              className={`step ${currentStep >= step.id ? 'active' : ''} ${
                currentStep === step.id ? 'current' : ''
              }`}
            >
              <span className="step-number">{step.id}</span>
              <span className="step-title">{step.title}</span>
            </div>
          ))}
        </div>
      </div>

      <div className="wizard-content">
        <CurrentStepComponent
          courseData={courseData}
          setCourseData={setCourseData}
          generationProgress={generationProgress}
          isGenerating={isGenerating}
        />
      </div>

      <div className="wizard-footer">
        {currentStep > 1 && currentStep < 4 && (
          <button 
            onClick={prevStep}
            className="btn btn-secondary"
            disabled={isGenerating}
          >
            Previous
          </button>
        )}
        
        {currentStep < 3 && (
          <button 
            onClick={nextStep}
            className="btn btn-primary"
            disabled={!isStepValid(currentStep, courseData)}
          >
            Next
          </button>
        )}
        
        {currentStep === 3 && (
          <button 
            onClick={generateCourse}
            className="btn btn-success"
            disabled={isGenerating || !isStepValid(currentStep, courseData)}
          >
            Generate Course
          </button>
        )}

        {currentStep === 4 && !isGenerating && generationProgress.stage === 'complete' && (
          <button 
            onClick={onClose}
            className="btn btn-primary"
          >
            View Course
          </button>
        )}

        <button 
          onClick={onClose}
          className="btn btn-outline"
          disabled={isGenerating}
        >
          {isGenerating ? 'Generating...' : 'Cancel'}
        </button>
      </div>
    </div>
  );
};

// Step validation helper
const isStepValid = (step, data) => {
  switch (step) {
    case 1:
      return data.title.length >= 3 && data.description.length >= 10;
    case 2:
      return data.difficulty && data.tags.length > 0;
    case 3:
      return true; // Review step is always valid
    default:
      return false;
  }
};

// Course Details Step Component
const CourseDetailsStep = ({ courseData, setCourseData }) => {
  return (
    <div className="step-content">
      <h3>Tell us about your course</h3>
      
      <div className="form-group">
        <label htmlFor="course-title">Course Title *</label>
        <input
          id="course-title"
          type="text"
          value={courseData.title}
          onChange={(e) => setCourseData(prev => ({ 
            ...prev, 
            title: e.target.value 
          }))}
          placeholder="e.g., Introduction to Machine Learning"
          maxLength={200}
          required
        />
        <small>{courseData.title.length}/200 characters</small>
      </div>

      <div className="form-group">
        <label htmlFor="course-description">Course Description *</label>
        <textarea
          id="course-description"
          value={courseData.description}
          onChange={(e) => setCourseData(prev => ({ 
            ...prev, 
            description: e.target.value 
          }))}
          placeholder="Describe what students will learn in this course..."
          rows={4}
          maxLength={1000}
          required
        />
        <small>{courseData.description.length}/1000 characters</small>
      </div>

      <div className="form-group">
        <label htmlFor="difficulty">Difficulty Level</label>
        <select
          id="difficulty"
          value={courseData.difficulty}
          onChange={(e) => setCourseData(prev => ({ 
            ...prev, 
            difficulty: e.target.value 
          }))}
        >
          <option value="beginner">Beginner - No prior knowledge required</option>
          <option value="intermediate">Intermediate - Some background helpful</option>
          <option value="advanced">Advanced - Significant expertise required</option>
        </select>
      </div>
    </div>
  );
};

// Generation Progress Step Component
const ProgressStep = ({ generationProgress, isGenerating }) => {
  const getProgressColor = () => {
    if (generationProgress.stage === 'error') return '#ef4444';
    if (generationProgress.stage === 'complete') return '#10b981';
    return '#3b82f6';
  };

  const getStageDescription = (stage) => {
    const stages = {
      'starting': 'Initializing course generation...',
      'outline': 'Creating course structure and lesson outline...',
      'content': 'Generating educational content for lessons...',
      'image': 'Creating course thumbnail with AI...',
      'saving': 'Saving course to database...',
      'complete': 'Course generation completed successfully!',
      'error': 'An error occurred during generation'
    };
    return stages[stage] || 'Processing...';
  };

  return (
    <div className="progress-step">
      <h3>Generating Your Course</h3>
      
      <div className="progress-container">
        <div className="progress-circle">
          <svg width="120" height="120" viewBox="0 0 120 120">
            <circle
              cx="60"
              cy="60"
              r="50"
              fill="none"
              stroke="#e5e7eb"
              strokeWidth="8"
            />
            <circle
              cx="60"
              cy="60"
              r="50"
              fill="none"
              stroke={getProgressColor()}
              strokeWidth="8"
              strokeLinecap="round"
              strokeDasharray={`${2 * Math.PI * 50}`}
              strokeDashoffset={`${2 * Math.PI * 50 * (1 - generationProgress.progress / 100)}`}
              transform="rotate(-90 60 60)"
              style={{ transition: 'stroke-dashoffset 0.5s ease-in-out' }}
            />
          </svg>
          <div className="progress-text">
            <span className="progress-percentage">{generationProgress.progress}%</span>
          </div>
        </div>

        <div className="progress-info">
          <h4>{getStageDescription(generationProgress.stage)}</h4>
          <p>{generationProgress.message}</p>
          
          {isGenerating && (
            <div className="progress-stages">
              <div className={`stage ${generationProgress.stage === 'outline' ? 'active' : ''}`}>
                📋 Course Structure
              </div>
              <div className={`stage ${generationProgress.stage === 'content' ? 'active' : ''}`}>
                📚 Content Generation
              </div>
              <div className={`stage ${generationProgress.stage === 'image' ? 'active' : ''}`}>
                🎨 Visual Design
              </div>
              <div className={`stage ${generationProgress.stage === 'saving' ? 'active' : ''}`}>
                💾 Finalizing
              </div>
            </div>
          )}
        </div>
      </div>

      {generationProgress.stage === 'error' && (
        <div className="error-message">
          <p>Course generation encountered an error. Please try again or contact support if the problem persists.</p>
        </div>
      )}

      {generationProgress.stage === 'complete' && (
        <div className="success-message">
          <p>🎉 Your course has been successfully generated! You can now start adding it to your curriculum.</p>
        </div>
      )}
    </div>
  );
};

export default CourseGeneratorWizard;
```

## Learning Interface Component Patterns

### Advanced Section Progression Component
✅ **Good Example: Educational Section Navigation with Progress Tracking**
```jsx
// components/LearningInterface/SectionProgress.jsx
import React, { useState, useEffect, useCallback } from 'react';
import { useLearningProgress } from '../../hooks/useLearningProgress';
import { useLMS } from '../../contexts/LMSContext';

const SectionProgressInterface = ({ 
  courseId, 
  lessonId, 
  sections, 
  initialSectionId 
}) => {
  const { state } = useLMS();
  const { completeSection, timeTracker, progressStats } = useLearningProgress(courseId, lessonId);
  const [currentSectionIndex, setCurrentSectionIndex] = useState(0);
  const [sectionStates, setSectionStates] = useState({});

  // Initialize section states from progress data
  useEffect(() => {
    const states = {};
    sections.forEach((section, index) => {
      const progressKey = `${courseId}-${lessonId}-${section.id}`;
      const sectionProgress = state.progress.sections[progressKey];
      
      states[section.id] = {
        status: sectionProgress?.completed ? 'completed' : 
                index === 0 || states[sections[index - 1]?.id]?.status === 'completed' ? 'unlocked' : 'locked',
        completed: sectionProgress?.completed || false,
        timeSpent: sectionProgress?.timeSpent || 0,
        score: sectionProgress?.score || null
      };
    });
    setSectionStates(states);
  }, [sections, courseId, lessonId, state.progress.sections]);

  // Handle section completion
  const handleSectionComplete = useCallback(async (sectionId, additionalData = {}) => {
    try {
      await completeSection(sectionId, additionalData);
      
      // Update local section states
      setSectionStates(prev => {
        const updated = { ...prev };
        const sectionIndex = sections.findIndex(s => s.id === sectionId);
        
        // Mark current section as completed
        updated[sectionId] = {
          ...updated[sectionId],
          status: 'completed',
          completed: true,
          timeSpent: timeTracker.currentSectionTime,
          score: additionalData.score
        };
        
        // Unlock next section if exists
        if (sectionIndex < sections.length - 1) {
          const nextSection = sections[sectionIndex + 1];
          updated[nextSection.id] = {
            ...updated[nextSection.id],
            status: 'unlocked'
          };
        }
        
        return updated;
      });
      
      // Auto-advance to next section
      const nextIndex = currentSectionIndex + 1;
      if (nextIndex < sections.length) {
        setTimeout(() => {
          setCurrentSectionIndex(nextIndex);
        }, 1500); // Brief celebration delay
      }
      
    } catch (error) {
      console.error('Failed to complete section:', error);
      // Show error notification
    }
  }, [completeSection, currentSectionIndex, sections, timeTracker.currentSectionTime]);

  // Navigate to specific section (if unlocked)
  const navigateToSection = useCallback((sectionIndex) => {
    const section = sections[sectionIndex];
    const sectionState = sectionStates[section.id];
    
    if (sectionState?.status === 'locked') {
      // Show locked section message
      return;
    }
    
    setCurrentSectionIndex(sectionIndex);
  }, [sections, sectionStates]);

  const currentSection = sections[currentSectionIndex];
  const currentSectionState = sectionStates[currentSection?.id] || {};

  return (
    <div className="section-progress-interface">
      {/* Progress Overview */}
      <div className="lesson-progress-header">
        <div className="progress-summary">
          <h2>Lesson Progress</h2>
          <div className="progress-stats">
            <span className="completed-count">
              {progressStats.completedSections}/{progressStats.totalSections} sections completed
            </span>
            <div className="progress-bar">
              <div 
                className="progress-fill"
                style={{ width: `${progressStats.percentage}%` }}
              />
            </div>
            <span className="percentage">{progressStats.percentage}%</span>
          </div>
        </div>
        
        <div className="time-tracker">
          <span className="session-time">
            ⏱️ Session: {formatTime(timeTracker.totalSessionTime)}
          </span>
          <span className="section-time">
            Current: {formatTime(timeTracker.currentSectionTime)}
          </span>
        </div>
      </div>

      {/* Section Navigation */}
      <div className="section-navigation">
        <div className="section-list">
          {sections.map((section, index) => {
            const sectionState = sectionStates[section.id] || {};
            return (
              <SectionNavItem
                key={section.id}
                section={section}
                index={index}
                state={sectionState}
                isActive={index === currentSectionIndex}
                onClick={() => navigateToSection(index)}
              />
            );
          })}
        </div>
      </div>

      {/* Current Section Content */}
      <div className="section-content">
        {currentSection && (
          <SectionContentRenderer
            section={currentSection}
            sectionState={currentSectionState}
            onComplete={(additionalData) => 
              handleSectionComplete(currentSection.id, additionalData)
            }
            isCompleted={currentSectionState.completed}
          />
        )}
      </div>

      {/* Section Controls */}
      <div className="section-controls">
        <button
          onClick={() => navigateToSection(Math.max(0, currentSectionIndex - 1))}
          disabled={currentSectionIndex === 0}
          className="btn btn-secondary"
        >
          ← Previous Section
        </button>

        <div className="section-actions">
          {!currentSectionState.completed && (
            <button
              onClick={() => handleSectionComplete(currentSection.id)}
              className="btn btn-success"
            >
              Mark Complete & Continue →
            </button>
          )}
          
          {currentSectionState.completed && currentSectionIndex < sections.length - 1 && (
            <button
              onClick={() => navigateToSection(currentSectionIndex + 1)}
              className="btn btn-primary"
            >
              Next Section →
            </button>
          )}
          
          {currentSectionState.completed && currentSectionIndex === sections.length - 1 && (
            <button
              onClick={() => window.location.href = `/courses/${courseId}`}
              className="btn btn-success"
            >
              Complete Lesson ✅
            </button>
          )}
        </div>
      </div>
    </div>
  );
};

// Section Navigation Item Component
const SectionNavItem = ({ section, index, state, isActive, onClick }) => {
  const getStatusIcon = () => {
    switch (state.status) {
      case 'completed': return '✅';
      case 'unlocked': return '▶️';
      case 'locked': return '🔒';
      default: return '⭕';
    }
  };

  const getStatusClass = () => {
    let classes = ['section-nav-item'];
    if (isActive) classes.push('active');
    if (state.status) classes.push(`status-${state.status}`);
    return classes.join(' ');
  };

  return (
    <div 
      className={getStatusClass()}
      onClick={onClick}
      role="button"
      tabIndex={state.status === 'locked' ? -1 : 0}
      aria-disabled={state.status === 'locked'}
    >
      <div className="section-icon">
        {getStatusIcon()}
      </div>
      <div className="section-info">
        <div className="section-title">{section.title}</div>
        <div className="section-meta">
          <span className="section-type">{section.type}</span>
          {state.timeSpent > 0 && (
            <span className="time-spent">⏱️ {formatTime(state.timeSpent)}</span>
          )}
          {state.score !== null && (
            <span className="score">🎯 {state.score}%</span>
          )}
        </div>
      </div>
    </div>
  );
};

// Section Content Renderer
const SectionContentRenderer = ({ section, sectionState, onComplete, isCompleted }) => {
  const [interactionData, setInteractionData] = useState({});

  const handleContentInteraction = (data) => {
    setInteractionData(prev => ({ ...prev, ...data }));
  };

  const renderContentByType = () => {
    switch (section.type) {
      case 'text':
        return (
          <TextSectionContent
            content={section.content}
            onInteraction={handleContentInteraction}
          />
        );
      case 'quiz':
        return (
          <QuizSectionContent
            content={section.content}
            onComplete={(score) => onComplete({ score })}
            isCompleted={isCompleted}
          />
        );
      case 'exercise':
        return (
          <ExerciseSectionContent
            content={section.content}
            onComplete={(result) => onComplete({ score: result.score })}
            isCompleted={isCompleted}
          />
        );
      default:
        return <div>Unknown section type: {section.type}</div>;
    }
  };

  return (
    <div className="section-content-container">
      <div className="section-header">
        <h3>{section.title}</h3>
        <div className="section-badges">
          <span className={`type-badge type-${section.type}`}>
            {section.type.toUpperCase()}
          </span>
          {isCompleted && (
            <span className="completed-badge">✅ COMPLETED</span>
          )}
        </div>
      </div>
      
      <div className="section-body">
        {renderContentByType()}
      </div>
    </div>
  );
};

// Utility function for time formatting
const formatTime = (seconds) => {
  const mins = Math.floor(seconds / 60);
  const secs = seconds % 60;
  return `${mins}:${secs.toString().padStart(2, '0')}`;
};

export default SectionProgressInterface;
```

## Key React LMS Patterns Takeaways

### Critical React Patterns for Educational Platforms ✅

1. **Educational Context Management**
   - Implement comprehensive LMS state with user progress, courses, and learning analytics
   - Use reducers for complex state transitions in educational workflows
   - Persist learning progress to localStorage for offline capability
   - Structure context for scalable educational feature expansion

2. **Progress Tracking Hooks**
   - Create specialized hooks for learning progress with time tracking
   - Implement real-time progress persistence to backend APIs
   - Handle offline scenarios with retry mechanisms and local caching
   - Provide granular progress analytics at section, lesson, and course levels

3. **Course Generation Patterns**
   - Use Server-Sent Events for real-time AI course generation progress
   - Implement wizard-style interfaces for complex course creation workflows
   - Handle long-running AI operations with proper loading states and error recovery
   - Structure generation flow with clear step validation and user feedback

4. **Learning Interface Components**
   - Design section progression with visual status indicators (locked, unlocked, completed)
   - Implement automatic section unlocking based on completion logic
   - Provide comprehensive time tracking and session analytics
   - Create adaptive content rendering based on section types (text, quiz, exercise)

5. **Educational UX Patterns**
   - Use progressive disclosure for complex learning interfaces
   - Implement motivational elements like progress visualization and achievements
   - Provide clear navigation between educational content levels
   - Design for accessibility with proper ARIA labels and keyboard navigation

### React Anti-Patterns to Avoid in LMS Development ❌

1. **Prop Drilling for Learning State** - Passing progress data through multiple component levels instead of using context
2. **Blocking UI During Course Generation** - Not providing real-time feedback during long AI operations
3. **Lost Progress on Navigation** - Not persisting learning progress properly across page changes
4. **Inconsistent Section States** - Not maintaining proper section locking/unlocking logic
5. **Memory Leaks in Timers** - Not properly cleaning up time tracking intervals and event sources

React patterns for LMS development require specialized attention to educational workflows, progress persistence, and user engagement while maintaining performance and accessibility standards for effective learning experiences. 