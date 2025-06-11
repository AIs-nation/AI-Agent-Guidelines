# LMS Comprehensive Testing Framework: Educational Platform Testing Guide

## Table of Contents
1. [Educational Platform Testing Overview](#educational-platform-testing-overview)
2. [Unit Testing for LMS Components](#unit-testing-for-lms-components)
3. [Integration Testing for Learning Workflows](#integration-testing-for-learning-workflows)
4. [End-to-End Testing for Educational Journeys](#end-to-end-testing-for-educational-journeys)
5. [Performance Testing for Learning Analytics](#performance-testing-for-learning-analytics)
6. [Security Testing for Educational Data](#security-testing-for-educational-data)
7. [Accessibility Testing for Inclusive Learning](#accessibility-testing-for-inclusive-learning)
8. [Automated Testing Pipeline](#automated-testing-pipeline)

## Educational Platform Testing Overview

### 1. LMS-Specific Testing Challenges

**Unique Testing Requirements for Educational Platforms:**
```javascript
// ✅ Good: Comprehensive test planning for LMS features
const LMS_TESTING_DOMAINS = {
  EDUCATIONAL_WORKFLOWS: {
    critical: ['course_enrollment', 'lesson_progression', 'assessment_completion'],
    complex: ['adaptive_learning', 'prerequisite_validation', 'grade_calculation'],
    userTypes: ['student', 'instructor', 'admin', 'guest']
  },
  
  CONTENT_MANAGEMENT: {
    generation: ['ai_course_creation', 'content_validation', 'media_processing'],
    delivery: ['content_streaming', 'offline_access', 'mobile_optimization'],
    analytics: ['engagement_tracking', 'completion_rates', 'learning_analytics']
  },
  
  COMPLIANCE_TESTING: {
    privacy: ['ferpa_compliance', 'data_encryption', 'consent_management'],
    accessibility: ['wcag_compliance', 'screen_reader_support', 'keyboard_navigation'],
    performance: ['load_handling', 'concurrent_users', 'mobile_performance']
  },
  
  INTEGRATION_TESTING: {
    external: ['lti_integration', 'sso_providers', 'grade_passback'],
    internal: ['frontend_backend', 'database_consistency', 'cache_synchronization']
  }
};

// Testing strategy framework
class LMSTestingStrategy {
  constructor() {
    this.testPyramid = {
      unit: { percentage: 70, focus: 'individual_components' },
      integration: { percentage: 20, focus: 'workflow_interactions' },
      e2e: { percentage: 10, focus: 'complete_user_journeys' }
    };
  }

  planTestSuite(feature, userType, criticalityLevel) {
    const testPlan = {
      unit: this.generateUnitTests(feature, criticalityLevel),
      integration: this.generateIntegrationTests(feature, userType),
      e2e: this.generateE2ETests(feature, userType),
      performance: this.generatePerformanceTests(feature),
      accessibility: this.generateA11yTests(feature)
    };
    
    return testPlan;
  }
}
```

### 2. Testing Environment Setup for LMS

**✅ Good: LMS Testing Environment Configuration**
```javascript
// jest.config.js - LMS-specific Jest configuration
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/src/test/setup.js'],
  testMatch: [
    '<rootDir>/src/**/__tests__/**/*.{js,jsx,ts,tsx}',
    '<rootDir>/src/**/*.{spec,test}.{js,jsx,ts,tsx}'
  ],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '^@components/(.*)$': '<rootDir>/src/components/$1',
    '^@hooks/(.*)$': '<rootDir>/src/hooks/$1',
    '^@utils/(.*)$': '<rootDir>/src/utils/$1'
  },
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/test/**',
    '!src/stories/**'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    },
    // Higher coverage for critical LMS components
    './src/components/Course/': {
      branches: 90,
      functions: 90,
      lines: 90,
      statements: 90
    },
    './src/components/Assessment/': {
      branches: 95,
      functions: 95,
      lines: 95,
      statements: 95
    }
  }
};

// src/test/setup.js - LMS testing utilities
import '@testing-library/jest-dom';
import { server } from './mocks/server';
import { LMSTestUtils } from './utils/lms-test-utils';

// Mock global LMS context
global.LMSTestUtils = LMSTestUtils;

// Setup MSW for API mocking
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// Global test data for LMS
global.testData = {
  mockUser: {
    id: 'user-123',
    name: 'Test Student',
    email: 'student@test.edu',
    role: 'STUDENT'
  },
  mockCourse: {
    id: 'course-456',
    title: 'Introduction to React',
    difficulty: 'BEGINNER',
    estimatedDuration: 8,
    lessons: []
  },
  mockAssessment: {
    id: 'assessment-789',
    title: 'React Fundamentals Quiz',
    questions: [],
    timeLimit: 30,
    passingScore: 70
  }
};
```

## Unit Testing for LMS Components

### 1. Course Components Testing

**✅ Good: Comprehensive Course Component Testing**
```javascript
// src/components/Course/__tests__/CourseCard.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { CourseCard } from '../CourseCard';
import { LMSTestProvider } from '../../../test/utils/LMSTestProvider';

describe('CourseCard Component', () => {
  const mockCourse = {
    id: 'course-123',
    title: 'Introduction to JavaScript',
    description: 'Learn the fundamentals of JavaScript programming',
    difficulty: 'BEGINNER',
    estimatedDuration: 10,
    thumbnail: 'https://example.com/js-thumbnail.jpg',
    enrolledStudents: 1250,
    rating: 4.8,
    instructor: 'John Doe',
    tags: ['programming', 'javascript', 'web-development']
  };

  const renderCourseCard = (props = {}) => {
    return render(
      <LMSTestProvider>
        <CourseCard course={mockCourse} {...props} />
      </LMSTestProvider>
    );
  };

  // ✅ Good: Test core functionality
  test('renders course information correctly', () => {
    renderCourseCard();
    
    expect(screen.getByText('Introduction to JavaScript')).toBeInTheDocument();
    expect(screen.getByText(/Learn the fundamentals/)).toBeInTheDocument();
    expect(screen.getByText('BEGINNER')).toBeInTheDocument();
    expect(screen.getByText('10 hours')).toBeInTheDocument();
    expect(screen.getByText('1,250 students')).toBeInTheDocument();
    expect(screen.getByText('4.8')).toBeInTheDocument();
    expect(screen.getByText('John Doe')).toBeInTheDocument();
  });

  // ✅ Good: Test user interactions
  test('handles enrollment click for unenrolled student', async () => {
    const mockOnEnroll = jest.fn();
    renderCourseCard({ onEnroll: mockOnEnroll, isEnrolled: false });
    
    const enrollButton = screen.getByRole('button', { name: /enroll now/i });
    fireEvent.click(enrollButton);
    
    await waitFor(() => {
      expect(mockOnEnroll).toHaveBeenCalledWith('course-123');
    });
  });

  // ✅ Good: Test different states
  test('shows continue learning for enrolled student', () => {
    renderCourseCard({ isEnrolled: true, progress: 45 });
    
    expect(screen.getByText('Continue Learning')).toBeInTheDocument();
    expect(screen.getByText('45% Complete')).toBeInTheDocument();
    expect(screen.getByRole('progressbar')).toBeInTheDocument();
  });

  // ✅ Good: Test accessibility
  test('has proper accessibility attributes', () => {
    renderCourseCard();
    
    const courseCard = screen.getByRole('article');
    expect(courseCard).toHaveAttribute('aria-label', expect.stringContaining('Introduction to JavaScript'));
    
    const enrollButton = screen.getByRole('button', { name: /enroll now/i });
    expect(enrollButton).toHaveAttribute('aria-describedby');
    
    const thumbnail = screen.getByRole('img');
    expect(thumbnail).toHaveAttribute('alt', 'Introduction to JavaScript course thumbnail');
  });

  // ✅ Good: Test error states
  test('handles missing thumbnail gracefully', () => {
    const courseWithoutThumbnail = { ...mockCourse, thumbnail: null };
    renderCourseCard({ course: courseWithoutThumbnail });
    
    const placeholderImage = screen.getByRole('img');
    expect(placeholderImage).toHaveAttribute('src', expect.stringContaining('placeholder'));
  });

  // ✅ Good: Test responsive behavior
  test('adapts to mobile viewport', () => {
    // Mock mobile viewport
    Object.defineProperty(window, 'innerWidth', {
      writable: true,
      configurable: true,
      value: 375,
    });
    
    renderCourseCard();
    
    const courseCard = screen.getByRole('article');
    expect(courseCard).toHaveClass('course-card--mobile');
  });
});
```

### 2. Assessment Components Testing

**✅ Good: Assessment Component Testing**
```javascript
// src/components/Assessment/__tests__/QuizQuestion.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { QuizQuestion } from '../QuizQuestion';
import { LMSTestProvider } from '../../../test/utils/LMSTestProvider';

describe('QuizQuestion Component', () => {
  const mockQuestion = {
    id: 'q1',
    type: 'MULTIPLE_CHOICE',
    question: 'What is the correct way to declare a variable in JavaScript?',
    options: [
      { id: 'a', text: 'var myVar = 5;', correct: true },
      { id: 'b', text: 'variable myVar = 5;', correct: false },
      { id: 'c', text: 'v myVar = 5;', correct: false },
      { id: 'd', text: 'declare myVar = 5;', correct: false }
    ],
    explanation: 'Variables in JavaScript are declared using var, let, or const keywords.',
    points: 10,
    timeLimit: 60
  };

  const renderQuizQuestion = (props = {}) => {
    return render(
      <LMSTestProvider>
        <QuizQuestion 
          question={mockQuestion}
          questionNumber={1}
          totalQuestions={5}
          onAnswer={jest.fn()}
          {...props}
        />
      </LMSTestProvider>
    );
  };

  // ✅ Good: Test question rendering
  test('renders question content correctly', () => {
    renderQuizQuestion();
    
    expect(screen.getByText('Question 1 of 5')).toBeInTheDocument();
    expect(screen.getByText('What is the correct way to declare a variable in JavaScript?')).toBeInTheDocument();
    expect(screen.getByText('var myVar = 5;')).toBeInTheDocument();
    expect(screen.getByText('variable myVar = 5;')).toBeInTheDocument();
    expect(screen.getByText('10 points')).toBeInTheDocument();
  });

  // ✅ Good: Test answer selection
  test('handles answer selection correctly', async () => {
    const mockOnAnswer = jest.fn();
    renderQuizQuestion({ onAnswer: mockOnAnswer });
    
    const correctOption = screen.getByRole('radio', { name: /var myVar = 5;/ });
    fireEvent.click(correctOption);
    
    expect(correctOption).toBeChecked();
  });

  // ✅ Good: Test timer functionality
  test('displays and counts down timer', async () => {
    renderQuizQuestion({ showTimer: true });
    
    expect(screen.getByText('01:00')).toBeInTheDocument();
    
    // Fast-forward timer
    jest.advanceTimersByTime(1000);
    
    await waitFor(() => {
      expect(screen.getByText('00:59')).toBeInTheDocument();
    });
  });

  // ✅ Good: Test different question types
  test('handles true/false questions', () => {
    const trueFalseQuestion = {
      ...mockQuestion,
      type: 'TRUE_FALSE',
      options: [
        { id: 'true', text: 'True', correct: true },
        { id: 'false', text: 'False', correct: false }
      ]
    };
    
    renderQuizQuestion({ question: trueFalseQuestion });
    
    expect(screen.getByRole('radio', { name: /true/i })).toBeInTheDocument();
    expect(screen.getByRole('radio', { name: /false/i })).toBeInTheDocument();
  });

  // ✅ Good: Test explanation display
  test('shows explanation after answer submission', async () => {
    const mockOnAnswer = jest.fn();
    renderQuizQuestion({ onAnswer: mockOnAnswer, showExplanation: true });
    
    expect(screen.getByText('Variables in JavaScript are declared using var, let, or const keywords.')).toBeInTheDocument();
    expect(screen.getByText('Correct Answer')).toBeInTheDocument();
  });

  // ✅ Good: Test accessibility
  test('has proper ARIA attributes for screen readers', () => {
    renderQuizQuestion();
    
    const questionContainer = screen.getByRole('group');
    expect(questionContainer).toHaveAttribute('aria-labelledby');
    
    const radioButtons = screen.getAllByRole('radio');
    radioButtons.forEach(radio => {
      expect(radio).toHaveAttribute('aria-describedby');
    });
  });
});
```

## Integration Testing for Learning Workflows

### 1. Course Enrollment Flow Testing

**✅ Good: End-to-End Enrollment Testing**
```javascript
// src/test/integration/course-enrollment.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { BrowserRouter } from 'react-router-dom';
import { CourseEnrollmentFlow } from '../../components/CourseEnrollmentFlow';
import { LMSTestProvider } from '../utils/LMSTestProvider';
import { server } from '../mocks/server';
import { rest } from 'msw';

describe('Course Enrollment Integration', () => {
  const mockCourse = {
    id: 'course-123',
    title: 'Advanced React Patterns',
    price: 99.99,
    prerequisites: ['course-101', 'course-102'],
    maxStudents: 100,
    currentEnrollment: 85
  };

  const renderEnrollmentFlow = () => {
    return render(
      <BrowserRouter>
        <LMSTestProvider>
          <CourseEnrollmentFlow courseId="course-123" />
        </LMSTestProvider>
      </BrowserRouter>
    );
  };

  // ✅ Good: Test complete enrollment workflow
  test('completes successful enrollment flow', async () => {
    // Mock successful API responses
    server.use(
      rest.get('/api/courses/course-123', (req, res, ctx) => {
        return res(ctx.json({ success: true, course: mockCourse }));
      }),
      rest.post('/api/enrollments', (req, res, ctx) => {
        return res(ctx.json({ success: true, enrollmentId: 'enroll-456' }));
      }),
      rest.post('/api/payments', (req, res, ctx) => {
        return res(ctx.json({ success: true, paymentId: 'payment-789' }));
      })
    );

    renderEnrollmentFlow();

    // Step 1: Course information display
    await waitFor(() => {
      expect(screen.getByText('Advanced React Patterns')).toBeInTheDocument();
    });

    // Step 2: Prerequisites check
    expect(screen.getByText('Prerequisites')).toBeInTheDocument();
    expect(screen.getByText('Prerequisites met ✓')).toBeInTheDocument();

    // Step 3: Payment processing
    const enrollButton = screen.getByRole('button', { name: /enroll now/i });
    fireEvent.click(enrollButton);

    // Fill payment form
    fireEvent.change(screen.getByLabelText(/card number/i), {
      target: { value: '4111111111111111' }
    });
    fireEvent.change(screen.getByLabelText(/expiry date/i), {
      target: { value: '12/25' }
    });
    fireEvent.change(screen.getByLabelText(/cvv/i), {
      target: { value: '123' }
    });

    const payButton = screen.getByRole('button', { name: /pay and enroll/i });
    fireEvent.click(payButton);

    // Step 4: Enrollment confirmation
    await waitFor(() => {
      expect(screen.getByText('Enrollment Successful!')).toBeInTheDocument();
      expect(screen.getByText('Welcome to Advanced React Patterns')).toBeInTheDocument();
    });

    // Step 5: Redirect to course
    const startLearningButton = screen.getByRole('button', { name: /start learning/i });
    expect(startLearningButton).toBeInTheDocument();
  });

  // ✅ Good: Test prerequisite validation
  test('blocks enrollment when prerequisites not met', async () => {
    const courseWithUnmetPrereqs = {
      ...mockCourse,
      prerequisites: ['course-101', 'course-102', 'course-103']
    };

    server.use(
      rest.get('/api/courses/course-123', (req, res, ctx) => {
        return res(ctx.json({ success: true, course: courseWithUnmetPrereqs }));
      }),
      rest.get('/api/users/current/prerequisites', (req, res, ctx) => {
        return res(ctx.json({ 
          success: true, 
          completed: ['course-101'], 
          missing: ['course-102', 'course-103'] 
        }));
      })
    );

    renderEnrollmentFlow();

    await waitFor(() => {
      expect(screen.getByText('Prerequisites not met')).toBeInTheDocument();
      expect(screen.getByText('Complete these courses first:')).toBeInTheDocument();
    });

    const enrollButton = screen.queryByRole('button', { name: /enroll now/i });
    expect(enrollButton).not.toBeInTheDocument();
  });

  // ✅ Good: Test capacity limits
  test('shows waitlist when course is full', async () => {
    const fullCourse = {
      ...mockCourse,
      currentEnrollment: 100,
      maxStudents: 100
    };

    server.use(
      rest.get('/api/courses/course-123', (req, res, ctx) => {
        return res(ctx.json({ success: true, course: fullCourse }));
      })
    );

    renderEnrollmentFlow();

    await waitFor(() => {
      expect(screen.getByText('Course Full')).toBeInTheDocument();
      expect(screen.getByText('Join Waitlist')).toBeInTheDocument();
    });
  });

  // ✅ Good: Test payment failure handling
  test('handles payment failure gracefully', async () => {
    server.use(
      rest.get('/api/courses/course-123', (req, res, ctx) => {
        return res(ctx.json({ success: true, course: mockCourse }));
      }),
      rest.post('/api/payments', (req, res, ctx) => {
        return res(ctx.status(400), ctx.json({ 
          success: false, 
          error: 'Payment declined' 
        }));
      })
    );

    renderEnrollmentFlow();

    await waitFor(() => {
      expect(screen.getByText('Advanced React Patterns')).toBeInTheDocument();
    });

    const enrollButton = screen.getByRole('button', { name: /enroll now/i });
    fireEvent.click(enrollButton);

    // Fill payment form
    fireEvent.change(screen.getByLabelText(/card number/i), {
      target: { value: '4000000000000002' } // Declined card
    });

    const payButton = screen.getByRole('button', { name: /pay and enroll/i });
    fireEvent.click(payButton);

    await waitFor(() => {
      expect(screen.getByText('Payment failed')).toBeInTheDocument();
      expect(screen.getByText('Please try a different payment method')).toBeInTheDocument();
    });
  });
});
```

### 2. Learning Progress Tracking Integration

**✅ Good: Progress Tracking Integration Testing**
```javascript
// src/test/integration/progress-tracking.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { LearningProgressTracker } from '../../components/LearningProgressTracker';
import { LMSTestProvider } from '../utils/LMSTestProvider';
import { server } from '../mocks/server';
import { rest } from 'msw';

describe('Learning Progress Tracking Integration', () => {
  const mockProgressData = {
    courseId: 'course-123',
    userId: 'user-456',
    overallProgress: 65,
    completedLessons: 13,
    totalLessons: 20,
    timeSpent: 420, // minutes
    lastAccessedLesson: 'lesson-13',
    achievements: ['first_lesson', 'halfway_point'],
    streakDays: 7
  };

  const renderProgressTracker = () => {
    return render(
      <LMSTestProvider>
        <LearningProgressTracker 
          courseId="course-123" 
          userId="user-456"
        />
      </LMSTestProvider>
    );
  };

  // ✅ Good: Test progress data loading and display
  test('loads and displays progress data correctly', async () => {
    server.use(
      rest.get('/api/progress/course-123/user-456', (req, res, ctx) => {
        return res(ctx.json({ success: true, progress: mockProgressData }));
      })
    );

    renderProgressTracker();

    await waitFor(() => {
      expect(screen.getByText('65% Complete')).toBeInTheDocument();
      expect(screen.getByText('13 of 20 lessons completed')).toBeInTheDocument();
      expect(screen.getByText('7 hours total')).toBeInTheDocument();
      expect(screen.getByText('7 day streak')).toBeInTheDocument();
    });

    // Check progress bar
    const progressBar = screen.getByRole('progressbar');
    expect(progressBar).toHaveAttribute('aria-valuenow', '65');
  });

  // ✅ Good: Test lesson completion tracking
  test('updates progress when lesson is completed', async () => {
    server.use(
      rest.get('/api/progress/course-123/user-456', (req, res, ctx) => {
        return res(ctx.json({ success: true, progress: mockProgressData }));
      }),
      rest.post('/api/progress/complete-lesson', (req, res, ctx) => {
        const updatedProgress = {
          ...mockProgressData,
          overallProgress: 70,
          completedLessons: 14,
          lastAccessedLesson: 'lesson-14'
        };
        return res(ctx.json({ success: true, progress: updatedProgress }));
      })
    );

    renderProgressTracker();

    // Wait for initial load
    await waitFor(() => {
      expect(screen.getByText('65% Complete')).toBeInTheDocument();
    });

    // Simulate lesson completion
    const completeButton = screen.getByRole('button', { name: /mark lesson complete/i });
    fireEvent.click(completeButton);

    // Check updated progress
    await waitFor(() => {
      expect(screen.getByText('70% Complete')).toBeInTheDocument();
      expect(screen.getByText('14 of 20 lessons completed')).toBeInTheDocument();
    });
  });

  // ✅ Good: Test achievement unlocking
  test('displays achievement notifications', async () => {
    const progressWithNewAchievement = {
      ...mockProgressData,
      achievements: ['first_lesson', 'halfway_point', 'week_streak']
    };

    server.use(
      rest.get('/api/progress/course-123/user-456', (req, res, ctx) => {
        return res(ctx.json({ success: true, progress: progressWithNewAchievement }));
      })
    );

    renderProgressTracker();

    await waitFor(() => {
      expect(screen.getByText('New Achievement Unlocked!')).toBeInTheDocument();
      expect(screen.getByText('Week Streak')).toBeInTheDocument();
    });
  });

  // ✅ Good: Test offline progress synchronization
  test('syncs offline progress when connection restored', async () => {
    const offlineProgress = {
      ...mockProgressData,
      offline: true,
      pendingUpdates: ['lesson-14-complete', 'lesson-15-complete']
    };

    server.use(
      rest.get('/api/progress/course-123/user-456', (req, res, ctx) => {
        return res(ctx.json({ success: true, progress: offlineProgress }));
      }),
      rest.post('/api/progress/sync-offline', (req, res, ctx) => {
        return res(ctx.json({ 
          success: true, 
          synced: 2,
          updatedProgress: {
            ...mockProgressData,
            overallProgress: 75,
            completedLessons: 15
          }
        }));
      })
    );

    renderProgressTracker();

    await waitFor(() => {
      expect(screen.getByText('Offline Progress Detected')).toBeInTheDocument();
    });

    const syncButton = screen.getByRole('button', { name: /sync progress/i });
    fireEvent.click(syncButton);

    await waitFor(() => {
      expect(screen.getByText('Progress synced successfully')).toBeInTheDocument();
      expect(screen.getByText('75% Complete')).toBeInTheDocument();
    });
  });
});
```

## End-to-End Testing for Educational Journeys

### 1. Complete Learning Journey E2E Testing

**✅ Good: Comprehensive E2E Testing with Playwright**
```javascript
// e2e/learning-journey.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Complete Learning Journey', () => {
  test.beforeEach(async ({ page }) => {
    // Setup test user and course data
    await page.goto('/');
    await page.fill('[data-testid="email-input"]', 'student@test.edu');
    await page.fill('[data-testid="password-input"]', 'testpassword');
    await page.click('[data-testid="login-button"]');
    
    // Wait for dashboard to load
    await expect(page.locator('[data-testid="dashboard"]')).toBeVisible();
  });

  // ✅ Good: Test complete course discovery to completion
  test('student can discover, enroll, and complete a course', async ({ page }) => {
    // Step 1: Course Discovery
    await page.click('[data-testid="browse-courses"]');
    await expect(page.locator('[data-testid="course-catalog"]')).toBeVisible();
    
    // Filter courses by difficulty
    await page.selectOption('[data-testid="difficulty-filter"]', 'BEGINNER');
    await page.waitForLoadState('networkidle');
    
    // Search for specific course
    await page.fill('[data-testid="course-search"]', 'JavaScript Fundamentals');
    await page.press('[data-testid="course-search"]', 'Enter');
    
    // Step 2: Course Details and enrollment
    await page.click('[data-testid="course-card"]:first-child');
    await expect(page.locator('[data-testid="course-title"]')).toContainText('JavaScript Fundamentals');
    
    // Check course details
    await expect(page.locator('[data-testid="course-description"]')).toBeVisible();
    await expect(page.locator('[data-testid="course-curriculum"]')).toBeVisible();
    
    // Enroll in course
    await page.click('[data-testid="enroll-button"]');
    await expect(page.locator('[data-testid="enrollment-success"]')).toBeVisible();
    
    // Step 3: Start Learning
    await page.click('[data-testid="start-learning-button"]');
    await expect(page.locator('[data-testid="lesson-content"]')).toBeVisible();
    
    // Step 4: Progress through lessons
    const lessonCount = await page.locator('[data-testid="lesson-item"]').count();
    
    for (let i = 0; i < Math.min(lessonCount, 3); i++) {
      // Complete current lesson
      await page.click('[data-testid="mark-complete-button"]');
      await expect(page.locator('[data-testid="lesson-completed"]')).toBeVisible();
      
      // Navigate to next lesson
      if (i < lessonCount - 1) {
        await page.click('[data-testid="next-lesson-button"]');
      }
    }
    
    // Step 5: Take Assessment
    await page.click('[data-testid="take-assessment-button"]');
    await expect(page.locator('[data-testid="quiz-question"]')).toBeVisible();
    
    // Answer quiz questions
    const questionCount = await page.locator('[data-testid="quiz-question"]').count();
    
    for (let q = 0; q < questionCount; q++) {
      await page.click(`[data-testid="quiz-answer-0"]`); // Select first answer
      await page.click('[data-testid="next-question-button"]');
    }
    
    // Submit quiz
    await page.click('[data-testid="submit-quiz-button"]');
    await expect(page.locator('[data-testid="quiz-results"]')).toBeVisible();
    
    // Step 6: Course Completion
    await expect(page.locator('[data-testid="course-completion"]')).toBeVisible();
    await expect(page.locator('[data-testid="certificate-link"]')).toBeVisible();
    
    // Verify progress tracking
    await page.goto('/dashboard');
    await expect(page.locator('[data-testid="completed-courses"]')).toContainText('1');
  });

  // ✅ Good: Test instructor workflow
  test('instructor can create and manage course content', async ({ page }) => {
    // Switch to instructor account
    await page.goto('/logout');
    await page.fill('[data-testid="email-input"]', 'instructor@test.edu');
    await page.fill('[data-testid="password-input"]', 'testpassword');
    await page.click('[data-testid="login-button"]');
    
    // Navigate to course creation
    await page.click('[data-testid="create-course-button"]');
    await expect(page.locator('[data-testid="course-builder"]')).toBeVisible();
    
    // Fill course details
    await page.fill('[data-testid="course-title"]', 'Advanced React Patterns');
    await page.fill('[data-testid="course-description"]', 'Learn advanced React patterns and best practices');
    await page.selectOption('[data-testid="course-difficulty"]', 'ADVANCED');
    
    // Add lessons
    await page.click('[data-testid="add-lesson-button"]');
    await page.fill('[data-testid="lesson-title"]', 'Higher-Order Components');
    await page.fill('[data-testid="lesson-content"]', 'Introduction to HOCs...');
    await page.click('[data-testid="save-lesson-button"]');
    
    // Add assessment
    await page.click('[data-testid="add-assessment-button"]');
    await page.fill('[data-testid="assessment-title"]', 'React Patterns Quiz');
    
    // Add quiz questions
    await page.click('[data-testid="add-question-button"]');
    await page.fill('[data-testid="question-text"]', 'What is a Higher-Order Component?');
    await page.fill('[data-testid="answer-1"]', 'A function that returns a component');
    await page.check('[data-testid="correct-answer-1"]');
    await page.click('[data-testid="save-question-button"]');
    
    // Publish course
    await page.click('[data-testid="publish-course-button"]');
    await expect(page.locator('[data-testid="course-published"]')).toBeVisible();
    
    // Verify course appears in catalog
    await page.goto('/courses');
    await expect(page.locator('[data-testid="course-title"]')).toContainText('Advanced React Patterns');
  });

  // ✅ Good: Test mobile learning experience
  test('mobile learning experience works correctly', async ({ page }) => {
    // Set mobile viewport
    await page.setViewportSize({ width: 375, height: 667 });
    
    // Navigate to course
    await page.goto('/courses/javascript-fundamentals');
    
    // Check mobile-optimized layout
    await expect(page.locator('[data-testid="mobile-course-header"]')).toBeVisible();
    await expect(page.locator('[data-testid="mobile-navigation"]')).toBeVisible();
    
    // Test mobile video player
    await page.click('[data-testid="play-video-button"]');
    await expect(page.locator('[data-testid="video-player"]')).toBeVisible();
    
    // Test mobile quiz interface
    await page.click('[data-testid="mobile-quiz-button"]');
    await expect(page.locator('[data-testid="mobile-quiz-container"]')).toBeVisible();
    
    // Test swipe navigation
    await page.locator('[data-testid="lesson-content"]').swipe('left');
    await expect(page.locator('[data-testid="next-lesson"]')).toBeVisible();
  });
});
```

## Performance Testing for Learning Analytics

### 1. Load Testing for Concurrent Users

**✅ Good: Performance Testing Setup**
```javascript
// performance/load-testing.js
import { check, sleep } from 'k6';
import http from 'k6/http';

export let options = {
  stages: [
    { duration: '2m', target: 100 }, // Ramp up to 100 users
    { duration: '5m', target: 100 }, // Stay at 100 users
    { duration: '2m', target: 200 }, // Ramp up to 200 users
    { duration: '5m', target: 200 }, // Stay at 200 users
    { duration: '2m', target: 0 },   // Ramp down to 0 users
  ],
  thresholds: {
    http_req_duration: ['p(95)<2000'], // 95% of requests under 2s
    http_req_failed: ['rate<0.1'],     // Error rate under 10%
  },
};

export default function () {
  // Test course loading performance
  let courseResponse = http.get('http://localhost:3000/api/courses');
  check(courseResponse, {
    'course list loads successfully': (r) => r.status === 200,
    'course list response time < 1s': (r) => r.timings.duration < 1000,
  });

  // Test course generation performance
  let generateResponse = http.post('http://localhost:3000/api/courses/generate', {
    title: 'Performance Test Course',
    difficulty: 'INTERMEDIATE',
    topics: ['testing', 'performance']
  });
  
  check(generateResponse, {
    'course generation completes': (r) => r.status === 200,
    'course generation under 30s': (r) => r.timings.duration < 30000,
  });

  // Test progress tracking under load
  let progressResponse = http.get('http://localhost:3000/api/progress/user/123');
  check(progressResponse, {
    'progress loads quickly': (r) => r.status === 200 && r.timings.duration < 500,
  });

  sleep(1);
}

// Test database performance under concurrent access
export function testDatabasePerformance() {
  const scenarios = {
    concurrent_enrollments: {
      executor: 'constant-arrival-rate',
      rate: 50, // 50 enrollments per second
      timeUnit: '1s',
      duration: '5m',
      preAllocatedVUs: 100,
    },
    concurrent_progress_updates: {
      executor: 'constant-arrival-rate',
      rate: 200, // 200 progress updates per second
      timeUnit: '1s',
      duration: '5m',
      preAllocatedVUs: 200,
    },
  };

  return { scenarios };
}
```

### 2. Memory and Resource Usage Testing

**✅ Good: Resource Usage Monitoring**
```javascript
// performance/memory-testing.js
class LMSPerformanceMonitor {
  constructor() {
    this.metrics = {
      memoryUsage: [],
      cpuUsage: [],
      responseTime: [],
      activeConnections: []
    };
  }

  async monitorLearningSession(duration = 300000) { // 5 minutes
    const startTime = Date.now();
    
    while (Date.now() - startTime < duration) {
      // Monitor memory usage
      const memoryUsage = process.memoryUsage();
      this.metrics.memoryUsage.push({
        timestamp: new Date(),
        heapUsed: memoryUsage.heapUsed,
        heapTotal: memoryUsage.heapTotal,
        rss: memoryUsage.rss
      });

      // Monitor API response times
      const startRequest = Date.now();
      await fetch('/api/courses/progress-update', {
        method: 'POST',
        body: JSON.stringify({ lessonId: 'lesson-123', progress: 0.5 })
      });
      const responseTime = Date.now() - startRequest;
      
      this.metrics.responseTime.push({
        timestamp: new Date(),
        duration: responseTime
      });

      // Check for memory leaks
      if (memoryUsage.heapUsed > 100 * 1024 * 1024) { // 100MB threshold
        console.warn('Memory usage exceeding threshold:', memoryUsage.heapUsed);
      }

      await this.sleep(1000); // Check every second
    }
  }

  generatePerformanceReport() {
    const avgMemory = this.calculateAverage(this.metrics.memoryUsage.map(m => m.heapUsed));
    const avgResponseTime = this.calculateAverage(this.metrics.responseTime.map(r => r.duration));
    
    return {
      memory: {
        average: avgMemory,
        peak: Math.max(...this.metrics.memoryUsage.map(m => m.heapUsed)),
        memoryLeakDetected: this.detectMemoryLeak()
      },
      performance: {
        averageResponseTime: avgResponseTime,
        slowestRequest: Math.max(...this.metrics.responseTime.map(r => r.duration)),
        performanceIssues: avgResponseTime > 2000
      }
    };
  }

  detectMemoryLeak() {
    if (this.metrics.memoryUsage.length < 10) return false;
    
    const recent = this.metrics.memoryUsage.slice(-10);
    const earlier = this.metrics.memoryUsage.slice(0, 10);
    
    const recentAvg = this.calculateAverage(recent.map(m => m.heapUsed));
    const earlierAvg = this.calculateAverage(earlier.map(m => m.heapUsed));
    
    return recentAvg > earlierAvg * 1.5; // 50% increase indicates potential leak
  }

  calculateAverage(array) {
    return array.reduce((sum, val) => sum + val, 0) / array.length;
  }

  sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Usage in tests
describe('LMS Performance Testing', () => {
  test('learning session does not cause memory leaks', async () => {
    const monitor = new LMSPerformanceMonitor();
    
    await monitor.monitorLearningSession(60000); // 1 minute test
    
    const report = monitor.generatePerformanceReport();
    
    expect(report.memory.memoryLeakDetected).toBe(false);
    expect(report.performance.averageResponseTime).toBeLessThan(2000);
    expect(report.memory.peak).toBeLessThan(200 * 1024 * 1024); // 200MB limit
  });
});
```

This comprehensive testing framework provides robust testing strategies specifically designed for LMS platforms, covering all critical educational workflows, performance requirements, and user experience scenarios. The framework emphasizes good practices with detailed examples and covers the unique testing challenges of educational technology systems. 