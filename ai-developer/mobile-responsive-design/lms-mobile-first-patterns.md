# Mobile-First Responsive Design for LMS Educational Platforms (2025)

## 🔄 Fundamental Principle: Constant Research & Web Updates
**Critical Success Factor**: This guide must be continuously updated with latest 2025+ web research on mobile-first design patterns for educational platforms. All team members must regularly research and update this knowledge base.

## Educational Mobile-First Design Architecture

### 1. Educational Mobile-First CSS Patterns

#### ✅ Good Example: Educational Mobile-First Responsive Design
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ EDUCATIONAL MOBILE-FIRST DESIGN ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
**Mobile Standards**: iOS Safari 14+, Chrome Mobile 88+, Educational Accessibility AA

**Educational Mobile-First CSS Architecture**:
```scss
// Good: Educational mobile-first responsive design system
/* Educational Design Tokens for Mobile Learning */
:root {
  /* Educational Mobile Typography Scale */
  --educational-font-size-xs: 0.75rem;    /* 12px - Small text */
  --educational-font-size-sm: 0.875rem;   /* 14px - Body text */
  --educational-font-size-base: 1rem;     /* 16px - Base reading */
  --educational-font-size-lg: 1.125rem;   /* 18px - Emphasized text */
  --educational-font-size-xl: 1.25rem;    /* 20px - Lesson titles */
  --educational-font-size-2xl: 1.5rem;    /* 24px - Course titles */
  --educational-font-size-3xl: 2rem;      /* 32px - Main headings */

  /* Educational Mobile Spacing System */
  --educational-spacing-xs: 0.25rem;      /* 4px - Tight spacing */
  --educational-spacing-sm: 0.5rem;       /* 8px - Component spacing */
  --educational-spacing-md: 1rem;         /* 16px - Section spacing */
  --educational-spacing-lg: 1.5rem;       /* 24px - Layout spacing */
  --educational-spacing-xl: 2rem;         /* 32px - Page spacing */
  --educational-spacing-2xl: 3rem;        /* 48px - Large breaks */

  /* Educational Touch Target Sizes */
  --educational-touch-min: 44px;          /* Minimum touch target */
  --educational-touch-comfortable: 48px;   /* Comfortable touch */
  --educational-touch-large: 56px;        /* Large interactive elements */

  /* Educational Reading Optimized Colors */
  --educational-text-primary: #1a1a1a;
  --educational-text-secondary: #4a4a4a;
  --educational-text-muted: #6b7280;
  --educational-background: #ffffff;
  --educational-background-alt: #f8fafc;
  --educational-accent: #3b82f6;
  --educational-success: #10b981;
  --educational-warning: #f59e0b;
  --educational-error: #ef4444;

  /* Educational Mobile Breakpoints */
  --educational-mobile-s: 320px;          /* Small phones */
  --educational-mobile-m: 375px;          /* Medium phones */
  --educational-mobile-l: 414px;          /* Large phones */
  --educational-tablet: 768px;            /* Tablets */
  --educational-desktop: 1024px;          /* Desktop */
  --educational-desktop-lg: 1440px;       /* Large desktop */
}

/* Educational Mobile-First Base Styles */
.educational-platform {
  /* Mobile-first foundation for educational content */
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  font-size: var(--educational-font-size-base);
  line-height: 1.6; /* Optimal for reading on mobile */
  color: var(--educational-text-primary);
  background-color: var(--educational-background);
  
  /* Educational mobile optimizations */
  -webkit-text-size-adjust: 100%; /* Prevent iOS text scaling */
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-rendering: optimizeLegibility;
  
  /* Educational touch optimization */
  touch-action: manipulation; /* Disable double-tap zoom */
  -webkit-tap-highlight-color: transparent;
}

/* Educational Course Card - Mobile First */
.educational-course-card {
  /* Mobile: Full width, card layout */
  width: 100%;
  padding: var(--educational-spacing-md);
  margin-bottom: var(--educational-spacing-md);
  background: var(--educational-background);
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  
  /* Educational mobile touch optimization */
  min-height: var(--educational-touch-large);
  cursor: pointer;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  
  /* Mobile interaction feedback */
  &:active {
    transform: scale(0.98);
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.15);
  }
  
  /* Educational content layout - mobile first */
  .course-header {
    display: flex;
    align-items: flex-start;
    gap: var(--educational-spacing-sm);
    margin-bottom: var(--educational-spacing-md);
    
    .course-image {
      width: 60px;
      height: 60px;
      border-radius: 8px;
      object-fit: cover;
      flex-shrink: 0;
    }
    
    .course-info {
      flex: 1;
      min-width: 0; /* Prevent text overflow */
      
      .course-title {
        font-size: var(--educational-font-size-lg);
        font-weight: 600;
        line-height: 1.4;
        margin: 0 0 var(--educational-spacing-xs) 0;
        
        /* Educational mobile text truncation */
        display: -webkit-box;
        -webkit-line-clamp: 2;
        -webkit-box-orient: vertical;
        overflow: hidden;
      }
      
      .course-instructor {
        font-size: var(--educational-font-size-sm);
        color: var(--educational-text-secondary);
        margin: 0;
      }
    }
  }
  
  .course-progress {
    margin-bottom: var(--educational-spacing-md);
    
    .progress-bar {
      width: 100%;
      height: 8px;
      background: var(--educational-background-alt);
      border-radius: 4px;
      overflow: hidden;
      
      .progress-fill {
        height: 100%;
        background: linear-gradient(90deg, var(--educational-accent), #60a5fa);
        border-radius: 4px;
        transition: width 0.3s ease;
      }
    }
    
    .progress-text {
      font-size: var(--educational-font-size-sm);
      color: var(--educational-text-secondary);
      margin-top: var(--educational-spacing-xs);
      text-align: right;
    }
  }
  
  .course-actions {
    display: flex;
    gap: var(--educational-spacing-sm);
    
    .educational-button {
      flex: 1;
      padding: var(--educational-spacing-sm) var(--educational-spacing-md);
      min-height: var(--educational-touch-min);
      border: none;
      border-radius: 8px;
      font-size: var(--educational-font-size-sm);
      font-weight: 500;
      cursor: pointer;
      transition: all 0.2s ease;
      
      /* Educational mobile accessibility */
      &:focus-visible {
        outline: 2px solid var(--educational-accent);
        outline-offset: 2px;
      }
      
      &.primary {
        background: var(--educational-accent);
        color: white;
        
        &:active {
          background: #2563eb;
        }
      }
      
      &.secondary {
        background: var(--educational-background-alt);
        color: var(--educational-text-primary);
        
        &:active {
          background: #e2e8f0;
        }
      }
    }
  }
  
  /* Tablet: 2-column grid adaptation */
  @media (min-width: 768px) {
    display: grid;
    grid-template-columns: auto 1fr auto;
    grid-template-rows: auto auto;
    gap: var(--educational-spacing-md);
    
    .course-header {
      grid-column: 1 / -1;
      margin-bottom: 0;
    }
    
    .course-progress {
      margin-bottom: 0;
    }
    
    .course-actions {
      grid-column: 3;
      grid-row: 2;
      flex-direction: column;
      width: 120px;
    }
  }
  
  /* Desktop: Enhanced layout */
  @media (min-width: 1024px) {
    padding: var(--educational-spacing-lg);
    
    .course-header .course-image {
      width: 80px;
      height: 80px;
    }
    
    .course-header .course-info .course-title {
      font-size: var(--educational-font-size-xl);
    }
    
    .course-actions {
      width: 140px;
    }
  }
}

/* Educational Lesson Layout - Mobile First */
.educational-lesson-container {
  /* Mobile: Full viewport utilization */
  min-height: 100vh;
  padding: var(--educational-spacing-md);
  background: var(--educational-background);
  
  /* Educational mobile navigation */
  .lesson-header {
    position: sticky;
    top: 0;
    z-index: 10;
    background: var(--educational-background);
    padding: var(--educational-spacing-sm) 0;
    margin: calc(-1 * var(--educational-spacing-md)) calc(-1 * var(--educational-spacing-md)) var(--educational-spacing-lg);
    padding-left: var(--educational-spacing-md);
    padding-right: var(--educational-spacing-md);
    border-bottom: 1px solid var(--educational-background-alt);
    
    .lesson-nav {
      display: flex;
      align-items: center;
      justify-content: space-between;
      
      .back-button {
        display: flex;
        align-items: center;
        gap: var(--educational-spacing-xs);
        padding: var(--educational-spacing-xs) var(--educational-spacing-sm);
        min-height: var(--educational-touch-min);
        background: none;
        border: none;
        color: var(--educational-accent);
        font-size: var(--educational-font-size-sm);
        cursor: pointer;
        
        &:active {
          opacity: 0.7;
        }
      }
      
      .lesson-progress {
        font-size: var(--educational-font-size-sm);
        color: var(--educational-text-secondary);
      }
    }
    
    .lesson-title {
      font-size: var(--educational-font-size-xl);
      font-weight: 600;
      margin: var(--educational-spacing-sm) 0 0;
      line-height: 1.3;
      
      /* Mobile text handling */
      display: -webkit-box;
      -webkit-line-clamp: 2;
      -webkit-box-orient: vertical;
      overflow: hidden;
    }
  }
  
  .lesson-content {
    max-width: 100%;
    
    /* Educational reading optimization for mobile */
    p, li {
      font-size: var(--educational-font-size-base);
      line-height: 1.7;
      margin-bottom: var(--educational-spacing-md);
      
      /* Prevent orphaned words on mobile */
      text-wrap: balance;
    }
    
    h2 {
      font-size: var(--educational-font-size-xl);
      font-weight: 600;
      margin: var(--educational-spacing-xl) 0 var(--educational-spacing-md);
      line-height: 1.3;
    }
    
    h3 {
      font-size: var(--educational-font-size-lg);
      font-weight: 600;
      margin: var(--educational-spacing-lg) 0 var(--educational-spacing-sm);
      line-height: 1.4;
    }
    
    /* Educational media responsive handling */
    img, video {
      max-width: 100%;
      height: auto;
      border-radius: 8px;
      margin: var(--educational-spacing-md) 0;
    }
    
    /* Educational code blocks for mobile */
    pre, code {
      font-family: 'SF Mono', Monaco, 'Cascadia Code', monospace;
      background: var(--educational-background-alt);
      border-radius: 6px;
      
      /* Mobile code optimization */
      overflow-x: auto;
      -webkit-overflow-scrolling: touch;
    }
    
    pre {
      padding: var(--educational-spacing-md);
      font-size: var(--educational-font-size-sm);
      line-height: 1.5;
    }
    
    code {
      padding: 2px 4px;
      font-size: 0.9em;
    }
  }
  
  .lesson-footer {
    margin-top: var(--educational-spacing-2xl);
    padding-top: var(--educational-spacing-lg);
    border-top: 1px solid var(--educational-background-alt);
    
    .lesson-actions {
      display: flex;
      gap: var(--educational-spacing-sm);
      
      .educational-button {
        flex: 1;
        padding: var(--educational-spacing-md);
        min-height: var(--educational-touch-large);
        border: none;
        border-radius: 12px;
        font-size: var(--educational-font-size-base);
        font-weight: 500;
        cursor: pointer;
        transition: all 0.2s ease;
        
        &.primary {
          background: var(--educational-accent);
          color: white;
          
          &:active {
            background: #2563eb;
          }
        }
        
        &.secondary {
          background: var(--educational-background-alt);
          color: var(--educational-text-primary);
          
          &:active {
            background: #e2e8f0;
          }
        }
        
        &:disabled {
          opacity: 0.5;
          cursor: not-allowed;
        }
      }
    }
  }
  
  /* Tablet: Improved reading layout */
  @media (min-width: 768px) {
    padding: var(--educational-spacing-lg) var(--educational-spacing-xl);
    
    .lesson-content {
      max-width: 65ch; /* Optimal reading width */
      margin: 0 auto;
    }
    
    .lesson-footer .lesson-actions {
      max-width: 400px;
      margin: 0 auto;
    }
  }
  
  /* Desktop: Enhanced layout */
  @media (min-width: 1024px) {
    display: grid;
    grid-template-columns: 1fr 300px;
    gap: var(--educational-spacing-2xl);
    max-width: 1200px;
    margin: 0 auto;
    padding: var(--educational-spacing-xl);
    
    .lesson-main {
      order: 1;
    }
    
    .lesson-sidebar {
      order: 2;
      position: sticky;
      top: var(--educational-spacing-lg);
      height: fit-content;
    }
  }
}

/* Educational Touch Optimization */
.educational-interactive {
  /* Ensure all interactive elements meet touch standards */
  min-height: var(--educational-touch-min);
  min-width: var(--educational-touch-min);
  
  /* Visual feedback for touch interactions */
  &:active {
    transform: scale(0.97);
    transition: transform 0.1s ease;
  }
  
  /* High contrast focus for accessibility */
  &:focus-visible {
    outline: 3px solid var(--educational-accent);
    outline-offset: 2px;
  }
}

/* Educational Loading States for Mobile */
.educational-skeleton {
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
  background-size: 200% 100%;
  animation: educational-loading 1.5s infinite;
  border-radius: 4px;
}

@keyframes educational-loading {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

/* Educational Print Styles */
@media print {
  .educational-lesson-container {
    /* Optimize for educational material printing */
    .lesson-header .lesson-nav { display: none; }
    .lesson-footer { display: none; }
    
    .lesson-content {
      max-width: none;
      
      h2 { break-after: avoid; }
      h3 { break-after: avoid; }
      
      pre, blockquote {
        break-inside: avoid;
      }
    }
  }
}
```
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ EDUCATIONAL MOBILE-FIRST DESIGN ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#### ❌ Bad Example: Desktop-First Design Without Mobile Considerations
```scss
// Bad: Desktop-first without mobile educational optimization
.course-card {
  width: 300px; // Fixed width, not responsive
  padding: 20px;
  
  .title {
    font-size: 24px; // Too large for mobile
  }
  
  .button {
    padding: 8px 12px; // Too small for touch
    min-height: 30px; // Below touch standards
  }
  
  // No mobile breakpoints
  // No touch optimization
  // No educational accessibility considerations
}
```

### 2. Educational Mobile Component Patterns

#### ✅ Good Example: Educational Mobile Component Architecture
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ EDUCATIONAL MOBILE COMPONENTS ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
**Mobile Component System**: React + CSS-in-JS with educational mobile patterns

**Educational Mobile Components**:
```typescript
// Good: Educational mobile-optimized React components
import React, { useState, useRef, useEffect } from 'react';
import styled, { css } from 'styled-components';

// Educational mobile-first button system
const EducationalButton = styled.button<{
  variant: 'primary' | 'secondary' | 'ghost';
  size: 'small' | 'medium' | 'large';
  fullWidth?: boolean;
}>`
  /* Educational mobile-first button foundation */
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  border: none;
  border-radius: 12px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
  font-family: inherit;
  
  /* Educational touch optimization */
  min-height: 44px; /* iOS minimum touch target */
  min-width: 44px;
  touch-action: manipulation;
  -webkit-tap-highlight-color: transparent;
  
  /* Educational accessibility */
  &:focus-visible {
    outline: 3px solid var(--educational-accent);
    outline-offset: 2px;
  }
  
  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
    transform: none !important;
  }
  
  /* Educational mobile feedback */
  &:active:not(:disabled) {
    transform: scale(0.97);
  }
  
  /* Size variants for educational contexts */
  ${props => props.size === 'small' && css`
    padding: 0.5rem 0.75rem;
    font-size: 0.875rem;
    min-height: 40px;
  `}
  
  ${props => props.size === 'medium' && css`
    padding: 0.75rem 1rem;
    font-size: 1rem;
    min-height: 44px;
  `}
  
  ${props => props.size === 'large' && css`
    padding: 1rem 1.5rem;
    font-size: 1.125rem;
    min-height: 48px;
  `}
  
  /* Educational color variants */
  ${props => props.variant === 'primary' && css`
    background: var(--educational-accent);
    color: white;
    
    &:active:not(:disabled) {
      background: #2563eb;
    }
  `}
  
  ${props => props.variant === 'secondary' && css`
    background: var(--educational-background-alt);
    color: var(--educational-text-primary);
    
    &:active:not(:disabled) {
      background: #e2e8f0;
    }
  `}
  
  ${props => props.variant === 'ghost' && css`
    background: transparent;
    color: var(--educational-accent);
    
    &:active:not(:disabled) {
      background: rgba(59, 130, 246, 0.1);
    }
  `}
  
  /* Full width for mobile layouts */
  ${props => props.fullWidth && css`
    width: 100%;
  `}
  
  /* Tablet and desktop adjustments */
  @media (min-width: 768px) {
    &:hover:not(:disabled) {
      transform: translateY(-1px);
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
    }
  }
`;

// Educational mobile-optimized course card
const EducationalCourseCard = styled.div`
  /* Mobile-first educational card layout */
  width: 100%;
  padding: 1rem;
  background: white;
  border-radius: 16px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  cursor: pointer;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  
  /* Mobile touch optimization */
  touch-action: manipulation;
  -webkit-tap-highlight-color: transparent;
  
  &:active {
    transform: scale(0.98);
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.15);
  }
  
  /* Tablet: Enhanced hover states */
  @media (min-width: 768px) {
    &:hover {
      transform: translateY(-2px);
      box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
    }
  }
  
  /* Desktop: Grid layout optimizations */
  @media (min-width: 1024px) {
    padding: 1.5rem;
  }
`;

// Educational mobile progress indicator
const EducationalProgressBar = styled.div<{ progress: number; animated?: boolean }>`
  width: 100%;
  height: 8px;
  background: var(--educational-background-alt);
  border-radius: 4px;
  overflow: hidden;
  position: relative;
  
  &::after {
    content: '';
    position: absolute;
    left: 0;
    top: 0;
    height: 100%;
    width: ${props => props.progress}%;
    background: linear-gradient(90deg, var(--educational-accent), #60a5fa);
    border-radius: 4px;
    transition: width 0.5s ease;
    
    ${props => props.animated && css`
      background-size: 20px 20px;
      background-image: linear-gradient(
        45deg,
        rgba(255, 255, 255, 0.2) 25%,
        transparent 25%,
        transparent 50%,
        rgba(255, 255, 255, 0.2) 50%,
        rgba(255, 255, 255, 0.2) 75%,
        transparent 75%,
        transparent
      );
      animation: educational-progress 1s linear infinite;
    `}
  }
  
  @keyframes educational-progress {
    0% { background-position: 0 0; }
    100% { background-position: 20px 0; }
  }
  
  /* Larger progress bar for better visibility on mobile */
  @media (max-width: 767px) {
    height: 10px;
  }
`;

// Educational mobile video player component
const EducationalVideoPlayer: React.FC<{
  src: string;
  poster?: string;
  captions?: string;
  onProgress?: (progress: number) => void;
}> = ({ src, poster, captions, onProgress }) => {
  const videoRef = useRef<HTMLVideoElement>(null);
  const [isPlaying, setIsPlaying] = useState(false);
  const [progress, setProgress] = useState(0);
  const [showControls, setShowControls] = useState(false);
  
  // Educational video progress tracking
  useEffect(() => {
    const video = videoRef.current;
    if (!video) return;
    
    const handleProgress = () => {
      const progressPercent = (video.currentTime / video.duration) * 100;
      setProgress(progressPercent);
      onProgress?.(progressPercent);
    };
    
    const handlePlay = () => setIsPlaying(true);
    const handlePause = () => setIsPlaying(false);
    
    video.addEventListener('timeupdate', handleProgress);
    video.addEventListener('play', handlePlay);
    video.addEventListener('pause', handlePause);
    
    return () => {
      video.removeEventListener('timeupdate', handleProgress);
      video.removeEventListener('play', handlePlay);
      video.removeEventListener('pause', handlePause);
    };
  }, [onProgress]);
  
  const togglePlay = () => {
    const video = videoRef.current;
    if (!video) return;
    
    if (isPlaying) {
      video.pause();
    } else {
      video.play();
    }
  };
  
  return (
    <VideoContainer
      onMouseEnter={() => setShowControls(true)}
      onMouseLeave={() => setShowControls(false)}
      onTouchStart={() => setShowControls(true)}
    >
      <Video
        ref={videoRef}
        src={src}
        poster={poster}
        playsInline // Critical for iOS mobile playback
        preload="metadata"
        onClick={togglePlay}
      >
        {captions && <track kind="captions" src={captions} srcLang="en" default />}
      </Video>
      
      <VideoControls visible={showControls || !isPlaying}>
        <PlayButton onClick={togglePlay} isPlaying={isPlaying}>
          {isPlaying ? '⏸️' : '▶️'}
        </PlayButton>
        
        <ProgressContainer>
          <EducationalProgressBar progress={progress} />
        </ProgressContainer>
      </VideoControls>
      
      {/* Educational accessibility enhancements */}
      <VisuallyHidden>
        Video progress: {Math.round(progress)}%
      </VisuallyHidden>
    </VideoContainer>
  );
};

const VideoContainer = styled.div`
  position: relative;
  width: 100%;
  border-radius: 12px;
  overflow: hidden;
  background: black;
  
  /* Educational mobile video optimization */
  @media (max-width: 767px) {
    border-radius: 8px;
  }
`;

const Video = styled.video`
  width: 100%;
  height: auto;
  display: block;
  
  /* Educational mobile video handling */
  object-fit: contain;
  max-height: 50vh; /* Prevent videos from dominating mobile screens */
  
  @media (min-width: 768px) {
    max-height: 60vh;
  }
`;

const VideoControls = styled.div<{ visible: boolean }>`
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: linear-gradient(transparent, rgba(0, 0, 0, 0.7));
  padding: 1rem;
  display: flex;
  align-items: center;
  gap: 1rem;
  opacity: ${props => props.visible ? 1 : 0};
  transition: opacity 0.3s ease;
  
  /* Educational mobile touch optimization */
  @media (max-width: 767px) {
    padding: 0.75rem;
    gap: 0.75rem;
  }
`;

const PlayButton = styled.button<{ isPlaying: boolean }>`
  background: rgba(255, 255, 255, 0.9);
  border: none;
  border-radius: 50%;
  width: 48px;
  height: 48px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 1.25rem;
  transition: all 0.2s ease;
  
  /* Educational mobile touch optimization */
  min-width: 44px;
  min-height: 44px;
  touch-action: manipulation;
  
  &:active {
    transform: scale(0.95);
    background: rgba(255, 255, 255, 0.8);
  }
  
  &:focus-visible {
    outline: 2px solid white;
    outline-offset: 2px;
  }
`;

const ProgressContainer = styled.div`
  flex: 1;
  margin: 0 0.5rem;
`;

const VisuallyHidden = styled.span`
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
`;

// Educational mobile navigation component
const EducationalMobileNav: React.FC<{
  title: string;
  onBack?: () => void;
  actions?: React.ReactNode;
}> = ({ title, onBack, actions }) => {
  return (
    <MobileNavContainer>
      <NavContent>
        {onBack && (
          <BackButton onClick={onBack}>
            <BackIcon>←</BackIcon>
            <span>Back</span>
          </BackButton>
        )}
        
        <NavTitle>{title}</NavTitle>
        
        <NavActions>
          {actions}
        </NavActions>
      </NavContent>
    </MobileNavContainer>
  );
};

const MobileNavContainer = styled.nav`
  position: sticky;
  top: 0;
  z-index: 50;
  background: var(--educational-background);
  border-bottom: 1px solid var(--educational-background-alt);
  padding: 0.5rem 1rem;
  
  /* Educational mobile safe area handling */
  padding-top: max(0.5rem, env(safe-area-inset-top));
`;

const NavContent = styled.div`
  display: grid;
  grid-template-columns: auto 1fr auto;
  align-items: center;
  gap: 1rem;
  max-width: 100%;
`;

const BackButton = styled.button`
  display: flex;
  align-items: center;
  gap: 0.25rem;
  padding: 0.5rem;
  background: none;
  border: none;
  color: var(--educational-accent);
  font-size: 0.875rem;
  cursor: pointer;
  border-radius: 8px;
  min-height: 44px; /* Educational touch target */
  
  &:active {
    background: rgba(59, 130, 246, 0.1);
  }
  
  &:focus-visible {
    outline: 2px solid var(--educational-accent);
    outline-offset: 2px;
  }
`;

const BackIcon = styled.span`
  font-size: 1.25rem;
  line-height: 1;
`;

const NavTitle = styled.h1`
  font-size: 1.125rem;
  font-weight: 600;
  margin: 0;
  text-align: center;
  
  /* Educational mobile text truncation */
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
`;

const NavActions = styled.div`
  display: flex;
  align-items: center;
  gap: 0.5rem;
  justify-self: end;
`;
```
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ EDUCATIONAL MOBILE COMPONENTS ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#### ❌ Bad Example: Non-Mobile Optimized Components
```typescript
// Bad: No mobile optimization for educational components
const CourseCard = styled.div`
  width: 300px; // Fixed width
  padding: 20px;
  
  .title {
    font-size: 24px; // Too large for mobile
  }
  
  .button {
    padding: 8px 12px; // Below touch standards
    font-size: 12px; // Too small for mobile
  }
  
  // No touch optimization
  // No mobile breakpoints
  // No educational accessibility
`;

const VideoPlayer = ({ src }) => (
  <video src={src} controls width="800" height="600" />
  // Fixed dimensions, no mobile optimization
  // No educational progress tracking
  // No accessibility features
);
```

## Educational Mobile Design Best Practices Summary

### ✅ DO's for Educational Mobile Design:
- ✅ Design mobile-first with educational reading optimization
- ✅ Use minimum 44px touch targets for all interactive educational elements
- ✅ Implement educational progress indicators optimized for mobile viewing
- ✅ Create responsive educational typography with optimal reading line lengths
- ✅ Use educational color schemes with high contrast for mobile readability
- ✅ Implement educational video players with mobile-optimized controls
- ✅ Design educational navigation with thumb-friendly positioning
- ✅ Use educational loading states and skeleton screens for mobile performance
- ✅ Implement proper educational focus management for mobile accessibility
- ✅ Test educational functionality across various mobile devices and orientations

### ❌ DON'Ts for Educational Mobile Design:
- ❌ Design desktop-first without mobile educational considerations
- ❌ Use touch targets smaller than 44px for educational interactions
- ❌ Create fixed-width layouts that don't adapt to educational mobile contexts
- ❌ Use small fonts that hinder educational content readability on mobile
- ❌ Ignore educational video optimization for mobile bandwidth and playback
- ❌ Create complex navigation that's difficult for educational mobile users
- ❌ Skip educational loading states that leave mobile users uncertain
- ❌ Use hover-dependent interactions for educational mobile functionality
- ❌ Ignore educational safe area handling for modern mobile devices
- ❌ Skip testing educational flows on actual mobile devices

Remember: Keep this mobile design guide updated with latest mobile-first patterns and educational accessibility standards. Research and validate these patterns against current 2025+ mobile design and educational technology best practices. 