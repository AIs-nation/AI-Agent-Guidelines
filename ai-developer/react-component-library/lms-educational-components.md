# LMS React Component Library: Educational Components Guide

## Table of Contents
1. [Educational Component Architecture](#educational-component-architecture)
2. [Learning Progress Components](#learning-progress-components)
3. [Interactive Learning Elements](#interactive-learning-elements)
4. [Assessment & Quiz Components](#assessment--quiz-components)
5. [Content Presentation Components](#content-presentation-components)
6. [Navigation & Course Structure](#navigation--course-structure)
7. [Educational Media Components](#educational-media-components)
8. [Accessibility-First Components](#accessibility-first-components)
9. [Performance Optimization](#performance-optimization)
10. [Testing Educational Components](#testing-educational-components)

## Educational Component Architecture

### 1. Educational Design System Foundation

**✅ Good: Educational Component Architecture**
```typescript
// Educational component library foundation
import React, { createContext, useContext, useState, useEffect } from 'react';
import { AccessibilityProvider } from './accessibility/AccessibilityProvider';
import { LearningAnalyticsProvider } from './analytics/LearningAnalyticsProvider';

// Educational context for learning-specific data
interface EducationalContextType {
  learnerProfile: LearnerProfile;
  currentCourse: CourseData;
  learningPreferences: LearningPreferences;
  progressTracker: ProgressTracker;
  accessibilitySettings: AccessibilitySettings;
}

const EducationalContext = createContext<EducationalContextType | null>(null);

// Educational component wrapper with learning-specific features
export const EducationalProvider: React.FC<{
  children: React.ReactNode;
  learnerProfile: LearnerProfile;
  courseData: CourseData;
}> = ({ children, learnerProfile, courseData }) => {
  const [progressTracker] = useState(new ProgressTracker(learnerProfile.id));
  const [learningPreferences, setLearningPreferences] = useState(
    learnerProfile.preferences
  );
  const [accessibilitySettings, setAccessibilitySettings] = useState(
    learnerProfile.accessibilitySettings
  );

  const contextValue: EducationalContextType = {
    learnerProfile,
    currentCourse: courseData,
    learningPreferences,
    progressTracker,
    accessibilitySettings
  };

  return (
    <EducationalContext.Provider value={contextValue}>
      <AccessibilityProvider settings={accessibilitySettings}>
        <LearningAnalyticsProvider>
          {children}
        </LearningAnalyticsProvider>
      </AccessibilityProvider>
    </EducationalContext.Provider>
  );
};

// Hook for accessing educational context
export const useEducationalContext = () => {
  const context = useContext(EducationalContext);
  if (!context) {
    throw new Error('useEducationalContext must be used within EducationalProvider');
  }
  return context;
};

// Base educational component with common learning features
export interface BaseEducationalProps {
  id: string;
  trackingEnabled?: boolean;
  accessibilityLevel?: 'AA' | 'AAA';
  learningObjectives?: string[];
  cognitiveLoad?: 'low' | 'medium' | 'high';
  adaptiveContent?: boolean;
  className?: string;
  children?: React.ReactNode;
}

export const BaseEducationalComponent: React.FC<BaseEducationalProps> = ({
  id,
  trackingEnabled = true,
  accessibilityLevel = 'AA',
  learningObjectives = [],
  cognitiveLoad = 'medium',
  adaptiveContent = false,
  className,
  children,
  ...props
}) => {
  const { progressTracker, learningPreferences } = useEducationalContext();
  const [interactionStartTime] = useState(Date.now());

  useEffect(() => {
    if (trackingEnabled) {
      progressTracker.trackComponentView(id, learningObjectives);
    }

    return () => {
      if (trackingEnabled) {
        const timeSpent = Date.now() - interactionStartTime;
        progressTracker.trackComponentInteraction(id, timeSpent);
      }
    };
  }, [id, trackingEnabled, progressTracker, interactionStartTime, learningObjectives]);

  const adaptedClassName = `
    educational-component
    accessibility-${accessibilityLevel}
    cognitive-load-${cognitiveLoad}
    ${adaptiveContent ? 'adaptive-content' : ''}
    ${className || ''}
  `.trim();

  return (
    <div
      id={id}
      className={adaptedClassName}
      data-learning-objectives={learningObjectives.join(',')}
      {...props}
    >
      {children}
    </div>
  );
};
```

### 2. Learning Progress Components

**✅ Good: Comprehensive Progress Tracking Components**
```typescript
// Learning progress indicator with educational features
interface LearningProgressProps extends BaseEducationalProps {
  courseId: string;
  sectionId?: string;
  totalSections: number;
  completedSections: number;
  showTimeEstimate?: boolean;
  showMilestones?: boolean;
  animateProgress?: boolean;
  onMilestoneReached?: (milestone: number) => void;
}

export const LearningProgressIndicator: React.FC<LearningProgressProps> = ({
  courseId,
  sectionId,
  totalSections,
  completedSections,
  showTimeEstimate = true,
  showMilestones = true,
  animateProgress = true,
  onMilestoneReached,
  ...baseProps
}) => {
  const { progressTracker, learningPreferences } = useEducationalContext();
  const [previousProgress, setPreviousProgress] = useState(completedSections);
  const [celebratingMilestone, setCelebratingMilestone] = useState(false);

  const progressPercentage = Math.round((completedSections / totalSections) * 100);
  const estimatedTimeRemaining = progressTracker.getEstimatedTimeRemaining(
    courseId,
    totalSections - completedSections
  );

  // Handle milestone celebrations
  useEffect(() => {
    const milestones = [25, 50, 75, 100];
    const currentMilestone = milestones.find(m => 
      progressPercentage >= m && (previousProgress / totalSections) * 100 < m
    );

    if (currentMilestone && onMilestoneReached) {
      setCelebratingMilestone(true);
      onMilestoneReached(currentMilestone);
      
      setTimeout(() => setCelebratingMilestone(false), 3000);
    }

    setPreviousProgress(completedSections);
  }, [completedSections, previousProgress, totalSections, progressPercentage, onMilestoneReached]);

  return (
    <BaseEducationalComponent
      {...baseProps}
      id={`progress-${courseId}`}
      cognitiveLoad="low"
    >
      <div className="learning-progress-container">
        {/* Accessible progress bar */}
        <div
          role="progressbar"
          aria-labelledby="progress-label"
          aria-describedby="progress-description"
          aria-valuenow={progressPercentage}
          aria-valuemin={0}
          aria-valuemax={100}
          className={`progress-bar ${animateProgress ? 'animated' : ''} ${
            celebratingMilestone ? 'celebrating' : ''
          }`}
        >
          <div
            className="progress-fill"
            style={{ width: `${progressPercentage}%` }}
          />
          
          {/* Milestone markers */}
          {showMilestones && (
            <div className="milestone-markers">
              {[25, 50, 75].map(milestone => (
                <div
                  key={milestone}
                  className={`milestone ${
                    progressPercentage >= milestone ? 'reached' : ''
                  }`}
                  style={{ left: `${milestone}%` }}
                  aria-label={`${milestone}% milestone ${
                    progressPercentage >= milestone ? 'completed' : ''
                  }`}
                />
              ))}
            </div>
          )}
        </div>

        {/* Progress information */}
        <div className="progress-info">
          <div id="progress-label" className="progress-label">
            Course Progress
          </div>
          <div id="progress-description" className="progress-description">
            {completedSections} of {totalSections} sections completed ({progressPercentage}%)
            {showTimeEstimate && estimatedTimeRemaining && (
              <span className="time-estimate">
                • Estimated time remaining: {estimatedTimeRemaining}
              </span>
            )}
          </div>
        </div>

        {/* Celebration animation */}
        {celebratingMilestone && (
          <div className="milestone-celebration" aria-live="polite">
            🎉 Milestone reached! Keep up the great work!
          </div>
        )}
      </div>
    </BaseEducationalComponent>
  );
};

// Section progress tracker
interface SectionProgressProps extends BaseEducationalProps {
  sectionTitle: string;
  subsections: SubsectionData[];
  currentSubsection?: number;
  onSubsectionComplete?: (subsectionId: string) => void;
}

export const SectionProgressTracker: React.FC<SectionProgressProps> = ({
  sectionTitle,
  subsections,
  currentSubsection = 0,
  onSubsectionComplete,
  ...baseProps
}) => {
  const [completedSubsections, setCompletedSubsections] = useState<Set<string>>(
    new Set()
  );

  const handleSubsectionComplete = (subsectionId: string) => {
    setCompletedSubsections(prev => new Set([...prev, subsectionId]));
    onSubsectionComplete?.(subsectionId);
  };

  return (
    <BaseEducationalComponent
      {...baseProps}
      id={`section-progress-${sectionTitle.replace(/\s+/g, '-').toLowerCase()}`}
    >
      <nav aria-label={`${sectionTitle} progress`} className="section-progress">
        <h3 className="section-title">{sectionTitle}</h3>
        <ol className="subsection-list">
          {subsections.map((subsection, index) => {
            const isCompleted = completedSubsections.has(subsection.id);
            const isCurrent = index === currentSubsection;
            const isAccessible = index <= currentSubsection || isCompleted;

            return (
              <li
                key={subsection.id}
                className={`subsection-item ${isCompleted ? 'completed' : ''} ${
                  isCurrent ? 'current' : ''
                } ${isAccessible ? 'accessible' : 'locked'}`}
              >
                <button
                  className="subsection-button"
                  disabled={!isAccessible}
                  aria-current={isCurrent ? 'step' : undefined}
                  onClick={() => {
                    if (isAccessible) {
                      // Handle navigation to subsection
                    }
                  }}
                >
                  <span className="subsection-icon">
                    {isCompleted ? '✓' : isCurrent ? '▶' : index + 1}
                  </span>
                  <span className="subsection-title">{subsection.title}</span>
                  <span className="subsection-duration">
                    {subsection.estimatedDuration}
                  </span>
                </button>
              </li>
            );
          })}
        </ol>
      </nav>
    </BaseEducationalComponent>
  );
};
```

### 3. Interactive Learning Elements

**✅ Good: Educational Interactive Components**
```typescript
// Interactive code editor for programming courses
interface CodeEditorProps extends BaseEducationalProps {
  initialCode: string;
  language: string;
  expectedOutput?: string;
  hints?: string[];
  onCodeChange?: (code: string) => void;
  onRunCode?: (code: string) => Promise<any>;
  readOnly?: boolean;
}

export const EducationalCodeEditor: React.FC<CodeEditorProps> = ({
  initialCode,
  language,
  expectedOutput,
  hints = [],
  onCodeChange,
  onRunCode,
  readOnly = false,
  ...baseProps
}) => {
  const [code, setCode] = useState(initialCode);
  const [output, setOutput] = useState('');
  const [isRunning, setIsRunning] = useState(false);
  const [showHints, setShowHints] = useState(false);
  const [currentHint, setCurrentHint] = useState(0);

  const handleCodeChange = (newCode: string) => {
    setCode(newCode);
    onCodeChange?.(newCode);
  };

  const handleRunCode = async () => {
    if (!onRunCode) return;
    
    setIsRunning(true);
    try {
      const result = await onRunCode(code);
      setOutput(result.output || result.toString());
    } catch (error) {
      setOutput(`Error: ${error.message}`);
    } finally {
      setIsRunning(false);
    }
  };

  return (
    <BaseEducationalComponent
      {...baseProps}
      id={`code-editor-${language}`}
      cognitiveLoad="high"
    >
      <div className="educational-code-editor">
        {/* Editor header */}
        <div className="editor-header">
          <span className="language-indicator">{language}</span>
          <div className="editor-controls">
            {hints.length > 0 && (
              <button
                className="hint-button"
                onClick={() => setShowHints(!showHints)}
                aria-expanded={showHints}
                aria-controls="hints-panel"
              >
                💡 Hints ({hints.length})
              </button>
            )}
            {onRunCode && (
              <button
                className="run-button"
                onClick={handleRunCode}
                disabled={isRunning || readOnly}
                aria-describedby="run-button-desc"
              >
                {isRunning ? '⏳ Running...' : '▶ Run Code'}
              </button>
            )}
          </div>
        </div>

        {/* Code editor */}
        <div className="editor-container">
          <textarea
            className="code-textarea"
            value={code}
            onChange={(e) => handleCodeChange(e.target.value)}
            readOnly={readOnly}
            aria-label={`${language} code editor`}
            aria-describedby="editor-instructions"
            spellCheck={false}
            autoComplete="off"
            data-language={language}
          />
          <div id="editor-instructions" className="sr-only">
            {readOnly 
              ? `Read-only ${language} code example`
              : `Editable ${language} code editor. Press Ctrl+Enter to run code.`
            }
          </div>
        </div>

        {/* Hints panel */}
        {showHints && hints.length > 0 && (
          <div id="hints-panel" className="hints-panel" role="region" aria-label="Code hints">
            <div className="hint-navigation">
              <button
                onClick={() => setCurrentHint(Math.max(0, currentHint - 1))}
                disabled={currentHint === 0}
                aria-label="Previous hint"
              >
                ←
              </button>
              <span className="hint-counter">
                Hint {currentHint + 1} of {hints.length}
              </span>
              <button
                onClick={() => setCurrentHint(Math.min(hints.length - 1, currentHint + 1))}
                disabled={currentHint === hints.length - 1}
                aria-label="Next hint"
              >
                →
              </button>
            </div>
            <div className="hint-content" aria-live="polite">
              {hints[currentHint]}
            </div>
          </div>
        )}

        {/* Output panel */}
        {output && (
          <div className="output-panel" role="region" aria-label="Code output">
            <div className="output-header">Output:</div>
            <pre className="output-content" aria-live="polite">
              {output}
            </pre>
            {expectedOutput && (
              <div className="expected-output">
                <div className="expected-header">Expected Output:</div>
                <pre className="expected-content">{expectedOutput}</pre>
                <div 
                  className={`match-indicator ${
                    output.trim() === expectedOutput.trim() ? 'match' : 'no-match'
                  }`}
                  aria-live="polite"
                >
                  {output.trim() === expectedOutput.trim() 
                    ? '✓ Output matches expected result!' 
                    : '⚠ Output does not match expected result'
                  }
                </div>
              </div>
            )}
          </div>
        )}
      </div>
    </BaseEducationalComponent>
  );
};

// Interactive concept explorer
interface ConceptExplorerProps extends BaseEducationalProps {
  concept: ConceptData;
  relatedConcepts?: ConceptData[];
  onConceptSelect?: (conceptId: string) => void;
}

export const InteractiveConceptExplorer: React.FC<ConceptExplorerProps> = ({
  concept,
  relatedConcepts = [],
  onConceptSelect,
  ...baseProps
}) => {
  const [selectedDefinition, setSelectedDefinition] = useState(0);
  const [showExamples, setShowExamples] = useState(false);
  const [explorationPath, setExplorationPath] = useState<string[]>([concept.id]);

  const handleConceptNavigation = (conceptId: string) => {
    setExplorationPath(prev => [...prev, conceptId]);
    onConceptSelect?.(conceptId);
  };

  return (
    <BaseEducationalComponent
      {...baseProps}
      id={`concept-explorer-${concept.id}`}
      cognitiveLoad="medium"
    >
      <div className="concept-explorer">
        {/* Concept header */}
        <header className="concept-header">
          <h3 className="concept-title">{concept.title}</h3>
          <div className="concept-metadata">
            <span className="difficulty-level">
              Difficulty: {concept.difficultyLevel}
            </span>
            <span className="estimated-time">
              Time: {concept.estimatedTime}
            </span>
          </div>
        </header>

        {/* Navigation breadcrumb */}
        {explorationPath.length > 1 && (
          <nav aria-label="Concept exploration path" className="exploration-breadcrumb">
            <ol>
              {explorationPath.map((conceptId, index) => (
                <li key={conceptId}>
                  {index < explorationPath.length - 1 ? (
                    <button
                      onClick={() => {
                        const newPath = explorationPath.slice(0, index + 1);
                        setExplorationPath(newPath);
                        onConceptSelect?.(conceptId);
                      }}
                    >
                      {conceptId}
                    </button>
                  ) : (
                    <span aria-current="page">{conceptId}</span>
                  )}
                </li>
              ))}
            </ol>
          </nav>
        )}

        {/* Concept definitions */}
        <section className="concept-definitions">
          <h4>Definitions</h4>
          <div className="definition-tabs" role="tablist">
            {concept.definitions.map((definition, index) => (
              <button
                key={index}
                role="tab"
                aria-selected={selectedDefinition === index}
                aria-controls={`definition-panel-${index}`}
                className={`definition-tab ${
                  selectedDefinition === index ? 'active' : ''
                }`}
                onClick={() => setSelectedDefinition(index)}
              >
                {definition.type}
              </button>
            ))}
          </div>
          <div
            id={`definition-panel-${selectedDefinition}`}
            role="tabpanel"
            className="definition-content"
          >
            {concept.definitions[selectedDefinition]?.content}
          </div>
        </section>

        {/* Examples section */}
        <section className="concept-examples">
          <button
            className="examples-toggle"
            onClick={() => setShowExamples(!showExamples)}
            aria-expanded={showExamples}
            aria-controls="examples-content"
          >
            📚 Examples ({concept.examples?.length || 0})
          </button>
          {showExamples && concept.examples && (
            <div id="examples-content" className="examples-content">
              {concept.examples.map((example, index) => (
                <div key={index} className="example-item">
                  <h5 className="example-title">{example.title}</h5>
                  <div className="example-description">{example.description}</div>
                  {example.code && (
                    <pre className="example-code">
                      <code>{example.code}</code>
                    </pre>
                  )}
                </div>
              ))}
            </div>
          )}
        </section>

        {/* Related concepts */}
        {relatedConcepts.length > 0 && (
          <section className="related-concepts">
            <h4>Related Concepts</h4>
            <div className="concept-grid">
              {relatedConcepts.map((relatedConcept) => (
                <button
                  key={relatedConcept.id}
                  className="related-concept-card"
                  onClick={() => handleConceptNavigation(relatedConcept.id)}
                  aria-describedby={`concept-desc-${relatedConcept.id}`}
                >
                  <div className="related-concept-title">
                    {relatedConcept.title}
                  </div>
                  <div 
                    id={`concept-desc-${relatedConcept.id}`}
                    className="related-concept-description"
                  >
                    {relatedConcept.shortDescription}
                  </div>
                </button>
              ))}
            </div>
          </section>
        )}
      </div>
    </BaseEducationalComponent>
  );
};
```

This comprehensive React component library guide provides reusable, accessible, and educationally-optimized components specifically designed for learning management systems, ensuring effective learning experiences while maintaining high code quality and performance standards. 