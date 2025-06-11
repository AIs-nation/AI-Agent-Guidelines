# Educational Platform Mobile Responsive Design & Accessibility Guide

## 🔄 Fundamental Principle: Constant Research & Web Updates
**Critical Success Factor**: This guide must be continuously updated with latest 2025+ web research on mobile responsive design patterns for educational platforms. All team members must regularly research and update this knowledge base.

## Core Mobile LMS Design Patterns

### 1. Educational Content Mobile-First Architecture
```css
/* ✅ Good Example: LMS Mobile-First CSS Architecture */
/* Base styles - Mobile First (320px+) */
.course-card {
  width: 100%;
  padding: 1rem;
  margin-bottom: 1rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  
  /* Touch-friendly target size for educational interactions */
  min-height: 120px;
  display: flex;
  flex-direction: column;
  
  /* Educational content accessibility */
  font-size: 16px; /* Minimum for readability */
  line-height: 1.5;
  color: #333;
  
  /* WCAG 2.2 compliance */
  focus-within: {
    outline: 3px solid #0066cc;
    outline-offset: 2px;
  }
}

.course-title {
  font-size: 1.25rem;
  font-weight: 600;
  margin-bottom: 0.5rem;
  
  /* Educational content hierarchy */
  line-height: 1.3;
  
  /* Screen reader support */
  &:focus {
    outline: 2px solid #0066cc;
  }
}

.course-progress {
  width: 100%;
  height: 8px;
  background: #e0e0e0;
  border-radius: 4px;
  overflow: hidden;
  margin: 0.5rem 0;
  
  /* Accessibility for progress indicators */
  &[role="progressbar"] {
    &[aria-label] {
      /* Ensures screen readers can announce progress */
    }
  }
}

.course-progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #4ade80, #22c55e);
  transition: width 0.3s ease;
  border-radius: 4px;
}

/* Tablet styles (768px+) */
@media (min-width: 768px) {
  .course-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 1.5rem;
    padding: 1.5rem;
  }
  
  .course-card {
    min-height: 200px;
    padding: 1.5rem;
  }
  
  .course-title {
    font-size: 1.5rem;
  }
}

/* Desktop styles (1024px+) */
@media (min-width: 1024px) {
  .course-grid {
    grid-template-columns: repeat(3, 1fr);
    gap: 2rem;
    max-width: 1200px;
    margin: 0 auto;
  }
  
  .course-card {
    min-height: 240px;
    transition: transform 0.2s ease, box-shadow 0.2s ease;
    
    &:hover {
      transform: translateY(-4px);
      box-shadow: 0 8px 16px rgba(0,0,0,0.15);
    }
  }
}

/* Large screens (1440px+) */
@media (min-width: 1440px) {
  .course-grid {
    grid-template-columns: repeat(4, 1fr);
    max-width: 1400px;
  }
}

/* ❌ Bad Example: Desktop-First Design */
/* .course-card {
  width: 300px; // Fixed width breaks on mobile
  float: left; // Old layout method
  font-size: 14px; // Too small for mobile
} */
```

### 2. Educational Lesson Viewer Mobile Optimization
```jsx
// ✅ Good Example: Mobile-Optimized Lesson Viewer
import React, { useState, useEffect } from 'react';
import { useMediaQuery } from '@/hooks/useMediaQuery';

const MobileLessonViewer = ({ lesson, onSectionComplete }) => {
  const isMobile = useMediaQuery('(max-width: 768px)');
  const [currentSection, setCurrentSection] = useState(0);
  const [isFullscreen, setIsFullscreen] = useState(false);

  return (
    <div className={`lesson-viewer ${isMobile ? 'mobile' : 'desktop'}`}>
      {/* Mobile-Optimized Header */}
      <header className="lesson-header">
        <button 
          className="back-button"
          aria-label="Go back to course"
          onClick={() => window.history.back()}
        >
          ← Back
        </button>
        <h1 className="lesson-title">
          {lesson.title}
        </h1>
        <button 
          className="fullscreen-toggle"
          aria-label={isFullscreen ? 'Exit fullscreen' : 'Enter fullscreen'}
          onClick={() => setIsFullscreen(!isFullscreen)}
        >
          {isFullscreen ? '⊡' : '⊞'}
        </button>
      </header>

      {/* Mobile Section Navigation */}
      {isMobile && (
        <nav className="mobile-section-nav" role="navigation" aria-label="Lesson sections">
          <div className="section-progress">
            <span className="current-section">
              Section {currentSection + 1} of {lesson.sections.length}
            </span>
            <div 
              className="section-progress-bar" 
              role="progressbar"
              aria-valuenow={((currentSection + 1) / lesson.sections.length) * 100}
              aria-valuemin="0"
              aria-valuemax="100"
              aria-label={`Lesson progress: ${currentSection + 1} of ${lesson.sections.length} sections`}
            >
              <div 
                className="progress-fill"
                style={{ width: `${((currentSection + 1) / lesson.sections.length) * 100}%` }}
              />
            </div>
          </div>
        </nav>
      )}

      {/* Main Content Area */}
      <main className="lesson-content">
        <section 
          className="current-section"
          tabIndex="0"
          role="main"
          aria-label={`Section ${currentSection + 1}: ${lesson.sections[currentSection]?.title}`}
        >
          <h2 className="section-title">
            {lesson.sections[currentSection]?.title}
          </h2>
          
          <div 
            className="section-content"
            dangerouslySetInnerHTML={{ __html: lesson.sections[currentSection]?.content }}
          />
          
          {/* Mobile-Optimized Action Buttons */}
          <div className="section-actions">
            <button
              className="previous-section"
              disabled={currentSection === 0}
              onClick={() => setCurrentSection(currentSection - 1)}
              aria-label="Go to previous section"
            >
              ← Previous
            </button>
            
            <button
              className="mark-complete"
              onClick={() => onSectionComplete(lesson.sections[currentSection].id)}
              aria-label="Mark this section as complete"
            >
              ✓ Complete
            </button>
            
            <button
              className="next-section"
              disabled={currentSection === lesson.sections.length - 1}
              onClick={() => setCurrentSection(currentSection + 1)}
              aria-label="Go to next section"
            >
              Next →
            </button>
          </div>
        </section>
      </main>

      {/* AI Teacher Integration - Mobile Optimized */}
      <aside className="ai-teacher-panel">
        <button 
          className="ai-teacher-toggle"
          aria-label="Open AI Teacher chat"
          onClick={() => setAiTeacherOpen(!aiTeacherOpen)}
        >
          🤖 Ask AI Teacher
        </button>
        
        {aiTeacherOpen && (
          <div className="ai-chat-mobile" role="dialog" aria-label="AI Teacher Chat">
            <div className="chat-messages" role="log" aria-live="polite">
              {/* AI chat messages */}
            </div>
            <form className="chat-input-form">
              <label htmlFor="ai-question" className="sr-only">
                Ask the AI teacher a question
              </label>
              <input
                id="ai-question"
                type="text"
                placeholder="Ask about this section..."
                className="chat-input"
                aria-describedby="chat-help"
              />
              <span id="chat-help" className="sr-only">
                Type your question and press enter to get help from the AI teacher
              </span>
              <button type="submit" aria-label="Send question">
                Send
              </button>
            </form>
          </div>
        )}
      </aside>
    </div>
  );
};

// ❌ Bad Example: Desktop-Only Lesson Viewer
// const LessonViewer = () => (
//   <div style={{ width: '1200px', margin: '0 auto' }}> // Fixed width
//     <div className="sidebar" style={{ width: '300px', float: 'left' }}> // No mobile consideration
//       // Desktop-only navigation
//     </div>
//   </div>
// );
```

### 3. Touch-Optimized Educational Interactions
```scss
// ✅ Good Example: Touch-Friendly Educational UI
.educational-interactive {
  // WCAG 2.2 AA minimum touch target size
  min-height: 44px;
  min-width: 44px;
  
  // Touch-friendly spacing
  margin: 8px;
  padding: 12px 16px;
  
  // Enhanced touch feedback for educational actions
  position: relative;
  overflow: hidden;
  
  &::before {
    content: '';
    position: absolute;
    top: 50%;
    left: 50%;
    width: 0;
    height: 0;
    border-radius: 50%;
    background: rgba(255, 255, 255, 0.5);
    transition: width 0.3s, height 0.3s, top 0.3s, left 0.3s;
    transform: translate(-50%, -50%);
  }
  
  &:active::before {
    width: 200px;
    height: 200px;
  }
}

// Progress Tracking - Mobile Optimized
.mobile-progress-tracker {
  position: sticky;
  top: 0;
  background: white;
  border-bottom: 1px solid #e0e0e0;
  padding: 1rem;
  z-index: 100;
  
  .progress-ring {
    width: 60px;
    height: 60px;
    border-radius: 50%;
    background: conic-gradient(
      #4ade80 var(--progress-angle, 0deg),
      #e5e7eb var(--progress-angle, 0deg)
    );
    display: flex;
    align-items: center;
    justify-content: center;
    position: relative;
    
    &::before {
      content: '';
      width: 46px;
      height: 46px;
      border-radius: 50%;
      background: white;
    }
    
    .progress-text {
      position: absolute;
      font-size: 14px;
      font-weight: 600;
      color: #333;
    }
  }
}

// Quiz/Assessment Mobile Layout
.mobile-quiz {
  .question-container {
    padding: 1.5rem;
    margin-bottom: 1rem;
    
    .question-text {
      font-size: 1.125rem;
      line-height: 1.6;
      margin-bottom: 1.5rem;
      color: #1f2937;
    }
    
    .answer-options {
      display: flex;
      flex-direction: column;
      gap: 1rem;
      
      .answer-option {
        padding: 1rem;
        border: 2px solid #e5e7eb;
        border-radius: 8px;
        background: white;
        cursor: pointer;
        transition: all 0.2s ease;
        
        // Touch feedback
        &:active {
          transform: scale(0.98);
          background: #f9fafb;
        }
        
        &.selected {
          border-color: #3b82f6;
          background: #eff6ff;
        }
        
        &.correct {
          border-color: #10b981;
          background: #ecfdf5;
        }
        
        &.incorrect {
          border-color: #ef4444;
          background: #fef2f2;
        }
      }
    }
  }
  
  .quiz-navigation {
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;
    background: white;
    border-top: 1px solid #e5e7eb;
    padding: 1rem;
    display: flex;
    justify-content: space-between;
    
    .nav-button {
      @extend .educational-interactive;
      background: #3b82f6;
      color: white;
      border: none;
      border-radius: 6px;
      font-weight: 500;
      
      &:disabled {
        background: #9ca3af;
        cursor: not-allowed;
      }
    }
  }
}

// ❌ Bad Example: Non-Touch Optimized
// .button {
//   width: 20px; // Too small for touch
//   height: 20px; // Too small for touch
//   margin: 2px; // Insufficient spacing
// }
```

### 4. Mobile Performance Optimization for Educational Content
```typescript
// ✅ Good Example: Mobile Performance Optimization
import React, { Suspense, lazy, useMemo } from 'react';
import { useIntersectionObserver } from '@/hooks/useIntersectionObserver';
import { useImageOptimization } from '@/hooks/useImageOptimization';

// Lazy load educational components for better mobile performance
const LessonVideo = lazy(() => import('@/components/LessonVideo'));
const InteractiveQuiz = lazy(() => import('@/components/InteractiveQuiz'));
const AITeacherChat = lazy(() => import('@/components/AITeacherChat'));

const MobileOptimizedCourse = ({ courseId }) => {
  // Intersection Observer for lazy loading course sections
  const { ref: courseRef, isIntersecting } = useIntersectionObserver({
    threshold: 0.1,
    rootMargin: '50px'
  });

  // Optimized image loading for course thumbnails
  const optimizedThumbnail = useImageOptimization({
    src: course.thumbnail,
    sizes: {
      mobile: '320w',
      tablet: '768w',
      desktop: '1024w'
    },
    formats: ['webp', 'avif', 'jpg']
  });

  // Memoize heavy computations for mobile
  const courseProgress = useMemo(() => {
    return calculateCourseProgress(course.lessons, userProgress);
  }, [course.lessons, userProgress]);

  // Virtual scrolling for long lesson lists on mobile
  const VirtualizedLessonList = useMemo(() => {
    return lessons.map((lesson, index) => (
      <VirtualizedLessonItem
        key={lesson.id}
        lesson={lesson}
        index={index}
        height={80} // Fixed height for performance
        onVisible={() => preloadLessonContent(lesson.id)}
      />
    ));
  }, [lessons]);

  return (
    <div className="mobile-optimized-course" ref={courseRef}>
      {/* Progressive image loading */}
      <div className="course-header">
        <picture>
          <source 
            srcSet={optimizedThumbnail.webp} 
            type="image/webp"
            media="(max-width: 768px)"
          />
          <source 
            srcSet={optimizedThumbnail.avif} 
            type="image/avif"
            media="(max-width: 768px)"
          />
          <img
            src={optimizedThumbnail.fallback}
            alt={course.title}
            loading="lazy"
            decoding="async"
            className="course-thumbnail"
          />
        </picture>
        
        <div className="course-meta">
          <h1 className="course-title">{course.title}</h1>
          <div className="progress-summary">
            <span>{courseProgress.completedLessons} of {course.totalLessons} lessons</span>
            <div className="progress-bar">
              <div 
                className="progress-fill"
                style={{ width: `${courseProgress.percentage}%` }}
                aria-label={`Course ${courseProgress.percentage}% complete`}
              />
            </div>
          </div>
        </div>
      </div>

      {/* Lazy-loaded content sections */}
      {isIntersecting && (
        <Suspense fallback={<SkeletonLoader type="lessons" />}>
          <div className="lesson-list">
            {VirtualizedLessonList}
          </div>
        </Suspense>
      )}

      {/* Performance-optimized floating action button */}
      <div className="floating-actions">
        <Suspense fallback={<div className="fab-skeleton" />}>
          <FloatingActionButton
            icon="🤖"
            label="AI Teacher"
            onClick={() => loadAITeacher()}
            position="bottom-right"
          />
        </Suspense>
      </div>
    </div>
  );
};

// Mobile-specific performance hooks
const useImageOptimization = ({ src, sizes, formats }) => {
  return useMemo(() => {
    const optimized = {};
    
    formats.forEach(format => {
      optimized[format] = generateResponsiveImageSources(src, sizes, format);
    });
    
    optimized.fallback = src;
    return optimized;
  }, [src, sizes, formats]);
};

const VirtualizedLessonItem = React.memo(({ lesson, height, onVisible }) => {
  const [isVisible, setIsVisible] = useState(false);
  const itemRef = useRef();

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting && !isVisible) {
          setIsVisible(true);
          onVisible();
        }
      },
      { threshold: 0.1 }
    );

    if (itemRef.current) {
      observer.observe(itemRef.current);
    }

    return () => observer.disconnect();
  }, [isVisible, onVisible]);

  return (
    <div 
      ref={itemRef}
      className="lesson-item"
      style={{ height: `${height}px` }}
    >
      {isVisible ? (
        <LessonContent lesson={lesson} />
      ) : (
        <SkeletonLoader type="lesson" />
      )}
    </div>
  );
});

// ❌ Bad Example: Unoptimized Mobile Loading
// const Course = ({ courseId }) => {
//   const [course, setCourse] = useState(null);
//   
//   useEffect(() => {
//     // Load all course data at once - bad for mobile
//     loadEntireCourse(courseId).then(setCourse);
//   }, [courseId]);
//   
//   return (
//     <div>
//       {course?.lessons.map(lesson => (
//         <div key={lesson.id}>
//           <img src={lesson.thumbnail} /> // No optimization
//           <video src={lesson.video} autoPlay /> // Bad for mobile data
//         </div>
//       ))}
//     </div>
//   );
// };
```

### 5. Accessibility-First Mobile Design for Educational Platforms
```jsx
// ✅ Good Example: Accessible Mobile Educational Interface
import React, { useReducer, useEffect } from 'react';
import { useA11yAnnouncements } from '@/hooks/useA11yAnnouncements';
import { useFocusManagement } from '@/hooks/useFocusManagement';

const AccessibleMobileLMS = () => {
  const { announce } = useA11yAnnouncements();
  const { manageFocus, trapFocus } = useFocusManagement();

  // Accessibility state management
  const [a11yState, dispatchA11y] = useReducer(accessibilityReducer, {
    isHighContrast: false,
    fontSize: 'normal',
    reducedMotion: false,
    screenReaderMode: false
  });

  return (
    <div 
      className={`lms-mobile ${a11yState.isHighContrast ? 'high-contrast' : ''}`}
      data-font-size={a11yState.fontSize}
      data-reduced-motion={a11yState.reducedMotion}
    >
      {/* Skip Navigation Links */}
      <nav className="skip-links" aria-label="Skip navigation">
        <a href="#main-content" className="skip-link">
          Skip to main content
        </a>
        <a href="#navigation" className="skip-link">
          Skip to navigation
        </a>
        <a href="#search" className="skip-link">
          Skip to search
        </a>
      </nav>

      {/* Accessibility Controls */}
      <aside className="accessibility-toolbar" role="toolbar" aria-label="Accessibility options">
        <button
          onClick={() => toggleHighContrast()}
          aria-pressed={a11yState.isHighContrast}
          aria-label="Toggle high contrast mode"
          className="a11y-control"
        >
          <span className="icon" aria-hidden="true">◐</span>
          High Contrast
        </button>
        
        <fieldset className="font-size-controls">
          <legend>Text Size</legend>
          <label className="sr-only" htmlFor="font-size-range">
            Adjust text size
          </label>
          <input
            id="font-size-range"
            type="range"
            min="12"
            max="24"
            step="2"
            value={fontSizeValue}
            onChange={(e) => setFontSize(e.target.value)}
            aria-describedby="font-size-description"
          />
          <span id="font-size-description" className="sr-only">
            Current text size: {fontSizeValue}px
          </span>
        </fieldset>
      </aside>

      {/* Main Navigation with ARIA */}
      <nav id="navigation" role="navigation" aria-label="Main navigation">
        <button
          className="nav-toggle"
          aria-expanded={isNavOpen}
          aria-controls="nav-menu"
          aria-label="Open main menu"
          onClick={() => setIsNavOpen(!isNavOpen)}
        >
          <span className="hamburger-icon" aria-hidden="true">
            <span></span>
            <span></span>
            <span></span>
          </span>
        </button>
        
        <ul 
          id="nav-menu"
          className={`nav-menu ${isNavOpen ? 'open' : ''}`}
          role="menu"
        >
          <li role="none">
            <a 
              href="/courses"
              role="menuitem"
              aria-current={currentPage === 'courses' ? 'page' : undefined}
            >
              Courses
            </a>
          </li>
          <li role="none">
            <a href="/progress" role="menuitem">
              My Progress
            </a>
          </li>
          <li role="none">
            <a href="/profile" role="menuitem">
              Profile
            </a>
          </li>
        </ul>
      </nav>

      {/* Course Search with Accessibility */}
      <section id="search" className="search-section">
        <form role="search" onSubmit={handleSearch}>
          <label htmlFor="course-search" className="search-label">
            Search for courses
          </label>
          <div className="search-input-group">
            <input
              id="course-search"
              type="search"
              placeholder="Enter course topic..."
              value={searchQuery}
              onChange={(e) => setSearchQuery(e.target.value)}
              aria-describedby="search-help search-results-count"
              autoComplete="off"
              role="combobox"
              aria-expanded={suggestions.length > 0}
              aria-owns="search-suggestions"
            />
            <button type="submit" aria-label="Search courses">
              <span className="icon" aria-hidden="true">🔍</span>
            </button>
          </div>
          
          <div id="search-help" className="help-text">
            Type to search through available courses
          </div>
          
          {suggestions.length > 0 && (
            <ul 
              id="search-suggestions"
              role="listbox"
              aria-label="Search suggestions"
            >
              {suggestions.map((suggestion, index) => (
                <li
                  key={suggestion.id}
                  role="option"
                  aria-selected={selectedSuggestion === index}
                  onClick={() => selectSuggestion(suggestion)}
                >
                  {suggestion.title}
                </li>
              ))}
            </ul>
          )}
        </form>
        
        <div id="search-results-count" aria-live="polite" aria-atomic="true">
          {searchResults.length > 0 && (
            `Found ${searchResults.length} course${searchResults.length !== 1 ? 's' : ''}`
          )}
        </div>
      </section>

      {/* Main Content Area */}
      <main id="main-content" role="main">
        <h1 className="page-title">
          {pageTitle}
        </h1>
        
        {/* Course Grid with Accessibility */}
        <section className="course-grid" role="region" aria-label="Available courses">
          <h2 className="section-title">Featured Courses</h2>
          
          <div className="grid" role="grid" aria-label="Course listings">
            {courses.map((course, index) => (
              <article
                key={course.id}
                className="course-card"
                role="gridcell"
                aria-rowindex={Math.floor(index / 2) + 1}
                aria-colindex={(index % 2) + 1}
              >
                <div className="course-content">
                  <img
                    src={course.thumbnail}
                    alt=""
                    role="presentation"
                    loading="lazy"
                  />
                  
                  <div className="course-info">
                    <h3 className="course-title">
                      <a 
                        href={`/course/${course.id}`}
                        aria-describedby={`course-${course.id}-meta`}
                      >
                        {course.title}
                      </a>
                    </h3>
                    
                    <div id={`course-${course.id}-meta`} className="course-meta">
                      <span className="difficulty" aria-label={`Difficulty: ${course.difficulty}`}>
                        {course.difficulty}
                      </span>
                      <span className="duration" aria-label={`Duration: ${course.duration} hours`}>
                        {course.duration}h
                      </span>
                    </div>
                    
                    <div className="progress-section">
                      <div 
                        className="progress-bar"
                        role="progressbar"
                        aria-valuenow={course.progress}
                        aria-valuemin="0"
                        aria-valuemax="100"
                        aria-label={`Course progress: ${course.progress}% complete`}
                      >
                        <div 
                          className="progress-fill"
                          style={{ width: `${course.progress}%` }}
                        />
                      </div>
                      <span className="progress-text">
                        {course.progress}% complete
                      </span>
                    </div>
                  </div>
                </div>
              </article>
            ))}
          </div>
        </section>
      </main>

      {/* Status announcements for screen readers */}
      <div
        id="status-announcements"
        role="status"
        aria-live="polite"
        aria-atomic="true"
        className="sr-only"
      >
        {statusMessage}
      </div>
    </div>
  );
};

// Accessibility utility hooks
const useA11yAnnouncements = () => {
  const announce = (message, priority = 'polite') => {
    const announcement = document.createElement('div');
    announcement.setAttribute('aria-live', priority);
    announcement.setAttribute('aria-atomic', 'true');
    announcement.setAttribute('class', 'sr-only');
    announcement.textContent = message;
    
    document.body.appendChild(announcement);
    
    setTimeout(() => {
      document.body.removeChild(announcement);
    }, 1000);
  };

  return { announce };
};

// ❌ Bad Example: Inaccessible Mobile Interface
// const BadMobileInterface = () => (
//   <div>
//     <div onClick={handleClick}>Click me</div> // No keyboard access
//     <img src="course.jpg" /> // No alt text
//     <div className="progress-bar">
//       <div style={{ width: '60%' }} /> // No aria attributes
//     </div>
//   </div>
// );
```

## Advanced Mobile Educational Patterns

### 6. Progressive Web App for Educational Platforms
```typescript
// ✅ Good Example: Educational PWA Implementation
// public/sw.js - Service Worker for Educational Content
const CACHE_NAME = 'lms-v1.2.0';
const EDUCATIONAL_CACHE = 'educational-content-v1';
const API_CACHE = 'api-responses-v1';

// Educational content caching strategy
const CACHE_STRATEGIES = {
  // Cache-first for static educational resources
  EDUCATIONAL_ASSETS: [
    '/courses',
    '/lessons',
    '/static/images/',
    '/static/videos/'
  ],
  
  // Network-first for dynamic learning progress
  DYNAMIC_CONTENT: [
    '/api/progress/',
    '/api/user/',
    '/api/ai-teacher/'
  ],
  
  // Stale-while-revalidate for course listings
  STALE_WHILE_REVALIDATE: [
    '/api/courses',
    '/api/search'
  ]
};

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.addAll([
        '/',
        '/offline',
        '/static/css/main.css',
        '/static/js/main.js',
        '/manifest.json',
        // Essential educational icons
        '/static/icons/course-icon.svg',
        '/static/icons/progress-icon.svg',
        '/static/icons/ai-teacher-icon.svg'
      ]);
    })
  );
});

self.addEventListener('fetch', (event) => {
  const { request } = event;
  const url = new URL(request.url);

  // Educational content caching strategy
  if (CACHE_STRATEGIES.EDUCATIONAL_ASSETS.some(path => url.pathname.startsWith(path))) {
    event.respondWith(cacheFirstStrategy(request));
  }
  
  // Dynamic learning progress - always fresh
  else if (CACHE_STRATEGIES.DYNAMIC_CONTENT.some(path => url.pathname.startsWith(path))) {
    event.respondWith(networkFirstStrategy(request));
  }
  
  // Course listings - stale while revalidate
  else if (CACHE_STRATEGIES.STALE_WHILE_REVALIDATE.some(path => url.pathname.startsWith(path))) {
    event.respondWith(staleWhileRevalidateStrategy(request));
  }
});

// PWA manifest.json for educational platform
{
  "name": "Learning Management System",
  "short_name": "LMS",
  "description": "AI-powered learning platform for personalized education",
  "start_url": "/",
  "display": "standalone",
  "orientation": "portrait-primary",
  "theme_color": "#3b82f6",
  "background_color": "#ffffff",
  "categories": ["education", "productivity"],
  "lang": "en",
  "dir": "ltr",
  
  "icons": [
    {
      "src": "/static/icons/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/static/icons/icon-192x192.png",
      "sizes": "192x192", 
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/static/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable any"
    }
  ],
  
  "shortcuts": [
    {
      "name": "My Courses",
      "short_name": "Courses",
      "description": "View your enrolled courses",
      "url": "/courses",
      "icons": [{ "src": "/static/icons/courses-shortcut.png", "sizes": "96x96" }]
    },
    {
      "name": "Learning Progress",
      "short_name": "Progress", 
      "description": "Check your learning progress",
      "url": "/progress",
      "icons": [{ "src": "/static/icons/progress-shortcut.png", "sizes": "96x96" }]
    }
  ]
}
```

## Research & Continuous Improvement

**Latest 2025 Mobile Educational Design Trends**:
- Progressive Web Apps for offline learning capability
- Voice User Interface (VUI) integration for accessibility
- Haptic feedback for educational interactions
- Dark mode optimization for extended reading sessions
- Gesture-based navigation for intuitive learning flows

**Mobile Performance Optimization Research**:
- Core Web Vitals optimization for educational content loading
- Image optimization techniques for course thumbnails and diagrams
- Video streaming optimization for educational content delivery
- Bundle splitting strategies for mobile educational applications
- Service worker patterns for offline educational content access

**Accessibility Compliance for Mobile EdTech**:
- WCAG 2.2 AA compliance for educational mobile interfaces
- Screen reader optimization for complex educational content
- Keyboard navigation patterns for mobile educational workflows
- Color contrast requirements for educational content readability
- Touch target sizing for educational interaction elements

---

## 🔄 Next Research Update
**Focus**: Mobile-first educational UX patterns, PWA offline learning strategies, mobile accessibility automation testing 