# LMS Testing Frameworks & Educational Platform Quality Assurance (2025)

## 🔄 Fundamental Principle: Constant Research & Web Updates
**Critical Success Factor**: This guide must be continuously updated with latest 2025+ web research on testing patterns for educational platforms. All team members must regularly research and update this knowledge base.

## Educational Testing Architecture

### 1. Educational Testing Pyramid for LMS Platforms

#### ✅ Good Example: Comprehensive Educational Testing Strategy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ EDUCATIONAL TESTING FRAMEWORK ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
**Testing Philosophy**: Multi-layer testing approach specific to educational platform requirements

**Educational Testing Pyramid**:
```typescript
// Good: Educational testing architecture with learning context
interface EducationalTestingSuite {
  unitTests: EducationalUnitTests;
  integrationTests: LearningIntegrationTests;
  e2eTests: StudentJourneyTests;
  performanceTests: EducationalLoadTests;
  accessibilityTests: EducationalA11yTests;
  securityTests: EducationalDataProtectionTests;
}

// Educational unit testing for learning components
describe('Educational Course Progress Calculator', () => {
  describe('Learning Progress Tracking', () => {
    it('should calculate course completion percentage correctly for educational progression', () => {
      // Good: Educational context in test data
      const studentProgress = {
        completedLessons: 8,
        totalLessons: 10,
        completedAssignments: 5,
        totalAssignments: 6,
        completedQuizzes: 3,
        totalQuizzes: 4
      };
      
      const courseCompletion = calculateEducationalProgress(studentProgress);
      
      // Educational-specific assertions
      expect(courseCompletion.overallPercentage).toBe(83); // Weighted average
      expect(courseCompletion.lessonProgress).toBe(80);
      expect(courseCompletion.assignmentProgress).toBe(83);
      expect(courseCompletion.quizProgress).toBe(75);
      expect(courseCompletion.canReceiveCertificate).toBe(true); // >80% threshold
    });
    
    it('should handle learning pathway prerequisites correctly', () => {
      const student = createTestStudent();
      const course = createTestCourse({
        prerequisites: ['basic-math-101', 'intro-programming-102']
      });
      
      const enrollmentEligibility = checkEducationalPrerequisites(student, course);
      
      expect(enrollmentEligibility.isEligible).toBe(false);
      expect(enrollmentEligibility.missingPrerequisites).toContain('basic-math-101');
      expect(enrollmentEligibility.suggestedLearningPath).toBeDefined();
    });
    
    it('should track learning time accurately for educational analytics', () => {
      const learningSession = {
        lessonId: 'lesson-123',
        startTime: new Date('2025-01-01T10:00:00Z'),
        endTime: new Date('2025-01-01T10:45:00Z'),
        pausedDuration: 5 * 60 * 1000, // 5 minutes paused
        studentId: 'student-456'
      };
      
      const timeTracking = calculateEducationalTime(learningSession);
      
      expect(timeTracking.totalTime).toBe(45 * 60 * 1000); // 45 minutes
      expect(timeTracking.activeTime).toBe(40 * 60 * 1000); // 40 minutes active
      expect(timeTracking.engagementRate).toBe(0.89); // 89% engagement
    });
  });
  
  describe('Educational Content Validation', () => {
    it('should validate AI-generated educational content for appropriateness', () => {
      const aiGeneratedLesson = {
        title: 'Introduction to Quantum Computing',
        content: 'Quantum computing leverages quantum mechanics...',
        difficulty: 'intermediate',
        targetAge: '16-18',
        learningObjectives: [
          'Understand quantum superposition',
          'Explain quantum entanglement',
          'Compare classical vs quantum algorithms'
        ]
      };
      
      const validation = validateEducationalContent(aiGeneratedLesson);
      
      expect(validation.isAppropriate).toBe(true);
      expect(validation.readabilityScore).toBeGreaterThan(60); // Grade-appropriate
      expect(validation.learningObjectiveAlignment).toBe(true);
      expect(validation.contentSafety.inappropriate).toBe(false);
    });
    
    it('should ensure FERPA compliance in educational content', () => {
      const lessonWithData = {
        content: 'Student John Smith scored 95% on the quiz...',
        includedData: ['student_name', 'grade_information']
      };
      
      const complianceCheck = validateFERPACompliance(lessonWithData);
      
      expect(complianceCheck.hasPersonalData).toBe(true);
      expect(complianceCheck.requiresPermission).toBe(true);
      expect(complianceCheck.violationRisk).toBe('HIGH');
      expect(complianceCheck.sanitizedContent).not.toContain('John Smith');
    });
  });
});

// Educational integration testing for learning workflows
describe('Educational Learning Platform Integration', () => {
  let testEducationalEnvironment: EducationalTestEnvironment;
  
  beforeEach(async () => {
    testEducationalEnvironment = await setupEducationalTestEnvironment({
      courses: 3,
      students: 10,
      instructors: 2,
      testDatabase: true,
      mockAIServices: true
    });
  });
  
  describe('Complete Student Learning Journey', () => {
    it('should support full course enrollment to completion workflow', async () => {
      const student = testEducationalEnvironment.getStudent('student-1');
      const course = testEducationalEnvironment.getCourse('intro-programming');
      
      // Educational enrollment process
      const enrollment = await enrollStudentInCourse(student.id, course.id);
      expect(enrollment.status).toBe('ACTIVE');
      expect(enrollment.progress).toBe(0);
      
      // Simulate learning progression
      for (const lesson of course.lessons) {
        for (const section of lesson.sections) {
          const sectionCompletion = await completeEducationalSection(
            student.id,
            section.id,
            {
              timeSpent: section.estimatedMinutes * 60 * 1000,
              score: Math.random() * 40 + 60 // 60-100% score
            }
          );
          
          expect(sectionCompletion.completed).toBe(true);
          expect(sectionCompletion.timestamp).toBeDefined();
        }
        
        const lessonProgress = await getLessonProgress(student.id, lesson.id);
        expect(lessonProgress.completed).toBe(true);
      }
      
      // Verify course completion
      const finalProgress = await getCourseProgress(student.id, course.id);
      expect(finalProgress.percentage).toBe(100);
      expect(finalProgress.completed).toBe(true);
      expect(finalProgress.certificateEligible).toBe(true);
    });
    
    it('should handle concurrent student learning sessions correctly', async () => {
      const course = testEducationalEnvironment.getCourse('advanced-algorithms');
      const students = testEducationalEnvironment.getStudents().slice(0, 5);
      
      // Simulate 5 students taking the same lesson simultaneously
      const concurrentSessions = students.map(student =>
        startLearningSession(student.id, course.lessons[0].id)
      );
      
      const sessions = await Promise.all(concurrentSessions);
      
      // Verify no conflicts in progress tracking
      sessions.forEach(session => {
        expect(session.sessionId).toBeDefined();
        expect(session.startTime).toBeDefined();
        expect(session.conflicts).toBeUndefined();
      });
      
      // Complete sessions and verify individual progress
      const completions = await Promise.all(
        sessions.map(session =>
          completeEducationalSession(session.sessionId, {
            timeSpent: 30 * 60 * 1000, // 30 minutes
            score: 85
          })
        )
      );
      
      completions.forEach(completion => {
        expect(completion.recorded).toBe(true);
        expect(completion.progressUpdated).toBe(true);
      });
    });
  });
  
  describe('AI Educational Content Generation Integration', () => {
    it('should generate and validate complete educational courses', async () => {
      const coursePrompt = {
        title: 'Introduction to Machine Learning for Beginners',
        difficulty: 'beginner',
        duration: '8 weeks',
        learningObjectives: [
          'Understand basic ML concepts',
          'Implement simple algorithms',
          'Evaluate model performance'
        ]
      };
      
      const generatedCourse = await generateEducationalCourse(coursePrompt);
      
      // Validate course structure
      expect(generatedCourse.modules).toHaveLength(8); // 8 weeks = 8 modules
      expect(generatedCourse.totalLessons).toBeGreaterThanOrEqual(24); // ~3 lessons per week
      
      // Validate educational content quality
      for (const module of generatedCourse.modules) {
        expect(module.learningObjectives).toBeDefined();
        expect(module.assessments.length).toBeGreaterThan(0);
        
        for (const lesson of module.lessons) {
          const contentValidation = await validateEducationalContent(lesson.content);
          expect(contentValidation.appropriateForLevel).toBe(true);
          expect(contentValidation.learningValue).toBeGreaterThan(0.7);
        }
      }
      
      // Test course functionality
      const testStudent = testEducationalEnvironment.getStudent('student-1');
      const enrollment = await enrollStudentInCourse(testStudent.id, generatedCourse.id);
      expect(enrollment.success).toBe(true);
    });
  });
});
```
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ EDUCATIONAL TESTING FRAMEWORK ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#### ❌ Bad Example: Generic Testing Without Educational Context
```typescript
// Bad: Generic testing without educational platform considerations
describe('User API', () => {
  it('should return users', async () => {
    const response = await request(app).get('/users');
    expect(response.status).toBe(200);
    // No educational context, learning progression, or FERPA considerations
  });
});

// No consideration for educational workflows, progress tracking, or compliance
```

### 2. End-to-End Educational Testing

#### ✅ Good Example: Comprehensive Student Journey Testing
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ EDUCATIONAL E2E TESTING ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
**Testing Framework**: Playwright with educational platform test patterns

**Student Learning Journey Tests**:
```typescript
// Good: Complete educational user journey testing
import { test, expect, Page } from '@playwright/test';

class EducationalTestHelper {
  constructor(private page: Page) {}
  
  async navigateToLearningPlatform() {
    await this.page.goto('/dashboard');
    await expect(this.page.locator('[data-testid="educational-dashboard"]')).toBeVisible();
  }
  
  async enrollInCourse(courseName: string) {
    await this.page.click(`[data-testid="course-${courseName}"]`);
    await this.page.click('[data-testid="enroll-button"]');
    await expect(this.page.locator('[data-testid="enrollment-success"]')).toBeVisible();
  }
  
  async completeLearningSection(sectionTitle: string) {
    await this.page.click(`[data-testid="section-${sectionTitle}"]`);
    
    // Wait for educational content to load
    await expect(this.page.locator('[data-testid="learning-content"]')).toBeVisible();
    
    // Simulate learning time (educational platforms track time)
    await this.page.waitForTimeout(30000); // 30 seconds minimum learning time
    
    // Complete section
    await this.page.click('[data-testid="complete-section-button"]');
    await expect(this.page.locator('[data-testid="section-completed"]')).toBeVisible();
  }
  
  async verifyProgressTracking(expectedProgress: number) {
    const progressElement = this.page.locator('[data-testid="course-progress"]');
    await expect(progressElement).toContainText(`${expectedProgress}%`);
  }
  
  async accessAITeacher() {
    await this.page.click('[data-testid="ai-teacher-button"]');
    await expect(this.page.locator('[data-testid="ai-teacher-interface"]')).toBeVisible();
  }
}

test.describe('Educational Learning Platform - Complete Student Journey', () => {
  let educational: EducationalTestHelper;
  
  test.beforeEach(async ({ page }) => {
    educational = new EducationalTestHelper(page);
    await educational.navigateToLearningPlatform();
  });
  
  test('should complete full course learning workflow', async ({ page }) => {
    // Educational course discovery and enrollment
    await educational.enrollInCourse('introduction-to-programming');
    
    // Verify course structure loads correctly
    await expect(page.locator('[data-testid="course-modules"]')).toBeVisible();
    await expect(page.locator('[data-testid="learning-objectives"]')).toBeVisible();
    
    // Complete first module sections
    const module1Sections = [
      'what-is-programming',
      'programming-languages-overview',
      'setting-up-development-environment'
    ];
    
    for (const [index, section] of module1Sections.entries()) {
      await educational.completeLearningSection(section);
      
      // Verify progress updates after each section
      const expectedProgress = Math.round(((index + 1) / module1Sections.length) * 33.33);
      await educational.verifyProgressTracking(expectedProgress);
    }
    
    // Test AI teacher interaction
    await educational.accessAITeacher();
    
    // Ask educational question
    await page.fill('[data-testid="ai-teacher-input"]', 'Can you explain variables in programming?');
    await page.click('[data-testid="ask-ai-teacher"]');
    
    // Verify AI response is educational and contextual
    await expect(page.locator('[data-testid="ai-teacher-response"]')).toBeVisible();
    const aiResponse = await page.locator('[data-testid="ai-teacher-response"]').textContent();
    expect(aiResponse).toMatch(/variable/i);
    expect(aiResponse?.length).toBeGreaterThan(50); // Substantive educational response
  });
  
  test('should handle learning accessibility requirements', async ({ page }) => {
    // Test keyboard navigation for educational content
    await page.keyboard.press('Tab');
    await expect(page.locator(':focus')).toBeVisible();
    
    // Verify screen reader support for educational elements
    await expect(page.locator('[aria-label*="course progress"]')).toBeVisible();
    await expect(page.locator('[role="main"]')).toBeVisible();
    
    // Test high contrast mode for educational accessibility
    await page.emulateMedia({ colorScheme: 'dark' });
    await expect(page.locator('[data-testid="learning-content"]')).toBeVisible();
    
    // Verify text scaling for educational readability
    await page.addStyleTag({
      content: 'html { font-size: 150%; }'
    });
    await expect(page.locator('[data-testid="lesson-text"]')).toBeVisible();
  });
  
  test('should maintain learning session across browser refresh', async ({ page }) => {
    await educational.enrollInCourse('data-structures-fundamentals');
    await educational.completeLearningSection('introduction-to-arrays');
    
    // Record progress before refresh
    const progressBefore = await page.locator('[data-testid="course-progress"]').textContent();
    
    // Refresh browser (simulate accidental refresh during learning)
    await page.reload();
    
    // Verify educational session persistence
    await expect(page.locator('[data-testid="educational-dashboard"]')).toBeVisible();
    
    // Navigate back to course
    await page.click('[data-testid="my-courses"]');
    await page.click('[data-testid="course-data-structures-fundamentals"]');
    
    // Verify progress was maintained
    const progressAfter = await page.locator('[data-testid="course-progress"]').textContent();
    expect(progressAfter).toBe(progressBefore);
    
    // Verify student can continue from where they left off
    await expect(page.locator('[data-testid="continue-learning"]')).toBeVisible();
  });
  
  test('should support multiple concurrent learning sessions', async ({ browser }) => {
    // Simulate student accessing course from multiple devices
    const context1 = await browser.newContext();
    const context2 = await browser.newContext();
    
    const page1 = await context1.newPage();
    const page2 = await context2.newPage();
    
    const educational1 = new EducationalTestHelper(page1);
    const educational2 = new EducationalTestHelper(page2);
    
    // Start learning session on device 1
    await educational1.navigateToLearningPlatform();
    await educational1.enrollInCourse('web-development-basics');
    await educational1.completeLearningSection('html-fundamentals');
    
    // Access same course on device 2
    await educational2.navigateToLearningPlatform();
    await page2.click('[data-testid="my-courses"]');
    await page2.click('[data-testid="course-web-development-basics"]');
    
    // Verify progress sync between devices
    await educational2.verifyProgressTracking(25); // Assuming 25% for first section
    
    // Complete another section on device 2
    await educational2.completeLearningSection('css-styling-basics');
    
    // Verify progress updates on device 1
    await page1.reload();
    await educational1.verifyProgressTracking(50); // Now 50% completed
    
    await context1.close();
    await context2.close();
  });
});

// Educational performance testing
test.describe('Educational Platform Performance', () => {
  test('should load course content within educational performance standards', async ({ page }) => {
    const startTime = Date.now();
    
    await page.goto('/course/advanced-mathematics');
    
    // Educational content should load within 2 seconds for good learning experience
    await expect(page.locator('[data-testid="course-content"]')).toBeVisible({ timeout: 2000 });
    
    const loadTime = Date.now() - startTime;
    expect(loadTime).toBeLessThan(2000);
    
    // Verify educational media loads efficiently
    const videoElement = page.locator('[data-testid="educational-video"]');
    if (await videoElement.isVisible()) {
      await expect(videoElement).toHaveAttribute('preload', 'metadata');
    }
  });
  
  test('should handle large course content efficiently', async ({ page }) => {
    // Test with course containing extensive educational material
    await page.goto('/course/comprehensive-computer-science');
    
    // Verify lazy loading of educational sections
    const visibleSections = await page.locator('[data-testid^="section-"]').count();
    expect(visibleSections).toBeLessThanOrEqual(10); // Only load first 10 sections
    
    // Test infinite scroll for large course content
    await page.evaluate(() => window.scrollTo(0, document.body.scrollHeight));
    
    // Verify additional educational content loads
    await expect(page.locator('[data-testid="loading-more-content"]')).toBeVisible();
    await expect(page.locator('[data-testid="loading-more-content"]')).not.toBeVisible({ timeout: 5000 });
  });
});
```
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ EDUCATIONAL E2E TESTING ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#### ❌ Bad Example: Generic E2E Testing
```typescript
// Bad: No educational context or learning workflow consideration
test('should login and view page', async ({ page }) => {
  await page.goto('/');
  await page.click('button');
  await expect(page.locator('div')).toBeVisible();
  // No educational journey, progress tracking, or learning-specific testing
});
```

### 3. Educational Accessibility Testing

#### ✅ Good Example: Comprehensive Educational Accessibility Testing
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ EDUCATIONAL ACCESSIBILITY TESTING ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
**Accessibility Standards**: WCAG 2.2 AA compliance for educational platforms

**Educational A11y Testing Framework**:
```typescript
// Good: Educational accessibility testing with learning context
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test.describe('Educational Platform Accessibility', () => {
  test('should meet WCAG 2.2 AA standards for educational content', async ({ page }) => {
    await page.goto('/course/introduction-to-physics');
    
    // Run comprehensive accessibility audit for educational content
    const accessibilityScanResults = await new AxeBuilder({ page })
      .withTags(['wcag2a', 'wcag2aa', 'wcag21aa', 'wcag22aa'])
      .analyze();
    
    expect(accessibilityScanResults.violations).toEqual([]);
    
    // Educational-specific accessibility checks
    
    // Verify educational headings structure for screen readers
    const headings = await page.locator('h1, h2, h3, h4, h5, h6').all();
    for (const heading of headings) {
      const text = await heading.textContent();
      expect(text?.trim()).not.toBe(''); // No empty educational headings
    }
    
    // Verify educational images have appropriate alt text
    const educationalImages = await page.locator('[data-testid^="educational-image"]').all();
    for (const image of educationalImages) {
      const altText = await image.getAttribute('alt');
      expect(altText).toBeTruthy();
      expect(altText?.length).toBeGreaterThan(10); // Descriptive educational alt text
    }
    
    // Verify educational videos have captions
    const educationalVideos = await page.locator('[data-testid^="educational-video"]').all();
    for (const video of educationalVideos) {
      const trackElements = await video.locator('track[kind="captions"]').count();
      expect(trackElements).toBeGreaterThan(0);
    }
    
    // Test keyboard navigation for educational interface
    await page.keyboard.press('Tab');
    let focusedElement = await page.evaluate(() => document.activeElement?.tagName);
    expect(['BUTTON', 'A', 'INPUT', 'SELECT', 'TEXTAREA']).toContain(focusedElement);
    
    // Verify skip links for educational content
    await page.keyboard.press('Tab');
    const skipLink = page.locator('[data-testid="skip-to-main-content"]');
    if (await skipLink.isVisible()) {
      await expect(skipLink).toHaveText(/skip to main content/i);
    }
  });
  
  test('should support educational content for visually impaired students', async ({ page }) => {
    await page.goto('/lesson/mathematics-equations');
    
    // Test high contrast mode for educational readability
    await page.emulateMedia({ colorScheme: 'dark' });
    
    // Verify educational content visibility in high contrast
    await expect(page.locator('[data-testid="lesson-content"]')).toBeVisible();
    await expect(page.locator('[data-testid="mathematical-equations"]')).toBeVisible();
    
    // Test text scaling for educational accessibility
    await page.addStyleTag({
      content: `
        html { font-size: 200%; }
        * { font-size: inherit !important; }
      `
    });
    
    // Verify educational content remains functional with large text
    await expect(page.locator('[data-testid="lesson-text"]')).toBeVisible();
    await expect(page.locator('[data-testid="navigation-buttons"]')).toBeVisible();
    
    // Test screen reader announcements for educational progress
    const progressRegion = page.locator('[aria-live="polite"]');
    await page.click('[data-testid="complete-section"]');
    
    // Verify progress announcement for screen readers
    await expect(progressRegion).toContainText(/progress updated/i);
  });
  
  test('should provide educational content in multiple formats', async ({ page }) => {
    await page.goto('/lesson/scientific-concepts');
    
    // Verify multiple educational content formats for accessibility
    const contentFormats = {
      text: '[data-testid="text-content"]',
      audio: '[data-testid="audio-content"]',
      video: '[data-testid="video-content"]',
      interactive: '[data-testid="interactive-content"]'
    };
    
    let availableFormats = 0;
    for (const [format, selector] of Object.entries(contentFormats)) {
      if (await page.locator(selector).isVisible()) {
        availableFormats++;
        
        // Verify each format has accessibility features
        if (format === 'video') {
          await expect(page.locator(`${selector} track[kind="captions"]`)).toHaveCount(1);
        }
        if (format === 'audio') {
          await expect(page.locator(`${selector}[controls]`)).toBeVisible();
        }
        if (format === 'interactive') {
          await expect(page.locator(`${selector}[role="application"]`)).toBeVisible();
        }
      }
    }
    
    // Educational content should offer multiple accessible formats
    expect(availableFormats).toBeGreaterThanOrEqual(2);
  });
  
  test('should support educational keyboard shortcuts for efficiency', async ({ page }) => {
    await page.goto('/course/programming-fundamentals');
    
    // Test educational platform keyboard shortcuts
    const shortcuts = [
      { key: 'n', action: 'next lesson', testId: 'next-lesson-button' },
      { key: 'p', action: 'previous lesson', testId: 'previous-lesson-button' },
      { key: 'h', action: 'help menu', testId: 'help-menu' },
      { key: 't', action: 'toggle transcript', testId: 'transcript-toggle' }
    ];
    
    for (const shortcut of shortcuts) {
      await page.keyboard.press(shortcut.key);
      
      // Verify shortcut functionality for educational efficiency
      const targetElement = page.locator(`[data-testid="${shortcut.testId}"]`);
      if (await targetElement.isVisible()) {
        // Verify shortcut triggered the expected educational action
        await expect(targetElement).toHaveClass(/active|focused|highlighted/);
      }
    }
    
    // Test escape key for educational modal dialogs
    await page.click('[data-testid="ai-teacher-button"]');
    await expect(page.locator('[data-testid="ai-teacher-modal"]')).toBeVisible();
    
    await page.keyboard.press('Escape');
    await expect(page.locator('[data-testid="ai-teacher-modal"]')).not.toBeVisible();
  });
});

// Educational color contrast testing
test.describe('Educational Visual Accessibility', () => {
  test('should maintain proper color contrast for educational readability', async ({ page }) => {
    await page.goto('/course/literature-analysis');
    
    // Test color contrast for educational text elements
    const educationalTextElements = [
      '[data-testid="lesson-title"]',
      '[data-testid="lesson-content"]',
      '[data-testid="learning-objectives"]',
      '[data-testid="quiz-questions"]'
    ];
    
    for (const selector of educationalTextElements) {
      const element = page.locator(selector);
      if (await element.isVisible()) {
        // Get computed styles for contrast calculation
        const styles = await element.evaluate((el) => {
          const computed = getComputedStyle(el);
          return {
            color: computed.color,
            backgroundColor: computed.backgroundColor,
            fontSize: computed.fontSize
          };
        });
        
        // Verify educational content meets contrast requirements
        // Note: In actual implementation, use a color contrast library
        expect(styles.color).not.toBe(styles.backgroundColor);
      }
    }
  });
  
  test('should support educational platform dark mode accessibility', async ({ page }) => {
    await page.emulateMedia({ colorScheme: 'dark' });
    await page.goto('/dashboard');
    
    // Verify educational content adapts to dark mode
    await expect(page.locator('[data-testid="educational-dashboard"]')).toBeVisible();
    
    // Check that educational elements maintain visibility in dark mode
    const educationalElements = [
      '[data-testid="course-cards"]',
      '[data-testid="progress-indicators"]',
      '[data-testid="navigation-menu"]'
    ];
    
    for (const selector of educationalElements) {
      await expect(page.locator(selector)).toBeVisible();
    }
    
    // Verify dark mode doesn't break educational functionality
    await page.click('[data-testid="course-intro-to-science"]');
    await expect(page.locator('[data-testid="course-content"]')).toBeVisible();
  });
});
```
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ EDUCATIONAL ACCESSIBILITY TESTING ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#### ❌ Bad Example: Inadequate Accessibility Testing
```typescript
// Bad: Minimal accessibility testing without educational context
test('basic accessibility', async ({ page }) => {
  await page.goto('/');
  
  const accessibilityScanResults = await new AxeBuilder({ page }).analyze();
  expect(accessibilityScanResults.violations.length).toBe(0);
  
  // No educational accessibility considerations
  // No keyboard navigation testing
  // No screen reader testing
  // No educational content format testing
});
```

## Educational Testing Best Practices Summary

### ✅ DO's for Educational Platform Testing:
- ✅ Test complete student learning journeys from enrollment to completion
- ✅ Verify educational progress tracking accuracy across all levels
- ✅ Test AI-generated educational content for appropriateness and quality
- ✅ Ensure FERPA/GDPR compliance in all educational data handling
- ✅ Test accessibility features specifically for educational content
- ✅ Verify learning session persistence and synchronization
- ✅ Test concurrent student access and progress tracking
- ✅ Validate educational performance standards (loading times, responsiveness)
- ✅ Test educational keyboard shortcuts and navigation efficiency
- ✅ Verify multiple content formats for diverse learning needs

### ❌ DON'Ts for Educational Platform Testing:
- ❌ Use generic testing patterns without educational context
- ❌ Skip testing learning progression and prerequisite logic
- ❌ Ignore educational compliance requirements in testing
- ❌ Test without considering diverse student accessibility needs
- ❌ Skip testing AI educational content generation quality
- ❌ Ignore educational session management and persistence
- ❌ Test without educational performance standards
- ❌ Skip testing educational data privacy and security
- ❌ Use non-educational test data that doesn't reflect real usage
- ❌ Ignore educational platform-specific error scenarios

Remember: Keep this testing guide updated with latest educational platform testing methodologies and quality assurance standards. Research and validate these patterns against current 2025+ educational technology testing best practices. 