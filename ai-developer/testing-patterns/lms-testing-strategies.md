# LMS Testing Strategies and Best Practices

## Comprehensive Testing Architecture for Educational Platforms

### Testing Pyramid for LMS Applications
✅ **Good Example: Complete LMS Testing Strategy**
```
     /\
    /  \  E2E Tests (5-15%)
   /____\  - User journey testing
  /      \  - Critical workflow validation
 /        \ - Browser compatibility
/__________\ Integration Tests (15-25%)
/          \ - API endpoint testing
/   Unit    \ - Database integration
/   Tests   \ - AI service integration
/ (60-80%)  \ - Component integration
/____________\ Unit Tests
              - Pure functions
              - Component logic
              - Service methods
              - Utility functions
```

### Test Environment Configuration
✅ **Good Example: Multi-Environment Test Setup**
```javascript
// jest.config.js - Comprehensive Jest configuration for LMS
module.exports = {
  projects: [
    {
      displayName: 'unit',
      testMatch: ['<rootDir>/src/**/__tests__/**/*.test.js'],
      testEnvironment: 'node',
      setupFilesAfterEnv: ['<rootDir>/tests/setup/unit.setup.js'],
      collectCoverageFrom: [
        'src/**/*.{js,jsx}',
        '!src/**/*.stories.{js,jsx}',
        '!src/index.js',
        '!src/serviceWorker.js'
      ],
      coverageThreshold: {
        global: {
          branches: 80,
          functions: 80,
          lines: 80,
          statements: 80
        }
      }
    },
    {
      displayName: 'integration',
      testMatch: ['<rootDir>/tests/integration/**/*.test.js'],
      testEnvironment: 'node',
      setupFilesAfterEnv: ['<rootDir>/tests/setup/integration.setup.js'],
      globalSetup: '<rootDir>/tests/setup/globalSetup.js',
      globalTeardown: '<rootDir>/tests/setup/globalTeardown.js'
    },
    {
      displayName: 'e2e',
      testMatch: ['<rootDir>/tests/e2e/**/*.test.js'],
      testEnvironment: 'node',
      setupFilesAfterEnv: ['<rootDir>/tests/setup/e2e.setup.js'],
      testTimeout: 30000
    }
  ],
  collectCoverageFrom: [
    'src/**/*.{js,jsx}',
    '!**/node_modules/**',
    '!**/coverage/**'
  ],
  coverageReporters: ['text', 'lcov', 'html'],
  verbose: true
};

// tests/setup/integration.setup.js - Integration test environment
import { PrismaClient } from '@prisma/client';
import { execSync } from 'child_process';

const prisma = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_URL_TEST
    }
  }
});

beforeAll(async () => {
  // Reset test database
  execSync('npx prisma migrate reset --force --skip-seed', {
    env: { ...process.env, DATABASE_URL: process.env.DATABASE_URL_TEST }
  });
  
  // Run migrations
  execSync('npx prisma migrate deploy', {
    env: { ...process.env, DATABASE_URL: process.env.DATABASE_URL_TEST }
  });
  
  // Seed test data
  await seedTestData();
});

afterAll(async () => {
  await prisma.$disconnect();
});

beforeEach(async () => {
  // Clean up test data between tests
  await cleanupTestData();
});

const seedTestData = async () => {
  // Create test users
  await prisma.user.createMany({
    data: [
      {
        id: 'test-user-1',
        email: 'student@test.com',
        name: 'Test Student',
        role: 'student'
      },
      {
        id: 'test-instructor-1',
        email: 'instructor@test.com',
        name: 'Test Instructor',
        role: 'instructor'
      }
    ]
  });

  // Create test courses
  await prisma.course.create({
    data: {
      id: 'test-course-1',
      title: 'Test Course',
      description: 'A course for testing',
      difficulty: 'beginner',
      instructorId: 'test-instructor-1',
      lessons: {
        create: [
          {
            id: 'test-lesson-1',
            title: 'Test Lesson 1',
            description: 'First test lesson',
            order: 1,
            sections: {
              create: [
                {
                  id: 'test-section-1',
                  title: 'Test Section 1',
                  content: 'Test content',
                  type: 'text',
                  order: 1
                },
                {
                  id: 'test-section-2',
                  title: 'Test Section 2',
                  content: 'Test quiz content',
                  type: 'quiz',
                  order: 2
                }
              ]
            }
          }
        ]
      }
    }
  });
};

global.testData = {
  users: {
    student: 'test-user-1',
    instructor: 'test-instructor-1'
  },
  courses: {
    basic: 'test-course-1'
  },
  lessons: {
    first: 'test-lesson-1'
  },
  sections: {
    text: 'test-section-1',
    quiz: 'test-section-2'
  }
};

export { prisma };
```

## Unit Testing Patterns for LMS Components

### React Component Testing
✅ **Good Example: Comprehensive Component Testing**
```javascript
// src/components/__tests__/CourseCard.test.jsx
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { CourseCard } from '../CourseCard';
import { LMSProvider } from '../../contexts/LMSContext';

// Mock dependencies
jest.mock('../../hooks/useCourseProgress', () => ({
  useCourseProgress: jest.fn()
}));

const mockCourse = {
  id: 'course-1',
  title: 'Introduction to React',
  description: 'Learn React fundamentals',
  difficulty: 'beginner',
  imageUrl: 'https://example.com/image.jpg',
  lessonCount: 5,
  estimatedHours: 8,
  tags: ['javascript', 'frontend']
};

const mockUserProgress = {
  completedLessons: 2,
  totalLessons: 5,
  completionPercentage: 40
};

const defaultProps = {
  course: mockCourse,
  onEnroll: jest.fn(),
  onContinue: jest.fn()
};

const renderCourseCard = (props = {}) => {
  return render(
    <LMSProvider>
      <CourseCard {...defaultProps} {...props} />
    </LMSProvider>
  );
};

describe('CourseCard', () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('Course Information Display', () => {
    test('renders course title and description', () => {
      renderCourseCard();
      
      expect(screen.getByText('Introduction to React')).toBeInTheDocument();
      expect(screen.getByText('Learn React fundamentals')).toBeInTheDocument();
    });

    test('displays course metadata correctly', () => {
      renderCourseCard();
      
      expect(screen.getByText('5 lessons')).toBeInTheDocument();
      expect(screen.getByText('8h')).toBeInTheDocument();
      expect(screen.getByText('beginner')).toBeInTheDocument();
    });

    test('renders course image with proper alt text', () => {
      renderCourseCard();
      
      const image = screen.getByAltText('Introduction to React');
      expect(image).toBeInTheDocument();
      expect(image).toHaveAttribute('src', mockCourse.imageUrl);
    });
  });

  describe('Progress Display', () => {
    test('shows progress bar when user has progress', () => {
      renderCourseCard({ userProgress: mockUserProgress });
      
      expect(screen.getByRole('progressbar')).toBeInTheDocument();
      expect(screen.getByText('40%')).toBeInTheDocument();
    });

    test('hides progress bar when showProgress is false', () => {
      renderCourseCard({ 
        userProgress: mockUserProgress, 
        showProgress: false 
      });
      
      expect(screen.queryByRole('progressbar')).not.toBeInTheDocument();
    });

    test('displays correct progress information', () => {
      renderCourseCard({ userProgress: mockUserProgress });
      
      expect(screen.getByText('2/5 lessons completed')).toBeInTheDocument();
    });
  });

  describe('User Interactions', () => {
    test('calls onEnroll when course is not started', async () => {
      const user = userEvent.setup();
      const onEnrollMock = jest.fn();
      
      renderCourseCard({ onEnroll: onEnrollMock });
      
      const enrollButton = screen.getByText('Start Course');
      await user.click(enrollButton);
      
      expect(onEnrollMock).toHaveBeenCalledWith(mockCourse.id);
    });

    test('calls onContinue when course is in progress', async () => {
      const user = userEvent.setup();
      const onContinueMock = jest.fn();
      
      renderCourseCard({ 
        userProgress: mockUserProgress,
        onContinue: onContinueMock 
      });
      
      const continueButton = screen.getByText('Continue Learning');
      await user.click(continueButton);
      
      expect(onContinueMock).toHaveBeenCalledWith(mockCourse.id);
    });

    test('shows loading state during async operations', async () => {
      const user = userEvent.setup();
      const slowEnroll = jest.fn(() => 
        new Promise(resolve => setTimeout(resolve, 100))
      );
      
      renderCourseCard({ onEnroll: slowEnroll });
      
      const enrollButton = screen.getByText('Start Course');
      await user.click(enrollButton);
      
      expect(screen.getByText('Loading...')).toBeInTheDocument();
      
      await waitFor(() => {
        expect(screen.queryByText('Loading...')).not.toBeInTheDocument();
      });
    });
  });

  describe('Accessibility', () => {
    test('has proper ARIA labels and roles', () => {
      renderCourseCard();
      
      const card = screen.getByRole('article');
      expect(card).toHaveAttribute('data-testid', 'course-card');
      
      const button = screen.getByRole('button', { name: /start course/i });
      expect(button).toHaveAttribute('data-testid', 'course-action-button');
    });

    test('supports keyboard navigation', async () => {
      const user = userEvent.setup();
      const onEnrollMock = jest.fn();
      
      renderCourseCard({ onEnroll: onEnrollMock });
      
      const button = screen.getByRole('button', { name: /start course/i });
      button.focus();
      
      expect(button).toHaveFocus();
      
      await user.keyboard('{Enter}');
      expect(onEnrollMock).toHaveBeenCalled();
    });
  });

  describe('Error Handling', () => {
    test('handles enrollment failure gracefully', async () => {
      const user = userEvent.setup();
      const failingEnroll = jest.fn().mockRejectedValue(new Error('Network error'));
      
      renderCourseCard({ onEnroll: failingEnroll });
      
      const enrollButton = screen.getByText('Start Course');
      await user.click(enrollButton);
      
      await waitFor(() => {
        expect(screen.getByText('Start Course')).toBeInTheDocument();
      });
    });
  });
});
```

### Custom Hook Testing
✅ **Good Example: Custom Hook Testing Strategy**
```javascript
// src/hooks/__tests__/useCourseProgress.test.js
import { renderHook, act } from '@testing-library/react';
import { useCourseProgress } from '../useCourseProgress';
import { LMSProvider } from '../../contexts/LMSContext';

// Mock fetch globally
global.fetch = jest.fn();

const mockProgressData = {
  progress: {
    'course-1': {
      lessons: {
        'lesson-1': {
          sections: {
            'section-1': { completed: true, timeSpent: 300 },
            'section-2': { completed: false, timeSpent: 150 }
          }
        }
      }
    }
  },
  dispatch: jest.fn()
};

const mockCourseData = {
  courses: [
    {
      id: 'course-1',
      title: 'Test Course',
      lessons: [
        {
          id: 'lesson-1',
          title: 'Test Lesson',
          sections: [
            { id: 'section-1', title: 'Section 1' },
            { id: 'section-2', title: 'Section 2' }
          ]
        }
      ]
    }
  ]
};

const wrapper = ({ children }) => (
  <LMSProvider 
    initialUserProgress={mockProgressData}
    initialCourseData={mockCourseData}
  >
    {children}
  </LMSProvider>
);

describe('useCourseProgress', () => {
  beforeEach(() => {
    fetch.mockClear();
    mockProgressData.dispatch.mockClear();
  });

  describe('Progress Statistics', () => {
    test('calculates completion statistics correctly', () => {
      const { result } = renderHook(
        () => useCourseProgress('course-1'),
        { wrapper }
      );

      expect(result.current.statistics).toEqual({
        totalSections: 2,
        completedSections: 1,
        completionPercentage: 50,
        totalTimeSpent: 450
      });
    });

    test('handles empty progress data', () => {
      const emptyWrapper = ({ children }) => (
        <LMSProvider initialUserProgress={{ progress: {}, dispatch: jest.fn() }}>
          {children}
        </LMSProvider>
      );

      const { result } = renderHook(
        () => useCourseProgress('course-1'),
        { wrapper: emptyWrapper }
      );

      expect(result.current.statistics).toEqual({
        totalSections: 0,
        completedSections: 0,
        completionPercentage: 0,
        totalTimeSpent: 0
      });
    });
  });

  describe('Progress Updates', () => {
    test('updates section progress and calls API', async () => {
      fetch.mockResolvedValueOnce({ ok: true });

      const { result } = renderHook(
        () => useCourseProgress('course-1'),
        { wrapper }
      );

      await act(async () => {
        await result.current.updateSectionProgress('lesson-1', 'section-2', true, 200);
      });

      expect(mockProgressData.dispatch).toHaveBeenCalledWith({
        type: 'UPDATE_SECTION_PROGRESS',
        payload: {
          courseId: 'course-1',
          lessonId: 'lesson-1',
          sectionId: 'section-2',
          completed: true,
          timeSpent: 200
        }
      });

      expect(fetch).toHaveBeenCalledWith('/api/progress', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          courseId: 'course-1',
          lessonId: 'lesson-1',
          sectionId: 'section-2',
          completed: true,
          timeSpent: 200
        })
      });
    });

    test('handles API failure gracefully', async () => {
      const consoleSpy = jest.spyOn(console, 'error').mockImplementation();
      fetch.mockRejectedValueOnce(new Error('Network error'));

      const { result } = renderHook(
        () => useCourseProgress('course-1'),
        { wrapper }
      );

      await act(async () => {
        await result.current.updateSectionProgress('lesson-1', 'section-2', true, 200);
      });

      expect(consoleSpy).toHaveBeenCalledWith(
        'Failed to save progress:',
        expect.any(Error)
      );

      consoleSpy.mockRestore();
    });
  });

  describe('Next Section Navigation', () => {
    test('finds next incomplete section', () => {
      const { result } = renderHook(
        () => useCourseProgress('course-1'),
        { wrapper }
      );

      const nextSection = result.current.getNextSection();

      expect(nextSection).toEqual({
        courseId: 'course-1',
        lessonId: 'lesson-1',
        sectionId: 'section-2',
        section: { id: 'section-2', title: 'Section 2' },
        lesson: expect.objectContaining({ id: 'lesson-1' })
      });
    });

    test('returns null when course is complete', () => {
      const completeProgressWrapper = ({ children }) => (
        <LMSProvider 
          initialUserProgress={{
            progress: {
              'course-1': {
                lessons: {
                  'lesson-1': {
                    sections: {
                      'section-1': { completed: true },
                      'section-2': { completed: true }
                    }
                  }
                }
              }
            },
            dispatch: jest.fn()
          }}
          initialCourseData={mockCourseData}
        >
          {children}
        </LMSProvider>
      );

      const { result } = renderHook(
        () => useCourseProgress('course-1'),
        { wrapper: completeProgressWrapper }
      );

      expect(result.current.getNextSection()).toBeNull();
    });
  });
});
```

## Integration Testing for LMS APIs

### API Integration Testing
✅ **Good Example: Comprehensive API Testing**
```javascript
// tests/integration/courseAPI.test.js
import request from 'supertest';
import { app } from '../../src/app.js';
import { prisma } from '../../src/database/connection.js';

describe('Course API Integration', () => {
  let authToken;
  let testCourseId;

  beforeAll(async () => {
    // Get authentication token for tests
    const authResponse = await request(app)
      .post('/api/auth/login')
      .send({
        email: 'instructor@test.com',
        password: 'testpassword'
      });
    
    authToken = authResponse.body.token;
  });

  afterEach(async () => {
    // Clean up any test courses created
    if (testCourseId) {
      await prisma.course.delete({
        where: { id: testCourseId }
      });
      testCourseId = null;
    }
  });

  describe('GET /api/courses', () => {
    test('returns paginated course list', async () => {
      const response = await request(app)
        .get('/api/courses')
        .query({ page: 1, limit: 10 })
        .expect(200);

      expect(response.body).toHaveProperty('courses');
      expect(response.body).toHaveProperty('pagination');
      expect(response.body.pagination).toMatchObject({
        page: 1,
        limit: 10,
        totalCount: expect.any(Number),
        totalPages: expect.any(Number),
        hasNextPage: expect.any(Boolean),
        hasPreviousPage: false
      });

      expect(Array.isArray(response.body.courses)).toBe(true);
    });

    test('filters courses by difficulty', async () => {
      const response = await request(app)
        .get('/api/courses')
        .query({ difficulty: 'beginner' })
        .expect(200);

      response.body.courses.forEach(course => {
        expect(course.difficulty).toBe('beginner');
      });
    });

    test('searches courses by title and description', async () => {
      const response = await request(app)
        .get('/api/courses')
        .query({ search: 'test' })
        .expect(200);

      response.body.courses.forEach(course => {
        const matchesTitle = course.title.toLowerCase().includes('test');
        const matchesDescription = course.description.toLowerCase().includes('test');
        expect(matchesTitle || matchesDescription).toBe(true);
      });
    });

    test('validates pagination parameters', async () => {
      await request(app)
        .get('/api/courses')
        .query({ page: 0 })
        .expect(400);

      await request(app)
        .get('/api/courses')
        .query({ limit: 101 })
        .expect(400);
    });
  });

  describe('GET /api/courses/:courseId', () => {
    test('returns detailed course information', async () => {
      const response = await request(app)
        .get(`/api/courses/${global.testData.courses.basic}`)
        .expect(200);

      expect(response.body).toMatchObject({
        id: global.testData.courses.basic,
        title: expect.any(String),
        description: expect.any(String),
        difficulty: expect.any(String),
        lessons: expect.any(Array)
      });

      // Verify lesson structure
      response.body.lessons.forEach(lesson => {
        expect(lesson).toMatchObject({
          id: expect.any(String),
          title: expect.any(String),
          order: expect.any(Number),
          sections: expect.any(Array)
        });

        lesson.sections.forEach(section => {
          expect(section).toMatchObject({
            id: expect.any(String),
            title: expect.any(String),
            type: expect.any(String),
            order: expect.any(Number)
          });
        });
      });
    });

    test('returns 404 for non-existent course', async () => {
      const response = await request(app)
        .get('/api/courses/non-existent-id')
        .expect(404);

      expect(response.body).toMatchObject({
        error: 'Course not found',
        code: 'COURSE_NOT_FOUND'
      });
    });

    test('includes progress when requested and user is authenticated', async () => {
      const response = await request(app)
        .get(`/api/courses/${global.testData.courses.basic}`)
        .query({ includeProgress: true })
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(response.body).toHaveProperty('progress');
    });
  });

  describe('POST /api/courses/generate-stream', () => {
    test('generates course with AI and streams progress', (done) => {
      const courseData = {
        title: 'Test AI Generated Course',
        description: 'A course generated for testing purposes',
        difficulty: 'intermediate',
        tags: ['test', 'ai']
      };

      request(app)
        .post('/api/courses/generate-stream')
        .send(courseData)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200)
        .expect('Content-Type', /text\/event-stream/)
        .end((err, res) => {
          if (err) return done(err);

          // Parse SSE stream
          const events = res.text.split('\n\n').filter(Boolean);
          const progressEvents = events.filter(event => 
            event.includes('event: progress')
          );
          const completeEvents = events.filter(event => 
            event.includes('event: complete')
          );

          expect(progressEvents.length).toBeGreaterThan(0);
          expect(completeEvents.length).toBe(1);

          // Extract course ID from complete event for cleanup
          const completeEvent = completeEvents[0];
          const completeData = JSON.parse(
            completeEvent.split('data: ')[1]
          );
          testCourseId = completeData.courseId;

          done();
        });
    }, 30000); // Extended timeout for AI generation

    test('validates course creation input', async () => {
      const invalidCourseData = {
        title: 'AB', // Too short
        description: 'Short', // Too short
        difficulty: 'invalid', // Invalid difficulty
      };

      await request(app)
        .post('/api/courses/generate-stream')
        .send(invalidCourseData)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(400);
    });

    test('requires authentication for course generation', async () => {
      const courseData = {
        title: 'Test Course',
        description: 'A test course description',
        difficulty: 'beginner'
      };

      await request(app)
        .post('/api/courses/generate-stream')
        .send(courseData)
        .expect(401);
    });
  });

  describe('Course Management Operations', () => {
    beforeEach(async () => {
      // Create a test course for management operations
      const course = await prisma.course.create({
        data: {
          title: 'Test Management Course',
          description: 'Course for testing management operations',
          difficulty: 'beginner',
          instructorId: global.testData.users.instructor
        }
      });
      testCourseId = course.id;
    });

    test('updates course information', async () => {
      const updateData = {
        title: 'Updated Course Title',
        description: 'Updated description',
        difficulty: 'advanced',
        tags: ['updated', 'test']
      };

      const response = await request(app)
        .put(`/api/courses/${testCourseId}`)
        .send(updateData)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(response.body.course).toMatchObject(updateData);
    });

    test('soft deletes course', async () => {
      await request(app)
        .delete(`/api/courses/${testCourseId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      // Verify course is soft deleted
      const deletedCourse = await prisma.course.findUnique({
        where: { id: testCourseId }
      });

      expect(deletedCourse.status).toBe('deleted');
      expect(deletedCourse.deletedAt).toBeTruthy();
    });
  });
});
```

## End-to-End Testing for LMS User Journeys

### Playwright E2E Testing
✅ **Good Example: Complete User Journey Testing**
```javascript
// tests/e2e/learningJourney.test.js
import { test, expect } from '@playwright/test';

test.describe('Complete Learning Journey', () => {
  test.beforeEach(async ({ page }) => {
    // Navigate to the application
    await page.goto('/');
    
    // Skip authentication for Stage 1 testing
    await expect(page).toHaveTitle(/LMS Platform/);
  });

  test('User can discover, enroll, and complete a course', async ({ page }) => {
    // Step 1: Browse courses
    await test.step('Browse available courses', async () => {
      await page.waitForSelector('[data-testid="course-grid"]');
      
      const courseCards = page.locator('[data-testid="course-card"]');
      await expect(courseCards.first()).toBeVisible();
      
      // Verify course information is displayed
      await expect(courseCards.first().locator('h3')).toContainText(/./);
      await expect(courseCards.first().locator('.course-card__description')).toContainText(/./);
    });

    // Step 2: View course details
    let courseTitle;
    await test.step('View course details', async () => {
      const firstCourse = page.locator('[data-testid="course-card"]').first();
      courseTitle = await firstCourse.locator('h3').textContent();
      
      await firstCourse.click();
      await page.waitForURL(/\/courses\/[^/]+$/);
      
      // Verify course detail page elements
      await expect(page.locator('h1')).toContainText(courseTitle);
      await expect(page.locator('[data-testid="lesson-grid"]')).toBeVisible();
    });

    // Step 3: Start learning
    let lessonTitle;
    await test.step('Navigate to first lesson', async () => {
      const firstLesson = page.locator('[data-testid="lesson-card"]').first();
      lessonTitle = await firstLesson.locator('h3').textContent();
      
      await firstLesson.click();
      await page.waitForURL(/\/courses\/[^/]+\/lessons\/[^/]+$/);
      
      // Verify lesson view components
      await expect(page.locator('h1')).toContainText(lessonTitle);
      await expect(page.locator('[data-testid="section-content"]')).toBeVisible();
      await expect(page.locator('[data-testid="lesson-navigation"]')).toBeVisible();
    });

    // Step 4: Complete sections progressively
    await test.step('Complete lesson sections', async () => {
      // Check initial section status
      const firstSection = page.locator('[data-testid="section-item"]').first();
      await expect(firstSection).toHaveClass(/unlocked/);
      
      // Complete first section
      await page.locator('[data-testid="mark-complete-button"]').click();
      await expect(firstSection).toHaveClass(/completed/);
      
      // Verify next section is unlocked
      const secondSection = page.locator('[data-testid="section-item"]').nth(1);
      await expect(secondSection).toHaveClass(/unlocked/);
      
      // Navigate to next section
      await page.locator('[data-testid="next-section-button"]').click();
      
      // Complete second section
      await page.locator('[data-testid="mark-complete-button"]').click();
      await expect(secondSection).toHaveClass(/completed/);
    });

    // Step 5: Verify progress tracking
    await test.step('Verify progress persistence', async () => {
      // Check lesson completion status
      await expect(page.locator('[data-testid="lesson-progress"]')).toContainText('100%');
      
      // Navigate back to course overview
      await page.locator('[data-testid="back-to-course"]').click();
      await page.waitForURL(/\/courses\/[^/]+$/);
      
      // Verify progress is reflected in course view
      const completedLesson = page.locator('[data-testid="lesson-card"]').first();
      await expect(completedLesson).toHaveClass(/completed/);
      
      // Check course progress
      await expect(page.locator('[data-testid="course-progress"]')).toContainText(/\d+%/);
    });

    // Step 6: Test AI Teacher integration
    await test.step('Access AI Teacher', async () => {
      // Navigate back to lesson
      const firstLesson = page.locator('[data-testid="lesson-card"]').first();
      await firstLesson.click();
      
      // Open AI Teacher
      await page.locator('[data-testid="ai-teacher-button"]').click();
      await expect(page.locator('[data-testid="ai-teacher-interface"]')).toBeVisible();
      
      // Verify AI Teacher functionality
      await expect(page.locator('[data-testid="ai-chat-input"]')).toBeVisible();
      await expect(page.locator('[data-testid="ai-chat-history"]')).toBeVisible();
    });
  });

  test('User can generate new courses with AI', async ({ page }) => {
    await test.step('Navigate to course generation', async () => {
      await page.locator('[data-testid="generate-course-button"]').click();
      await expect(page.locator('[data-testid="course-generator"]')).toBeVisible();
    });

    await test.step('Fill course generation form', async () => {
      await page.fill('[data-testid="course-title-input"]', 'E2E Test Course');
      await page.fill('[data-testid="course-description-input"]', 'A course generated during E2E testing for validation purposes');
      await page.selectOption('[data-testid="difficulty-select"]', 'beginner');
    });

    await test.step('Generate course and monitor progress', async () => {
      await page.locator('[data-testid="generate-button"]').click();
      
      // Wait for generation to start
      await expect(page.locator('[data-testid="generation-progress"]')).toBeVisible();
      
      // Monitor progress updates
      await expect(page.locator('[data-testid="progress-message"]')).toContainText(/generating/i);
      
      // Wait for completion (with extended timeout for AI generation)
      await expect(page.locator('[data-testid="generation-complete"]')).toBeVisible({ 
        timeout: 60000 
      });
    });

    await test.step('Verify generated course', async () => {
      // Should redirect to new course
      await page.waitForURL(/\/courses\/[^/]+$/);
      
      // Verify course structure
      await expect(page.locator('h1')).toContainText('E2E Test Course');
      await expect(page.locator('[data-testid="lesson-grid"]')).toBeVisible();
      
      // Check that lessons were generated
      const lessons = page.locator('[data-testid="lesson-card"]');
      await expect(lessons).toHaveCount(6); // Should generate 6 lessons
      
      // Verify lesson structure
      for (let i = 0; i < 6; i++) {
        const lesson = lessons.nth(i);
        await expect(lesson.locator('h3')).toContainText(/./);
        await expect(lesson.locator('[data-testid="section-count"]')).toContainText('2 sections');
      }
    });
  });

  test('User experience remains consistent across devices', async ({ page, browserName }) => {
    // Test responsive behavior
    await test.step('Test mobile viewport', async () => {
      await page.setViewportSize({ width: 375, height: 812 });
      
      await page.waitForSelector('[data-testid="course-grid"]');
      await expect(page.locator('[data-testid="mobile-menu-button"]')).toBeVisible();
      
      // Test mobile navigation
      await page.locator('[data-testid="mobile-menu-button"]').click();
      await expect(page.locator('[data-testid="mobile-menu"]')).toBeVisible();
    });

    await test.step('Test tablet viewport', async () => {
      await page.setViewportSize({ width: 768, height: 1024 });
      
      // Verify layout adapts properly
      const courseGrid = page.locator('[data-testid="course-grid"]');
      await expect(courseGrid).toBeVisible();
      
      // Check that courses are displayed in grid format
      const courseCards = page.locator('[data-testid="course-card"]');
      await expect(courseCards.first()).toBeVisible();
    });

    await test.step('Test desktop viewport', async () => {
      await page.setViewportSize({ width: 1920, height: 1080 });
      
      // Verify full desktop layout
      await expect(page.locator('[data-testid="desktop-navigation"]')).toBeVisible();
      await expect(page.locator('[data-testid="course-grid"]')).toBeVisible();
    });
  });
});

// tests/e2e/accessibility.test.js
test.describe('Accessibility Compliance', () => {
  test('LMS interface meets WCAG guidelines', async ({ page }) => {
    await page.goto('/');
    
    // Test keyboard navigation
    await test.step('Keyboard navigation works correctly', async () => {
      await page.keyboard.press('Tab');
      await expect(page.locator(':focus')).toBeVisible();
      
      // Navigate through course cards
      for (let i = 0; i < 3; i++) {
        await page.keyboard.press('Tab');
        const focusedElement = page.locator(':focus');
        await expect(focusedElement).toBeVisible();
      }
    });

    // Test screen reader compatibility
    await test.step('Screen reader landmarks are present', async () => {
      await expect(page.locator('main')).toBeVisible();
      await expect(page.locator('nav')).toBeVisible();
      await expect(page.locator('[role="banner"]')).toBeVisible();
    });

    // Test contrast and readability
    await test.step('Text contrast meets standards', async () => {
      const courseTitle = page.locator('[data-testid="course-card"] h3').first();
      await expect(courseTitle).toBeVisible();
      
      // Verify text is readable (basic contrast check)
      const styles = await courseTitle.evaluate(el => {
        const computed = window.getComputedStyle(el);
        return {
          color: computed.color,
          backgroundColor: computed.backgroundColor,
          fontSize: computed.fontSize
        };
      });
      
      expect(styles.fontSize).toBeTruthy();
      expect(styles.color).toBeTruthy();
    });
  });
});
```

## Key LMS Testing Strategies Takeaways

### Critical Testing Patterns for Educational Platforms ✅

1. **Test Pyramid Structure**
   - Emphasize unit tests (60-80%) for business logic and utility functions
   - Integration tests (15-25%) for API endpoints and database operations
   - E2E tests (5-15%) for critical user journeys and workflows
   - Focus testing effort on core educational functionality

2. **Learning-Specific Testing**
   - Test progress tracking accuracy across all levels (section → lesson → course)
   - Validate AI-generated content quality and structure
   - Ensure course completion workflows function correctly
   - Test time tracking and analytics calculations

3. **User Journey Testing**
   - Test complete learning paths from course discovery to completion
   - Validate course generation and AI integration workflows
   - Ensure responsive design works across all device sizes
   - Test accessibility compliance for educational content

4. **Data Consistency Testing**
   - Validate database transactions for progress updates
   - Test concurrent user scenarios and data conflicts
   - Ensure proper rollback behavior on failures
   - Verify real-time updates and synchronization

5. **Performance and Reliability Testing**
   - Test AI service integration under various load conditions
   - Validate course generation performance with complex content
   - Ensure graceful degradation during external service failures
   - Test error handling and recovery mechanisms

### Testing Anti-Patterns to Avoid in LMS Development ❌

1. **Testing Implementation Instead of Behavior** - Testing internal component state rather than user-visible outcomes
2. **Insufficient AI Integration Testing** - Not testing edge cases and failure modes of AI services
3. **Missing Progress Tracking Validation** - Not thoroughly testing learning progress persistence and accuracy
4. **Inadequate Accessibility Testing** - Skipping keyboard navigation and screen reader compatibility tests
5. **Ignoring Real User Scenarios** - Testing isolated features without considering complete learning workflows

Comprehensive testing strategies are essential for ensuring LMS platforms provide reliable, accurate, and accessible learning experiences that meet the high standards expected in educational technology. 