# Comprehensive React Best Practices for LMS Development

## Table of Contents
1. [Educational React Architecture Philosophy](#educational-react-architecture-philosophy)
2. [LMS Component Design Patterns](#lms-component-design-patterns)
3. [Educational State Management](#educational-state-management)
4. [Accessibility & WCAG Compliance](#accessibility--wcag-compliance)
5. [Educational Data Protection & Privacy](#educational-data-protection--privacy)
6. [Performance Optimization for Learning](#performance-optimization-for-learning)
7. [Testing Strategies for Educational Apps](#testing-strategies-for-educational-apps)

## Educational React Architecture Philosophy

### 🎓 Learning-Centered Component Architecture

```typescript
// Educational React Architecture Framework
interface EducationalReactArchitecture {
  design_philosophy: 'learning_outcome_driven_development',
  component_patterns: {
    educational_hierarchy: 'course > module > lesson > activity > assessment',
    accessibility_first: 'wcag_2_1_aa_minimum_compliance',
    data_protection: 'ferpa_coppa_compliant_by_design',
    learning_analytics: 'privacy_preserving_progress_tracking'
  },
  state_management: {
    educational_context: 'course_progress_and_learning_state_management',
    offline_capabilities: 'progressive_web_app_for_continuous_learning',
    real_time_collaboration: 'synchronized_learning_experiences',
    adaptive_learning: 'personalized_content_delivery_optimization'
  },
  performance_targets: {
    page_load_time: 'under_2_seconds_for_educational_content',
    interaction_responsiveness: 'under_100ms_for_learning_interactions',
    accessibility_compliance: 'wcag_2_1_aa_with_aaa_targets',
    offline_functionality: '100_percent_core_learning_content_available'
  }
}

// Good Example: Educational Component Architecture ✅
interface LearningContentProps {
  courseId: string;
  moduleId: string;
  lessonId: string;
  studentId: string;
  accessibilityPreferences: AccessibilityPreferences;
  learningContext: LearningContext;
  privacySettings: EducationalPrivacySettings;
}

interface LearningContext {
  currentProgress: CourseProgress;
  learningObjectives: LearningObjective[];
  adaptiveSettings: AdaptiveLearningSettings;
  collaborationMode: CollaborationMode;
  assessmentContext: AssessmentContext;
}

interface EducationalPrivacySettings {
  ferpaCompliance: FERPAComplianceSettings;
  coppaApplicable: boolean;
  dataMinimization: DataMinimizationSettings;
  parentalConsent: ParentalConsentStatus;
  analyticsOptIn: AnalyticsConsentSettings;
}

// Educational Component with Learning-First Design
const LearningContentComponent: React.FC<LearningContentProps> = ({
  courseId,
  moduleId,
  lessonId,
  studentId,
  accessibilityPreferences,
  learningContext,
  privacySettings
}) => {
  // Educational state management with privacy protection
  const {
    contentData,
    progressData,
    interactionAnalytics,
    assessmentData,
    loading,
    error
  } = useEducationalContent({
    courseId,
    moduleId,
    lessonId,
    studentId,
    privacySettings,
    accessibilityPreferences
  });

  // Learning analytics with differential privacy
  const { trackLearningInteraction } = useLearningAnalytics({
    privacyMode: privacySettings.analyticsOptIn,
    dataMinimization: privacySettings.dataMinimization,
    ferpaCompliance: privacySettings.ferpaCompliance
  });

  // Accessibility enhancement with WCAG 2.1 compliance
  const {
    announceToScreenReader,
    focusManagement,
    keyboardNavigation
  } = useAccessibilityEnhancement({
    preferences: accessibilityPreferences,
    wcagLevel: 'AA',
    educationalContext: true
  });

  // Real-time collaboration with educational privacy
  const {
    collaborationState,
    joinCollaborativeSession,
    shareSecurely
  } = useEducationalCollaboration({
    lessonId,
    studentId,
    privacySettings,
    collaborationMode: learningContext.collaborationMode
  });

  // Progressive enhancement for offline learning
  const {
    isOffline,
    syncPendingData,
    offlineCapabilities
  } = useOfflineLearning({
    contentId: lessonId,
    prioritizeEducationalContent: true,
    syncStrategy: 'educational_priority_based'
  });

  // Educational error boundary with learning continuity
  if (error) {
    return (
      <EducationalErrorBoundary
        error={error}
        courseContext={{ courseId, moduleId, lessonId }}
        recoveryOptions={{
          retryWithOfflineContent: true,
          contactInstructor: true,
          alternativeLearningPath: true
        }}
        accessibilityCompliant={true}
      />
    );
  }

  return (
    <div
      className="learning-content-container"
      role="main"
      aria-labelledby="lesson-title"
      aria-describedby="lesson-description"
    >
      {/* Educational Progress Indicator with Privacy Protection */}
      <EducationalProgressIndicator
        currentProgress={learningContext.currentProgress}
        learningObjectives={learningContext.learningObjectives}
        privacyMode={privacySettings.dataMinimization}
        accessibilityEnhanced={true}
      />

      {/* Adaptive Learning Content with Personalization */}
      <AdaptiveLearningContent
        content={contentData}
        adaptiveSettings={learningContext.adaptiveSettings}
        accessibilityPreferences={accessibilityPreferences}
        onInteraction={(interaction) => 
          trackLearningInteraction(interaction, { 
            privacyPreserving: true,
            educationalContext: learningContext 
          })
        }
      />

      {/* Real-time Educational Collaboration */}
      {learningContext.collaborationMode.enabled && (
        <EducationalCollaborationPanel
          collaborationState={collaborationState}
          privacySettings={privacySettings}
          accessibilityCompliant={true}
          educationalGuidelines={{
            peerLearning: true,
            instructorModeration: true,
            academicIntegrity: true
          }}
        />
      )}

      {/* Assessment Component with Academic Integrity */}
      {learningContext.assessmentContext.hasAssessment && (
        <EducationalAssessmentComponent
          assessmentData={assessmentData}
          privacySettings={privacySettings}
          accessibilityCompliant={true}
          academicIntegrityMeasures={{
            proctoring: learningContext.assessmentContext.requiresProctoring,
            timeConstraints: learningContext.assessmentContext.timeLimit,
            secureMode: learningContext.assessmentContext.secureMode
          }}
        />
      )}

      {/* Educational Navigation with Learning Path Awareness */}
      <EducationalNavigation
        currentLesson={lessonId}
        courseStructure={{
          courseId,
          moduleId,
          availableLessons: contentData.availableLessons,
          prerequisites: contentData.prerequisites,
          learningPath: learningContext.currentProgress.learningPath
        }}
        accessibilityEnhanced={true}
        keyboardNavigationOptimized={true}
      />
    </div>
  );
};

// Bad Example: Generic Component Without Educational Context ❌
const GenericContent: React.FC<{ data: any }> = ({ data }) => {
  return (
    <div>
      <h1>{data.title}</h1>
      <p>{data.content}</p>
      <button onClick={() => console.log('clicked')}>
        Next
      </button>
    </div>
  );
};
```

## LMS Component Design Patterns

### 📚 Educational Component Hierarchy

```typescript
// Educational Component Design Patterns
interface EducationalComponentPatterns {
  hierarchical_design: {
    course_container: 'top_level_educational_context_provider',
    module_wrapper: 'learning_objective_scoped_container',
    lesson_component: 'individual_learning_unit_with_progress_tracking',
    activity_element: 'interactive_learning_component_with_assessment',
    content_fragment: 'atomic_educational_content_with_accessibility'
  },
  educational_functionality: {
    progress_tracking: 'privacy_preserving_learning_analytics',
    adaptive_content: 'personalized_learning_path_optimization',
    assessment_integration: 'secure_academic_integrity_assessment',
    collaboration_features: 'educationally_appropriate_peer_interaction',
    accessibility_compliance: 'wcag_2_1_aa_educational_enhancement'
  },
  privacy_protection: {
    ferpa_compliance: 'educational_record_protection_by_design',
    coppa_adherence: 'parental_consent_and_child_privacy_protection',
    data_minimization: 'purpose_limited_educational_data_collection',
    consent_management: 'granular_educational_data_consent_controls'
  }
}

// Good Example: Educational Course Container ✅
interface CourseContainerProps {
  courseId: string;
  studentId: string;
  instructorId: string;
  educationalPrivacySettings: EducationalPrivacySettings;
  accessibilityRequirements: AccessibilityRequirements;
  institutionalPolicies: InstitutionalPolicies;
}

const EducationalCourseContainer: React.FC<CourseContainerProps> = ({
  courseId,
  studentId,
  instructorId,
  educationalPrivacySettings,
  accessibilityRequirements,
  institutionalPolicies
}) => {
  // Educational context management with privacy protection
  const educationalContext = useEducationalContext({
    courseId,
    studentId,
    instructorId,
    privacySettings: educationalPrivacySettings,
    institutionalPolicies
  });

  // Course-level accessibility configuration
  const accessibilityContext = useAccessibilityContext({
    requirements: accessibilityRequirements,
    wcagLevel: 'AA',
    educationalEnhancements: true,
    assistiveTechnologySupport: true
  });

  // Educational data protection and compliance
  const privacyContext = useEducationalPrivacyContext({
    ferpaCompliance: educationalPrivacySettings.ferpaCompliance,
    coppaApplicable: educationalPrivacySettings.coppaApplicable,
    dataGovernance: institutionalPolicies.dataGovernance,
    parentalRights: educationalPrivacySettings.parentalConsent
  });

  // Learning analytics with differential privacy
  const analyticsContext = useLearningAnalyticsContext({
    privacyPreserving: true,
    educationalPurposeOnly: true,
    dataMinimization: educationalPrivacySettings.dataMinimization,
    aggregationLevel: 'privacy_preserving_insights'
  });

  return (
    <EducationalContext.Provider value={educationalContext}>
      <AccessibilityContext.Provider value={accessibilityContext}>
        <PrivacyContext.Provider value={privacyContext}>
          <LearningAnalyticsContext.Provider value={analyticsContext}>
            <div
              className="educational-course-container"
              role="application"
              aria-label={`Course: ${educationalContext.courseMetadata.title}`}
              aria-describedby="course-description"
            >
              {/* Course Header with Educational Metadata */}
              <EducationalCourseHeader
                courseMetadata={educationalContext.courseMetadata}
                progressSummary={educationalContext.progressSummary}
                accessibilityEnhanced={true}
                privacyCompliant={true}
              />

              {/* Educational Navigation with Learning Path */}
              <EducationalCourseNavigation
                courseStructure={educationalContext.courseStructure}
                currentPosition={educationalContext.currentPosition}
                learningPath={educationalContext.personalizedLearningPath}
                accessibilityOptimized={true}
              />

              {/* Main Educational Content Area */}
              <main
                className="educational-content-main"
                role="main"
                aria-labelledby="current-lesson-title"
              >
                <Suspense 
                  fallback={
                    <EducationalLoadingIndicator
                      message="Loading educational content..."
                      accessibilityAnnouncement="Educational content is loading, please wait"
                      estimatedTime={educationalContext.estimatedLoadTime}
                    />
                  }
                >
                  <EducationalContentRouter
                    courseId={courseId}
                    currentModule={educationalContext.currentModule}
                    currentLesson={educationalContext.currentLesson}
                    privacySettings={educationalPrivacySettings}
                    accessibilityRequirements={accessibilityRequirements}
                  />
                </Suspense>
              </main>

              {/* Educational Sidebar with Learning Tools */}
              <EducationalSidebar
                learningTools={educationalContext.availableLearningTools}
                collaborationOptions={educationalContext.collaborationOptions}
                assessmentSchedule={educationalContext.assessmentSchedule}
                accessibilityEnhanced={true}
                privacyCompliant={true}
              />

              {/* Educational Footer with Support Resources */}
              <EducationalFooter
                supportResources={educationalContext.supportResources}
                academicPolicies={institutionalPolicies.academicPolicies}
                accessibilityResources={accessibilityContext.supportResources}
                privacyInformation={privacyContext.privacyInformation}
              />
            </div>
          </LearningAnalyticsContext.Provider>
        </PrivacyContext.Provider>
      </AccessibilityContext.Provider>
    </EducationalContext.Provider>
  );
};

// Educational Module Component with Learning Objectives
interface ModuleComponentProps {
  moduleId: string;
  learningObjectives: LearningObjective[];
  prerequisites: PrerequisiteRequirement[];
  assessmentCriteria: AssessmentCriteria[];
  collaborationSettings: CollaborationSettings;
}

const EducationalModuleComponent: React.FC<ModuleComponentProps> = ({
  moduleId,
  learningObjectives,
  prerequisites,
  assessmentCriteria,
  collaborationSettings
}) => {
  const { educationalContext } = useContext(EducationalContext);
  const { accessibilityContext } = useContext(AccessibilityContext);
  const { privacyContext } = useContext(PrivacyContext);
  const { analyticsContext } = useContext(LearningAnalyticsContext);

  // Module-specific learning state management
  const {
    moduleProgress,
    currentLesson,
    completedActivities,
    pendingAssessments,
    collaborativeElements
  } = useModuleLearningState({
    moduleId,
    studentId: educationalContext.studentId,
    privacySettings: privacyContext.privacySettings
  });

  // Learning objective tracking with privacy protection
  const {
    objectiveProgress,
    trackObjectiveAchievement,
    generateProgressReport
  } = useLearningObjectiveTracking({
    objectives: learningObjectives,
    privacyMode: 'educational_purpose_only',
    aggregationLevel: 'individual_student_privacy_preserving'
  });

  // Prerequisites validation with educational guidance
  const {
    prerequisitesStatus,
    recommendedPreparation,
    alternativeLearningPaths
  } = usePrerequisiteValidation({
    prerequisites,
    studentBackground: educationalContext.studentProfile,
    adaptiveLearning: true
  });

  return (
    <section
      className="educational-module"
      role="region"
      aria-labelledby={`module-${moduleId}-title`}
      aria-describedby={`module-${moduleId}-objectives`}
    >
      {/* Module Header with Learning Objectives */}
      <EducationalModuleHeader
        moduleId={moduleId}
        learningObjectives={learningObjectives}
        prerequisitesStatus={prerequisitesStatus}
        progressSummary={moduleProgress}
        accessibilityEnhanced={true}
      />

      {/* Prerequisites Guidance Panel */}
      {!prerequisitesStatus.allMet && (
        <EducationalPrerequisitesPanel
          unmetPrerequisites={prerequisitesStatus.unmet}
          recommendedPreparation={recommendedPreparation}
          alternativePaths={alternativeLearningPaths}
          accessibilityCompliant={true}
        />
      )}

      {/* Learning Objectives Progress Tracker */}
      <EducationalObjectivesTracker
        objectives={learningObjectives}
        progress={objectiveProgress}
        onAchievement={(objectiveId, achievement) =>
          trackObjectiveAchievement(objectiveId, achievement, {
            privacyPreserving: true,
            educationalContext: true
          })
        }
        accessibilityEnhanced={true}
      />

      {/* Module Lessons Grid with Adaptive Layout */}
      <EducationalLessonsGrid
        lessons={moduleProgress.lessons}
        currentLesson={currentLesson}
        completionStatus={moduleProgress.completionStatus}
        collaborationEnabled={collaborationSettings.enabled}
        accessibilityOptimized={true}
        adaptiveLayout={true}
      />

      {/* Collaborative Learning Elements */}
      {collaborationSettings.enabled && (
        <EducationalCollaborationSection
          collaborativeElements={collaborativeElements}
          collaborationSettings={collaborationSettings}
          privacySettings={privacyContext.privacySettings}
          accessibilityCompliant={true}
        />
      )}

      {/* Module Assessment Overview */}
      <EducationalAssessmentOverview
        assessmentCriteria={assessmentCriteria}
        pendingAssessments={pendingAssessments}
        completedAssessments={moduleProgress.completedAssessments}
        academicIntegrityGuidelines={educationalContext.academicIntegrityGuidelines}
        accessibilityAccommodations={accessibilityContext.assessmentAccommodations}
      />
    </section>
  );
};

// Bad Example: Generic Module Without Educational Context ❌
const GenericModule: React.FC<{ data: any }> = ({ data }) => {
  return (
    <div>
      <h2>{data.title}</h2>
      {data.lessons.map((lesson: any) => (
        <div key={lesson.id}>
          <h3>{lesson.title}</h3>
          <p>{lesson.content}</p>
        </div>
      ))}
    </div>
  );
};
```

## Educational State Management

### 🧠 Learning-Centered State Architecture

```typescript
// Educational State Management Framework
interface EducationalStateManagement {
  state_structure: {
    course_state: 'hierarchical_course_module_lesson_activity_structure',
    student_state: 'privacy_protected_learning_profile_and_progress',
    instructor_state: 'teaching_tools_and_class_management_state',
    collaboration_state: 'real_time_educational_collaboration_synchronization',
    assessment_state: 'secure_assessment_and_academic_integrity_tracking'
  },
  privacy_protection: {
    data_minimization: 'purpose_limited_educational_data_collection',
    encryption_at_rest: 'ferpa_compliant_educational_record_encryption',
    access_control: 'role_based_educational_access_with_audit_trails',
    consent_management: 'granular_educational_data_consent_tracking'
  },
  performance_optimization: {
    lazy_loading: 'progressive_educational_content_loading',
    caching_strategy: 'educational_content_prioritized_caching',
    offline_capabilities: 'progressive_web_app_for_continuous_learning',
    real_time_sync: 'optimistic_updates_with_educational_conflict_resolution'
  }
}

// Good Example: Educational State Management with Privacy Protection ✅
interface EducationalState {
  // Course and Academic Structure
  courseData: {
    metadata: CourseMetadata;
    structure: CourseStructure;
    learningObjectives: LearningObjective[];
    assessmentPlan: AssessmentPlan;
    privacySettings: CoursePrivacySettings;
  };
  
  // Student Learning State (Privacy Protected)
  studentState: {
    profile: PrivacyProtectedStudentProfile;
    progress: LearningProgress;
    preferences: LearningPreferences;
    achievements: LearningAchievement[];
    accessibilityNeeds: AccessibilityRequirements;
  };
  
  // Educational Context and Environment
  educationalContext: {
    currentSession: LearningSession;
    activeCollaboration: CollaborationSession[];
    pendingAssessments: Assessment[];
    learningAnalytics: PrivacyPreservingAnalytics;
    institutionalPolicies: InstitutionalPolicies;
  };
  
  // Application State and Performance
  applicationState: {
    connectivity: ConnectivityState;
    offlineCapabilities: OfflineCapabilities;
    performanceMetrics: EducationalPerformanceMetrics;
    errorRecovery: ErrorRecoveryState;
    accessibilityEnhancements: AccessibilityState;
  };
}

// Educational State Management with Redux Toolkit and Privacy Protection
const educationalStateSlice = createSlice({
  name: 'educational',
  initialState: {
    courseData: null,
    studentState: null,
    educationalContext: null,
    applicationState: {
      loading: false,
      error: null,
      offlineMode: false,
      accessibilityMode: 'standard'
    }
  } as EducationalState,
  reducers: {
    // Privacy-preserving course data management
    setCourseData: (state, action: PayloadAction<{
      courseData: CourseMetadata;
      privacyLevel: PrivacyLevel;
      encryptionRequired: boolean;
    }>) => {
      const { courseData, privacyLevel, encryptionRequired } = action.payload;
      
      // Apply privacy protection based on FERPA/COPPA requirements
      state.courseData = {
        metadata: privacyLevel === 'high' 
          ? sanitizeEducationalData(courseData, 'ferpa_compliant')
          : courseData,
        structure: courseData.structure,
        learningObjectives: courseData.learningObjectives,
        assessmentPlan: sanitizeAssessmentData(courseData.assessmentPlan, privacyLevel),
        privacySettings: {
          dataMinimization: true,
          ferpaCompliance: true,
          coppaApplicable: courseData.hasMinorStudents,
          encryptionRequired,
          auditTrail: true
        }
      };
    },

    // Student progress tracking with differential privacy
    updateStudentProgress: (state, action: PayloadAction<{
      progressUpdate: LearningProgressUpdate;
      privacyPreserving: boolean;
      educationalPurpose: boolean;
    }>) => {
      const { progressUpdate, privacyPreserving, educationalPurpose } = action.payload;
      
      if (!educationalPurpose || !state.studentState) return;
      
      // Apply differential privacy for learning analytics
      const privacyProtectedUpdate = privacyPreserving
        ? applyDifferentialPrivacy(progressUpdate, state.courseData.privacySettings)
        : progressUpdate;
      
      state.studentState.progress = {
        ...state.studentState.progress,
        ...privacyProtectedUpdate,
        lastUpdated: new Date().toISOString(),
        privacyLevel: privacyPreserving ? 'high' : 'standard'
      };
      
      // Trigger learning analytics with privacy protection
      state.educationalContext.learningAnalytics = updateLearningAnalytics(
        state.educationalContext.learningAnalytics,
        privacyProtectedUpdate,
        { privacyPreserving: true, aggregationOnly: true }
      );
    },

    // Accessibility enhancement state management
    updateAccessibilitySettings: (state, action: PayloadAction<{
      accessibilityRequirements: AccessibilityRequirements;
      wcagLevel: WCAGLevel;
      assistiveTechnology: AssistiveTechnologySettings;
    }>) => {
      const { accessibilityRequirements, wcagLevel, assistiveTechnology } = action.payload;
      
      state.studentState.accessibilityNeeds = accessibilityRequirements;
      state.applicationState.accessibilityEnhancements = {
        wcagLevel,
        assistiveTechnology,
        enhancedNavigation: true,
        screenReaderOptimized: true,
        keyboardNavigationEnhanced: true,
        colorContrastCompliant: wcagLevel === 'AAA',
        textScalingSupported: true
      };
    },

    // Real-time collaboration state with educational privacy
    updateCollaborationState: (state, action: PayloadAction<{
      collaborationUpdate: CollaborationUpdate;
      privacySettings: CollaborationPrivacySettings;
      educationalContext: EducationalCollaborationContext;
    }>) => {
      const { collaborationUpdate, privacySettings, educationalContext } = action.payload;
      
      // Ensure educational collaboration privacy
      if (!privacySettings.educationalPurposeOnly || 
          !privacySettings.ferpaCompliant) {
        return;
      }
      
      const existingCollaboration = state.educationalContext.activeCollaboration
        .find(collab => collab.sessionId === collaborationUpdate.sessionId);
      
      if (existingCollaboration) {
        // Update existing collaboration with privacy protection
        Object.assign(existingCollaboration, {
          ...collaborationUpdate,
          privacyProtected: true,
          educationalContext,
          lastUpdated: new Date().toISOString()
        });
      } else {
        // Add new collaboration session
        state.educationalContext.activeCollaboration.push({
          ...collaborationUpdate,
          privacySettings,
          educationalContext,
          createdAt: new Date().toISOString(),
          ferpaCompliant: true
        });
      }
    },

    // Offline learning capabilities management
    updateOfflineCapabilities: (state, action: PayloadAction<{
      offlineContent: OfflineEducationalContent;
      syncStrategy: OfflineSyncStrategy;
      priorityLevel: EducationalContentPriority;
    }>) => {
      const { offlineContent, syncStrategy, priorityLevel } = action.payload;
      
      state.applicationState.offlineCapabilities = {
        availableContent: offlineContent,
        syncStrategy,
        priorityLevel,
        lastSync: new Date().toISOString(),
        storageOptimized: true,
        educationalContentPrioritized: true,
        assessmentCapabilitiesOffline: offlineContent.includesAssessments,
        collaborationCapabilitiesOffline: false // Collaboration requires connectivity
      };
      
      state.applicationState.connectivity = {
        isOnline: navigator.onLine,
        offlineModeActive: !navigator.onLine,
        syncPending: !navigator.onLine && state.studentState?.progress?.pendingUpdates?.length > 0,
        educationalContinuityMaintained: true
      };
    }
  },
  extraReducers: (builder) => {
    builder
      // Handle educational data loading with privacy protection
      .addCase(loadEducationalDataAsync.pending, (state) => {
        state.applicationState.loading = true;
        state.applicationState.error = null;
      })
      .addCase(loadEducationalDataAsync.fulfilled, (state, action) => {
        state.applicationState.loading = false;
        
        // Apply privacy protection to loaded data
        const { courseData, studentData, privacyRequirements } = action.payload;
        
        state.courseData = applyEducationalPrivacyProtection(
          courseData, 
          privacyRequirements
        );
        
        state.studentState = sanitizeStudentData(
          studentData, 
          privacyRequirements.ferpaCompliance,
          privacyRequirements.coppaApplicable
        );
      })
      .addCase(loadEducationalDataAsync.rejected, (state, action) => {
        state.applicationState.loading = false;
        state.applicationState.error = {
          message: action.error.message,
          educationalImpact: 'learning_continuity_affected',
          recoveryOptions: [
            'retry_with_cached_content',
            'offline_mode_activation',
            'instructor_notification'
          ]
        };
      });
  }
});

// Privacy-preserving educational data selectors
export const selectEducationalData = createSelector(
  [(state: RootState) => state.educational],
  (educational) => ({
    courseMetadata: educational.courseData?.metadata,
    learningObjectives: educational.courseData?.learningObjectives,
    studentProgress: educational.studentState?.progress,
    privacyProtected: true,
    ferpaCompliant: educational.courseData?.privacySettings?.ferpaCompliance
  })
);

export const selectAccessibilityEnhancedData = createSelector(
  [(state: RootState) => state.educational],
  (educational) => ({
    accessibilityRequirements: educational.studentState?.accessibilityNeeds,
    enhancementSettings: educational.applicationState?.accessibilityEnhancements,
    wcagCompliant: educational.applicationState?.accessibilityEnhancements?.wcagLevel === 'AA',
    assistiveTechnologySupported: educational.applicationState?.accessibilityEnhancements?.assistiveTechnology
  })
);

// Bad Example: Generic State Without Educational Context ❌
const genericSlice = createSlice({
  name: 'generic',
  initialState: {
    data: null,
    loading: false
  },
  reducers: {
    setData: (state, action) => {
      state.data = action.payload;
    }
  }
});
```

## Conclusion

This comprehensive React best practices guide provides educational-focused development patterns with FERPA/COPPA compliance, accessibility enhancement, and learning-centered architecture. The framework emphasizes:

### ✅ Key React Principles for Educational Development
- **Learning-Centered Architecture**: All components designed around educational outcomes and student success
- **Privacy-First Development**: Built-in FERPA/COPPA compliance with data minimization and consent management
- **Accessibility Excellence**: WCAG 2.1 AA compliance with AAA targets and assistive technology support
- **Educational Performance**: Optimized for learning experiences with offline capabilities and real-time collaboration
- **Academic Integrity**: Secure assessment integration with educational ethics and plagiarism prevention

### 🎯 Implementation Standards
- **Component Design**: Hierarchical educational structure with learning objective alignment
- **State Management**: Privacy-preserving educational data with differential privacy for analytics
- **Accessibility Integration**: Universal design principles with educational context awareness
- **Performance Optimization**: Educational content prioritization with progressive enhancement
- **Privacy Protection**: FERPA/COPPA compliance by design with granular consent management

This guide ensures React development for educational applications maintains the highest standards for student privacy, accessibility, and learning effectiveness. 