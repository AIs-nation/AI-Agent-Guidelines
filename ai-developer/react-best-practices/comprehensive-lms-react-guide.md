# LMS React Best Practices: Educational Component Architecture Guide

## Table of Contents
1. [Educational React Architecture](#educational-react-architecture)
2. [Learning Component Design Patterns](#learning-component-design-patterns)
3. [Student Progress State Management](#student-progress-state-management)
4. [Educational Accessibility Implementation](#educational-accessibility-implementation)
5. [Assessment Component Architecture](#assessment-component-architecture)
6. [Learning Analytics Integration](#learning-analytics-integration)
7. [Educational Performance Optimization](#educational-performance-optimization)
8. [Educational Privacy Implementation](#educational-privacy-implementation)

## Educational React Architecture

### 1. Learning-Centered Component Design

**✅ Good: Educational Component Architecture for Learning Management Systems**
```jsx
// Educational React patterns optimized for Learning Management Systems
// Designed with learning objectives, student privacy, and educational effectiveness

import React, { useState, useEffect, useContext, useMemo, useCallback } from 'react';
import { motion, AnimatePresence } from 'framer-motion';
import { useEducationalState, useStudentProgress, useLearningObjectives } from '../hooks/educational-state';
import { EducationalAccessibility } from '../components/educational-accessibility';
import { LearningAnalytics } from '../services/learning-analytics';

// Educational Context Provider for Learning Management
const EducationalContextProvider = ({ children, educationalConfig }) => {
  const [educationalState, setEducationalState] = useState({
    currentStudent: null,
    learningSession: null,
    educationalSettings: {
      accessibility_features: {
        high_contrast: false,
        screen_reader_mode: false,
        keyboard_navigation: true,
        text_scaling: 1.0,
        reduced_motion: false
      },
      learning_preferences: {
        visual_learner: false,
        auditory_learner: false,
        kinesthetic_learner: false,
        reading_writing_learner: false
      },
      privacy_settings: {
        ferpa_consent: false,
        coppa_protected: false,
        data_collection_minimal: false,
        parental_controls_active: false
      }
    },
    courseContext: {
      current_course: null,
      learning_objectives: [],
      prerequisite_knowledge: [],
      assessment_strategy: 'formative_summative'
    }
  });

  // Educational state management with learning context
  const updateEducationalContext = useCallback((updates) => {
    setEducationalState(prevState => ({
      ...prevState,
      ...updates,
      // Ensure educational privacy compliance
      privacy_protected: updates.privacy_settings?.coppa_protected || prevState.educationalSettings.privacy_settings.coppa_protected,
      // Track educational state changes for analytics
      last_updated: new Date().toISOString(),
      educational_tracking: {
        ...prevState.educational_tracking,
        context_updates: (prevState.educational_tracking?.context_updates || 0) + 1
      }
    }));
  }, []);

  // Educational context value with learning-focused utilities
  const educationalContextValue = useMemo(() => ({
    ...educationalState,
    updateEducationalContext,
    // Educational utility functions
    isAccessibilityEnabled: (feature) => educationalState.educationalSettings.accessibility_features[feature],
    hasLearningPreference: (preference) => educationalState.educationalSettings.learning_preferences[preference],
    isPrivacyProtected: () => educationalState.educationalSettings.privacy_settings.coppa_protected,
    getCurrentLearningObjectives: () => educationalState.courseContext.learning_objectives,
    getEducationalCompliance: () => ({
      ferpa_compliant: educationalState.educationalSettings.privacy_settings.ferpa_consent,
      coppa_compliant: educationalState.educationalSettings.privacy_settings.coppa_protected,
      accessibility_compliant: Object.values(educationalState.educationalSettings.accessibility_features).some(Boolean)
    })
  }), [educationalState, updateEducationalContext]);

  return (
    <EducationalContext.Provider value={educationalContextValue}>
      <div 
        className={`educational-platform ${educationalState.educationalSettings.accessibility_features.high_contrast ? 'high-contrast' : ''}`}
        data-educational-context="learning-management-system"
        data-privacy-level={educationalState.educationalSettings.privacy_settings.coppa_protected ? 'coppa_protected' : 'standard'}
      >
        {children}
      </div>
    </EducationalContext.Provider>
  );
};

// Educational Course Component with Learning Context
const EducationalCourseComponent = ({ 
  courseId, 
  studentId, 
  learningObjectives = [],
  accessibilityRequirements = {},
  privacyLevel = 'standard'
}) => {
  const { educationalState } = useContext(EducationalContext);
  const studentProgress = useStudentProgress(studentId);
  const courseObjectives = useLearningObjectives(courseId);
  
  // Educational state management
  const [educationalInteraction, setEducationalInteraction] = useState({
    current_lesson: null,
    progress_tracking: {
      lessons_completed: [],
      time_spent: 0,
      engagement_score: 0,
      mastery_levels: {}
    },
    learning_evidence: [],
    educational_feedback: []
  });

  // Educational accessibility configuration
  const accessibilityConfig = useMemo(() => ({
    ...educationalState.educationalSettings.accessibility_features,
    ...accessibilityRequirements,
    // Educational-specific accessibility
    learning_support: {
      closed_captions: true,
      audio_descriptions: true,
      keyboard_shortcuts: true,
      focus_indicators: true,
      skip_navigation: true
    }
  }), [educationalState.educationalSettings.accessibility_features, accessibilityRequirements]);

  // Educational progress tracking with learning analytics
  const trackEducationalProgress = useCallback(async (progressData) => {
    const {
      lesson_id,
      section_id,
      completion_status,
      time_spent,
      learning_evidence,
      mastery_demonstration
    } = progressData;

    // Educational privacy compliance check
    if (privacyLevel === 'coppa_protected') {
      // Minimal tracking for children under 13
      const minimalProgressData = {
        lesson_id,
        completion_status,
        educational_purpose_only: true,
        behavioral_tracking_disabled: true
      };
      
      await LearningAnalytics.trackMinimalProgress(studentId, minimalProgressData);
    } else {
      // Full educational tracking with consent
      const comprehensiveProgressData = {
        lesson_id,
        section_id,
        completion_status,
        time_spent,
        learning_evidence,
        mastery_demonstration,
        learning_objective_correlation: courseObjectives.find(obj => obj.lesson_id === lesson_id),
        engagement_metrics: {
          interaction_count: educationalInteraction.interaction_count || 0,
          focus_time: educationalInteraction.focus_time || 0,
          help_requests: educationalInteraction.help_requests || 0
        }
      };
      
      await LearningAnalytics.trackComprehensiveProgress(studentId, comprehensiveProgressData);
    }

    // Update educational interaction state
    setEducationalInteraction(prev => ({
      ...prev,
      progress_tracking: {
        ...prev.progress_tracking,
        lessons_completed: completion_status === 'completed' 
          ? [...prev.progress_tracking.lessons_completed, lesson_id]
          : prev.progress_tracking.lessons_completed,
        time_spent: prev.progress_tracking.time_spent + (time_spent || 0),
        mastery_levels: {
          ...prev.progress_tracking.mastery_levels,
          [lesson_id]: mastery_demonstration?.mastery_score || prev.progress_tracking.mastery_levels[lesson_id]
        }
      },
      learning_evidence: learning_evidence 
        ? [...prev.learning_evidence, learning_evidence]
        : prev.learning_evidence
    }));
  }, [studentId, courseObjectives, privacyLevel]);

  // Educational lesson rendering with learning context
  const renderEducationalLesson = useCallback((lesson) => {
    return (
      <motion.div
        key={lesson.id}
        className="educational-lesson-container"
        initial={{ opacity: 0, y: 20 }}
        animate={{ opacity: 1, y: 0 }}
        exit={{ opacity: 0, y: -20 }}
        transition={{ duration: 0.3, ease: "easeInOut" }}
        // Educational accessibility attributes
        role="article"
        aria-labelledby={`lesson-title-${lesson.id}`}
        aria-describedby={`lesson-description-${lesson.id}`}
        data-testid={`educational-lesson-${lesson.id}`}
        data-educational-type="lesson"
        data-learning-objectives={lesson.learning_objectives?.join(',')}
        data-accessibility-compliant="true"
        data-privacy-level={privacyLevel}
      >
        {/* Educational lesson header with learning objectives */}
        <div className="educational-lesson-header">
          <h2 
            id={`lesson-title-${lesson.id}`}
            className="educational-lesson-title"
            data-testid="lesson-title"
          >
            {lesson.title}
          </h2>
          
          {/* Learning objectives display */}
          {lesson.learning_objectives && (
            <div 
              className="learning-objectives-display"
              data-testid="learning-objectives-display"
              role="complementary"
              aria-label="Learning objectives for this lesson"
            >
              <h3>Learning Objectives:</h3>
              <ul role="list">
                {lesson.learning_objectives.map((objective, index) => (
                  <li 
                    key={index}
                    role="listitem"
                    data-learning-objective={objective.id}
                    data-bloom-level={objective.bloom_taxonomy_level}
                  >
                    {objective.description}
                  </li>
                ))}
              </ul>
            </div>
          )}
          
          {/* Educational progress indicator */}
          <div 
            className="educational-progress-indicator"
            data-testid="progress-tracker"
            role="progressbar"
            aria-valuenow={studentProgress?.lesson_progress?.[lesson.id]?.completion_percentage || 0}
            aria-valuemin="0"
            aria-valuemax="100"
            aria-label={`Lesson progress: ${studentProgress?.lesson_progress?.[lesson.id]?.completion_percentage || 0}% complete`}
          >
            <div 
              className="progress-bar"
              style={{ 
                width: `${studentProgress?.lesson_progress?.[lesson.id]?.completion_percentage || 0}%`,
                backgroundColor: accessibilityConfig.high_contrast ? '#000000' : '#4CAF50'
              }}
            />
          </div>
        </div>

        {/* Educational lesson content with accessibility features */}
        <div 
          className="educational-lesson-content"
          data-testid="lesson-content"
          data-educational-content="true"
        >
          {/* Educational content sections */}
          {lesson.sections?.map((section) => (
            <EducationalSectionComponent
              key={section.id}
              section={section}
              lessonId={lesson.id}
              onProgressUpdate={(progressData) => 
                trackEducationalProgress({
                  lesson_id: lesson.id,
                  section_id: section.id,
                  ...progressData
                })
              }
              accessibilityConfig={accessibilityConfig}
              privacyLevel={privacyLevel}
            />
          ))}
        </div>

        {/* Educational navigation controls */}
        <div 
          className="educational-navigation"
          data-testid="educational-navigation"
          role="navigation"
          aria-label="Lesson navigation"
        >
          <button
            type="button"
            className="educational-nav-button previous"
            onClick={() => navigateToLesson('previous')}
            disabled={!canNavigateToPrevious(lesson.id)}
            aria-label="Go to previous lesson"
            data-testid="previous-lesson-button"
          >
            ← Previous Lesson
          </button>
          
          <button
            type="button"
            className="educational-nav-button next"
            onClick={() => navigateToLesson('next')}
            disabled={!canNavigateToNext(lesson.id)}
            aria-label="Go to next lesson"
            data-testid="next-lesson-button"
          >
            Next Lesson →
          </button>
        </div>
      </motion.div>
    );
  }, [studentProgress, accessibilityConfig, privacyLevel, trackEducationalProgress]);

  return (
    <div 
      className="educational-course-container"
      data-testid="educational-course"
      data-course-id={courseId}
      data-student-id={studentId}
      data-privacy-level={privacyLevel}
    >
      {/* Educational accessibility features */}
      <EducationalAccessibility 
        config={accessibilityConfig}
        onConfigChange={(newConfig) => 
          educationalState.updateEducationalContext({
            educationalSettings: {
              ...educationalState.educationalSettings,
              accessibility_features: newConfig
            }
          })
        }
      />
      
      {/* Educational course content */}
      <AnimatePresence mode="wait">
        {educationalInteraction.current_lesson && 
          renderEducationalLesson(educationalInteraction.current_lesson)
        }
      </AnimatePresence>
    </div>
  );
};

// Educational Section Component with Learning Context
const EducationalSectionComponent = ({ 
  section, 
  lessonId, 
  onProgressUpdate,
  accessibilityConfig,
  privacyLevel 
}) => {
  const [sectionProgress, setSectionProgress] = useState({
    started: false,
    completed: false,
    time_spent: 0,
    interactions: 0,
    learning_evidence: null
  });

  // Educational interaction tracking
  const trackEducationalInteraction = useCallback((interactionType, interactionData = {}) => {
    setSectionProgress(prev => ({
      ...prev,
      interactions: prev.interactions + 1,
      started: true,
      last_interaction: new Date().toISOString(),
      interaction_types: {
        ...prev.interaction_types,
        [interactionType]: (prev.interaction_types?.[interactionType] || 0) + 1
      }
    }));

    // Educational analytics tracking
    if (privacyLevel !== 'coppa_protected') {
      LearningAnalytics.trackInteraction({
        section_id: section.id,
        lesson_id: lessonId,
        interaction_type: interactionType,
        interaction_data: interactionData,
        educational_context: {
          learning_objective: section.learning_objective,
          bloom_taxonomy_level: section.bloom_level || 'remembering',
          section_type: section.type
        }
      });
    }
  }, [section.id, lessonId, privacyLevel]);

  // Educational section completion handler
  const handleSectionCompletion = useCallback(async () => {
    const completionData = {
      completion_status: 'completed',
      time_spent: sectionProgress.time_spent,
      interaction_count: sectionProgress.interactions,
      learning_evidence: sectionProgress.learning_evidence,
      mastery_demonstration: {
        section_mastered: true,
        mastery_score: calculateMasteryScore(sectionProgress),
        bloom_level_achieved: section.bloom_level || 'remembering'
      }
    };

    setSectionProgress(prev => ({ ...prev, completed: true }));
    await onProgressUpdate(completionData);
  }, [sectionProgress, section, onProgressUpdate]);

  return (
    <div
      className={`educational-section ${section.type}`}
      data-testid={`educational-section-${section.id}`}
      data-section-type={section.type}
      data-learning-objective={section.learning_objective}
      data-bloom-level={section.bloom_level}
      role="region"
      aria-labelledby={`section-title-${section.id}`}
    >
      {/* Educational section header */}
      <div className="educational-section-header">
        <h3 
          id={`section-title-${section.id}`}
          className="educational-section-title"
        >
          {section.title}
        </h3>
        
        {/* Section progress indicator */}
        <div 
          className="section-progress-indicator"
          data-testid="section-progress"
          role="status"
          aria-live="polite"
          aria-atomic="true"
        >
          {sectionProgress.completed ? '✅ Completed' : 
           sectionProgress.started ? '▶️ In Progress' : '🔒 Locked'}
        </div>
      </div>

      {/* Educational section content */}
      <div 
        className="educational-section-content"
        onClick={() => trackEducationalInteraction('content_interaction')}
        onFocus={() => trackEducationalInteraction('content_focus')}
        tabIndex={0}
        role="button"
        aria-label={`Interactive section: ${section.title}`}
      >
        {section.content}
      </div>

      {/* Educational section completion controls */}
      {sectionProgress.started && !sectionProgress.completed && (
        <div className="educational-section-controls">
          <button
            type="button"
            className="educational-complete-button"
            onClick={handleSectionCompletion}
            aria-label={`Mark section ${section.title} as complete`}
            data-testid="section-complete-button"
          >
            Mark Section Complete
          </button>
        </div>
      )}
    </div>
  );
};

// Educational utility functions
const calculateMasteryScore = (progressData) => {
  const {
    time_spent,
    interactions,
    interaction_types = {},
    learning_evidence
  } = progressData;

  // Educational mastery calculation based on engagement and interaction quality
  let masteryScore = 0;

  // Time engagement factor (0-30 points)
  if (time_spent >= 300) masteryScore += 30; // 5+ minutes = full engagement
  else masteryScore += (time_spent / 300) * 30;

  // Interaction quality factor (0-40 points)
  const qualityInteractions = (interaction_types.deep_thinking || 0) + 
                             (interaction_types.problem_solving || 0) + 
                             (interaction_types.critical_analysis || 0);
  masteryScore += Math.min(qualityInteractions * 10, 40);

  // Learning evidence factor (0-30 points)
  if (learning_evidence?.demonstrates_understanding) masteryScore += 30;
  else if (learning_evidence?.shows_progress) masteryScore += 15;

  return Math.min(Math.round(masteryScore), 100);
};

export {
  EducationalContextProvider,
  EducationalCourseComponent,
  EducationalSectionComponent
};
```

### 2. Educational State Management Patterns

**✅ Good: Learning-Focused React State Management**
```jsx
// Educational React state management with learning analytics and privacy compliance
// Designed for comprehensive student progress tracking and educational effectiveness

import React, { useReducer, useContext, useEffect, useCallback } from 'react';
import { useSelector, useDispatch } from 'react-redux';

// Educational state reducer with learning context
const educationalReducer = (state, action) => {
  switch (action.type) {
    case 'START_LEARNING_SESSION':
      return {
        ...state,
        currentSession: {
          session_id: action.payload.session_id,
          started_at: new Date().toISOString(),
          course_id: action.payload.course_id,
          student_id: action.payload.student_id,
          learning_objectives: action.payload.learning_objectives || [],
          session_type: action.payload.session_type || 'self_paced',
          privacy_level: action.payload.privacy_level || 'standard'
        },
        sessionProgress: {
          lessons_accessed: [],
          sections_completed: [],
          time_spent: 0,
          engagement_score: 0,
          learning_evidence_collected: []
        }
      };

    case 'TRACK_EDUCATIONAL_PROGRESS':
      const { lesson_id, section_id, progress_data } = action.payload;
      return {
        ...state,
        sessionProgress: {
          ...state.sessionProgress,
          lessons_accessed: lesson_id && !state.sessionProgress.lessons_accessed.includes(lesson_id)
            ? [...state.sessionProgress.lessons_accessed, lesson_id]
            : state.sessionProgress.lessons_accessed,
          sections_completed: progress_data.completed && section_id
            ? [...state.sessionProgress.sections_completed, section_id]
            : state.sessionProgress.sections_completed,
          time_spent: state.sessionProgress.time_spent + (progress_data.time_spent || 0),
          engagement_score: calculateEngagementScore(state.sessionProgress, progress_data),
          learning_evidence_collected: progress_data.learning_evidence
            ? [...state.sessionProgress.learning_evidence_collected, progress_data.learning_evidence]
            : state.sessionProgress.learning_evidence_collected
        },
        studentProgress: {
          ...state.studentProgress,
          [lesson_id]: {
            ...state.studentProgress[lesson_id],
            sections_completed: progress_data.completed
              ? [...(state.studentProgress[lesson_id]?.sections_completed || []), section_id]
              : state.studentProgress[lesson_id]?.sections_completed || [],
            completion_percentage: calculateCompletionPercentage(
              state.studentProgress[lesson_id], 
              progress_data
            ),
            mastery_level: progress_data.mastery_score || state.studentProgress[lesson_id]?.mastery_level || 0
          }
        }
      };

    case 'UPDATE_LEARNING_PREFERENCES':
      return {
        ...state,
        learningPreferences: {
          ...state.learningPreferences,
          ...action.payload,
          last_updated: new Date().toISOString()
        }
      };

    case 'UPDATE_ACCESSIBILITY_SETTINGS':
      return {
        ...state,
        accessibilitySettings: {
          ...state.accessibilitySettings,
          ...action.payload,
          last_updated: new Date().toISOString()
        }
      };

    case 'UPDATE_PRIVACY_SETTINGS':
      return {
        ...state,
        privacySettings: {
          ...state.privacySettings,
          ...action.payload,
          consent_updated_at: new Date().toISOString()
        }
      };

    case 'END_LEARNING_SESSION':
      const sessionSummary = {
        session_id: state.currentSession?.session_id,
        ended_at: new Date().toISOString(),
        total_time: state.sessionProgress.time_spent,
        lessons_completed: state.sessionProgress.lessons_accessed.length,
        sections_completed: state.sessionProgress.sections_completed.length,
        final_engagement_score: state.sessionProgress.engagement_score,
        learning_evidence_count: state.sessionProgress.learning_evidence_collected.length
      };

      return {
        ...state,
        currentSession: null,
        sessionHistory: [
          ...state.sessionHistory,
          sessionSummary
        ],
        sessionProgress: {
          lessons_accessed: [],
          sections_completed: [],
          time_spent: 0,
          engagement_score: 0,
          learning_evidence_collected: []
        }
      };

    default:
      return state;
  }
};

// Educational React hook for learning state management
const useEducationalLearning = (studentId, courseId) => {
  const dispatch = useDispatch();
  const educationalState = useSelector(state => state.education);

  // Initialize educational learning session
  const startLearningSession = useCallback((sessionConfig = {}) => {
    const sessionData = {
      session_id: `session-${Date.now()}`,
      student_id: studentId,
      course_id: courseId,
      learning_objectives: sessionConfig.learning_objectives || [],
      session_type: sessionConfig.session_type || 'self_paced',
      privacy_level: sessionConfig.privacy_level || 'standard'
    };

    dispatch({
      type: 'education/startLearningSession',
      payload: sessionData
    });

    // Educational analytics tracking
    if (sessionConfig.privacy_level !== 'coppa_protected') {
      LearningAnalytics.trackSessionStart({
        student_id: studentId,
        course_id: courseId,
        session_config: sessionData
      });
    }
  }, [dispatch, studentId, courseId]);

  // Track educational progress with learning context
  const trackLearningProgress = useCallback((progressData) => {
    const {
      lesson_id,
      section_id,
      completion_status,
      time_spent,
      interaction_type,
      learning_evidence,
      mastery_demonstration
    } = progressData;

    // Educational progress tracking
    dispatch({
      type: 'education/trackProgress',
      payload: {
        lesson_id,
        section_id,
        progress_data: {
          completed: completion_status === 'completed',
          time_spent: time_spent || 0,
          learning_evidence,
          mastery_score: mastery_demonstration?.mastery_score,
          bloom_level_achieved: mastery_demonstration?.bloom_level,
          interaction_quality: calculateInteractionQuality(interaction_type)
        }
      }
    });

    // Educational analytics with privacy compliance
    const currentPrivacyLevel = educationalState.privacySettings?.privacy_level || 'standard';
    if (currentPrivacyLevel !== 'coppa_protected') {
      LearningAnalytics.trackDetailedProgress({
        student_id: studentId,
        course_id: courseId,
        lesson_id,
        section_id,
        progress_data: progressData,
        educational_context: {
          learning_objectives: educationalState.currentSession?.learning_objectives,
          session_type: educationalState.currentSession?.session_type
        }
      });
    } else {
      // Minimal tracking for COPPA-protected students
      LearningAnalytics.trackMinimalProgress({
        student_id: studentId,
        lesson_id,
        completion_status,
        educational_purpose_only: true
      });
    }
  }, [dispatch, studentId, courseId, educationalState]);

  // Update learning preferences with educational context
  const updateLearningPreferences = useCallback((preferences) => {
    dispatch({
      type: 'education/updateLearningPreferences',
      payload: preferences
    });

    // Track learning preference changes for adaptive learning
    if (educationalState.privacySettings?.privacy_level !== 'coppa_protected') {
      LearningAnalytics.trackPreferenceUpdate({
        student_id: studentId,
        preferences,
        adaptive_learning_enabled: true
      });
    }
  }, [dispatch, studentId, educationalState.privacySettings]);

  // Educational accessibility management
  const updateAccessibilitySettings = useCallback((accessibilityConfig) => {
    dispatch({
      type: 'education/updateAccessibilitySettings',
      payload: accessibilityConfig
    });

    // Educational accessibility analytics
    LearningAnalytics.trackAccessibilityUsage({
      student_id: studentId,
      accessibility_features: accessibilityConfig,
      educational_impact: true
    });
  }, [dispatch, studentId]);

  return {
    // Educational state
    currentSession: educationalState.currentSession,
    sessionProgress: educationalState.sessionProgress,
    studentProgress: educationalState.studentProgress,
    learningPreferences: educationalState.learningPreferences,
    accessibilitySettings: educationalState.accessibilitySettings,
    privacySettings: educationalState.privacySettings,

    // Educational actions
    startLearningSession,
    trackLearningProgress,
    updateLearningPreferences,
    updateAccessibilitySettings,

    // Educational utilities
    isSessionActive: !!educationalState.currentSession,
    getSessionDuration: () => educationalState.sessionProgress?.time_spent || 0,
    getCompletionRate: (lessonId) => educationalState.studentProgress?.[lessonId]?.completion_percentage || 0,
    getLearningEvidence: () => educationalState.sessionProgress?.learning_evidence_collected || [],
    getEngagementScore: () => educationalState.sessionProgress?.engagement_score || 0
  };
};

// Educational utility functions
const calculateEngagementScore = (currentProgress, newProgressData) => {
  const {
    time_spent = 0,
    interaction_quality = 0,
    learning_evidence,
    mastery_score = 0
  } = newProgressData;

  // Educational engagement calculation based on learning science
  let engagementScore = currentProgress.engagement_score || 0;
  
  // Time engagement factor (promotes sustained learning)
  if (time_spent >= 300) engagementScore += 10; // 5+ minutes
  else if (time_spent >= 120) engagementScore += 5; // 2+ minutes
  
  // Interaction quality factor (promotes deep learning)
  engagementScore += interaction_quality * 2;
  
  // Learning evidence factor (promotes understanding demonstration)
  if (learning_evidence?.demonstrates_understanding) engagementScore += 15;
  else if (learning_evidence?.shows_progress) engagementScore += 5;
  
  // Mastery achievement factor (promotes skill development)
  if (mastery_score >= 80) engagementScore += 20;
  else if (mastery_score >= 60) engagementScore += 10;
  
  return Math.min(engagementScore, 100);
};

const calculateCompletionPercentage = (currentLessonProgress, newProgressData) => {
  const totalSections = newProgressData.total_sections || 1;
  const completedSections = currentLessonProgress?.sections_completed?.length || 0;
  
  if (newProgressData.completed) {
    return Math.round(((completedSections + 1) / totalSections) * 100);
  }
  
  return Math.round((completedSections / totalSections) * 100);
};

const calculateInteractionQuality = (interactionType) => {
  // Educational interaction quality scoring based on Bloom's taxonomy
  const qualityScores = {
    'content_view': 1,           // Remembering
    'content_interaction': 2,     // Understanding
    'problem_solving': 4,         // Applying
    'critical_analysis': 6,       // Analyzing
    'creative_synthesis': 8,      // Evaluating
    'knowledge_creation': 10      // Creating
  };
  
  return qualityScores[interactionType] || 1;
};

export { useEducationalLearning, educationalReducer };
```

**❌ Bad: Generic React Patterns Without Educational Context**
```jsx
// Generic React component without educational considerations (NOT for LMS)

const GenericComponent = ({ data, userId }) => {
  const [state, setState] = useState({});
  
  return (
    <div>
      <h1>{data.title}</h1>
      <p>{data.content}</p>
      {/* No educational context, accessibility, or learning tracking */}
    </div>
  );
};

❌ Problems with this approach for educational platforms:
- No educational learning objective integration or progress tracking
- Missing educational accessibility features for inclusive learning
- Lacks educational privacy compliance (FERPA/COPPA) implementation
- No learning analytics tracking or educational effectiveness measurement
- Generic user interaction without educational context or learning evidence
- Missing educational state management for student progress and mastery
- No educational content delivery optimization or learning preference adaptation
- Lacks educational assessment integration and feedback mechanisms
```

This comprehensive LMS React Best Practices Guide provides learning-focused component architecture, educational state management patterns, and student-centered React development strategies specifically designed for Learning Management System development. The framework prioritizes educational effectiveness, accessibility compliance, and student privacy protection over generic React development approaches. 