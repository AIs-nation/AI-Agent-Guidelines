# Coursebox LMS Platform - AI Agent Automation Guide

**THIS GUIDE IS FOR AI AGENTS - NOT HUMANS**

This document provides automation procedures for AI agents working with the Coursebox LMS platform at course.easycyber.com.

## Platform Authentication

### Login Procedure
1. Navigate to `https://course.easycyber.com`
2. Locate email textbox by:
   - Look for textbox with placeholder text containing "email" or "Insert email"
   - Usually the first visible input field on the page
3. Type email: `nir.yossef@easycyber.com` 
4. Click "Next" button:
   - Look for button with text "Next" or "הבא"
   - Usually positioned below or adjacent to email field
5. Wait for password field to appear, then locate password textbox by:
   - Look for textbox with type="password" or placeholder containing "password"
   - Appears after clicking Next button
6. Type password: `TheBestCourse837+`
7. Click login button:
   - Look for button with text "Log in", "Login", "התחבר" or similar
   - Usually the primary action button on the form

### Post-Login State
- Dashboard URL: `https://course.easycyber.com/#/dashboard`
- Key elements present: Profile section with course count, navigation menu, AI chat button

## Course Management Automation

### Accessing Course List
**Method 1 - Via Navigation Menu:**
1. Look for left sidebar navigation menu
2. Find button/link with text "Courses" or "קורסים"
3. Click courses navigation button

**Method 2 - Via Profile Link:**
1. Look for text pattern "X Courses Enrolled" or similar in Hebrew
2. Click the enrolled courses link

**Method 3 - Direct Navigation:**
1. Navigate to `https://course.easycyber.com/#/courses/all`
2. If on "My Courses" tab showing no results:
   - Look for tab with text "All" or "הכל"
   - Click the "All" tab

### Course Name Editing Procedure

#### Step 1: Navigate to Target Course
1. From course list, locate target course by:
   - Look for course title text containing target pattern
   - Course titles are usually clickable links or buttons
2. Click course link to access course page (URL pattern: `/#/courses/[ID]/about`)

#### Step 2: Access Course Settings
1. On course page, locate settings icon by:
   - **Visual Pattern**: Look for gear/cog icon (⚙️) or three dots (⋯) symbol
   - **Position Pattern**: Settings icon is ALWAYS positioned between "Unpublish" and "AI Tutor" buttons
   - **Button Group**: Look in the action buttons area near course title
2. Click settings icon to open dropdown menu

#### Step 3: Enter Edit Mode
1. From dropdown menu, locate "Edit" option by:
   - Look for text "Edit" or "עריכה"
   - Usually first or second option in dropdown
2. Click "Edit" to open course editing interface

#### Step 4: Modify Course Title
1. Locate course title textbox by:
   - Look for textbox with attribute name="Insert a title..." or similar
   - Usually the first and most prominent textbox in edit form
   - Contains current course title as value
2. Click textbox to focus
3. Select all text using `Ctrl+A`
4. Type new course title
5. **Format Pattern**: Change "קורס הכנה X" to "קורס הכנה לכיתה ט' X"

#### Step 5: Save Changes
1. Locate "Save" button by:
   - Look for button with text "Save" or "שמור"
   - Usually positioned at bottom of form
   - Often the primary/highlighted button
2. Click Save button
3. Verify page returns to course information view with updated title

## Course Content Editor Automation

### Accessing Course Editor
1. From course page (URL pattern: `/#/courses/[ID]/about`)
2. Locate "Editor" button by:
   - Look for button with text "Editor" or "עורך"
   - Positioned in course action buttons area, usually at bottom of course information
3. Click "Editor" button to open course content editor

### Course Section Navigation
1. **URL Pattern for Sections**: `https://course.easycyber.com/#/courses/[COURSE_ID]/activities/[SECTION_ID]/course_view/edit`
2. **Section Navigation Methods**:
   - Use sidebar navigation to click on specific sections
   - Navigate directly using section URLs if section IDs are known
   - Use topic expansion to access subsections within topics

### Text Editor RTL Alignment Procedure (RECOMMENDED METHOD)

#### Step 1: Access Text Editor
1. In course editor interface, locate main text editor by:
   - Look for large textbox/editor area containing course content
   - Usually the main content area with toolbar above it
   - Contains Hebrew text content
   - **Important**: Only align the FIRST/MAIN text editor in each section
2. Click on text editor to focus

#### Step 2: Select All Text
1. Use `Ctrl+A` to select all text in the editor
2. Verify text is highlighted/selected

#### Step 3: Apply RTL Alignment (KEYBOARD METHOD - PREFERRED)
1. **RECOMMENDED**: Use keyboard shortcut `Ctrl+Shift+R`
   - This is the most reliable and efficient method
   - Works consistently across all text editors
   - No need to navigate toolbar menus
   - Faster execution for batch operations
2. Verify Hebrew text is now properly right-aligned (RTL)

#### Step 4: Verification
1. Check that Hebrew text aligns to the right side of the editor
2. Ensure text displays properly in RTL format
3. Move to next section if alignment is successful

### Text Editor RTL Alignment Procedure (ALTERNATIVE TOOLBAR METHOD)

**Use this method only if keyboard shortcut fails**

#### Step 1: Access Text Editor
1. In course editor interface, locate main text editor by:
   - Look for large textbox/editor area containing course content
   - Usually the main content area with toolbar above it
   - Contains Hebrew text content
2. Click on text editor to focus

#### Step 2: Select All Text
1. Use `Ctrl+A` to select all text in the editor
2. Verify text is highlighted/selected

#### Step 3: Apply RTL Alignment (TOOLBAR METHOD)
1. Locate alignment button in toolbar by:
   - **Visual Pattern**: Look for alignment icon (usually lines with different alignments ≡)
   - **Toolbar Section**: Located in formatting toolbar above text editor
   - **Button Group**: Usually grouped with other formatting buttons
2. Click alignment button to open alignment submenu
3. **In Alignment Submenu**:
   - Look for submenu with alignment options (usually displays 4 alignment buttons in first row, 2 in second row)
   - **Target**: Find the THIRD alignment button from the left in the TOP ROW
   - **Position**: This is the RIGHT ALIGN button (typically shows lines aligned to right)
   - **Critical**: Must use the third button specifically - this applies proper RTL alignment for Hebrew text
4. Click the THIRD alignment button in the submenu (top row, third from left)
5. Verify Hebrew text is now properly right-aligned (RTL)

### Interactive Component RTL Alignment

#### Component Types Requiring RTL Alignment
1. **Flip Cards**: Each card textbox needs individual RTL alignment
2. **Info Tabs**: Each tab textbox needs individual RTL alignment  
3. **Accordion Sections**: Each accordion textbox needs individual RTL alignment
4. **Main Text Editors**: Primary content areas in each section

#### Component Alignment Procedure
1. **For Flip Cards**:
   - Click on each flip card textbox individually
   - Apply `Ctrl+A` to select all text
   - Apply `Ctrl+Shift+R` for RTL alignment
   - Repeat for each card in the component

2. **For Info Tabs**:
   - Click on each tab textbox individually
   - Apply `Ctrl+A` to select all text
   - Apply `Ctrl+Shift+R` for RTL alignment
   - Repeat for each tab in the component

3. **For Accordion Sections**:
   - Click on each accordion textbox individually
   - Apply `Ctrl+A` to select all text
   - Apply `Ctrl+Shift+R` for RTL alignment
   - Repeat for each accordion section in the component

### RTL Alignment Best Practices
- **ALWAYS use keyboard shortcut `Ctrl+Shift+R` as primary method**
- **Only align text editors/textboxes - do not align interactive games or assessments**
- **Focus on main content areas first, then interactive components**
- **Take screenshots to verify alignment before proceeding if uncertain**
- **Process sections systematically to avoid missing any content**
- **Wait for page loading between section navigation**

## Element Identification Strategies

### Reliable Identification Methods

#### By Text Content
- **Login Elements**: "email", "password", "Next", "Log in"
- **Navigation**: "Courses", "Dashboard", "All"
- **Course Actions**: "Edit", "Save", "Editor", "Unpublish", "AI Tutor"
- **Alignment**: Look for alignment symbols or "Right align" text

#### By Visual Patterns
- **Settings Icon**: Gear (⚙️) or three dots (⋯) symbol
- **Alignment Button**: Lines with different alignments (≡)
- **Right Align**: Lines aligned to right side (⋯)

#### By Position Patterns
- **Settings Icon**: ALWAYS between "Unpublish" and "AI Tutor" buttons
- **Email Field**: First input field on login page
- **Password Field**: Appears after clicking Next
- **Course Title**: First/main textbox in edit form
- **Save Button**: Bottom of forms, usually primary button
- **Editor Button**: Bottom of course information section
- **Alignment Button**: In formatting toolbar above text editor
- **RTL Alignment**: Third button in alignment submenu (top row) - OR use `Ctrl+Shift+R`

#### By HTML Attributes
- **Email Field**: placeholder containing "email" or name attribute
- **Password Field**: type="password"
- **Course Title**: name="Insert a title..." or similar
- **Textboxes**: role="textbox" or input elements

### Fallback Strategies

#### If Element Not Found by Primary Method
1. **Take Snapshot**: Use browser snapshot to see current page state
2. **Search by Similar Text**: Look for partial matches or translations
3. **Search by Position**: Look in expected UI areas (toolbars, forms, navigation)
4. **Search by Element Type**: Find all buttons/textboxes and filter by context

#### Navigation Fallbacks
- **Direct URL**: Use direct navigation when UI navigation fails
- **Breadcrumbs**: Use page breadcrumbs for navigation context
- **URL Patterns**: Verify current page by URL structure

## Automation Patterns

### Error Handling
- Always verify page state before attempting actions
- Use text-based identification as primary method
- Fall back to position-based identification if text fails
- Take snapshots when elements cannot be located
- Verify successful actions before proceeding
- **For RTL alignment**: Take screenshot first to check if already aligned

### Course Processing Workflow
1. Login to platform using text-based element identification
2. Navigate to course list using navigation menu
3. For each target course:
   - Find course by title text
   - Access course page
   - Find settings icon (between "Unpublish" and "AI Tutor")
   - Click "Edit" from dropdown
   - Find course title textbox (first major textbox)
   - Update title following pattern
   - Click "Save" button (bottom of form)
   - Verify update success

### Text Editor RTL Workflow (UPDATED)
1. Access course page
2. Find and click "Editor" button (bottom of course info)
3. Navigate to first section (ensure starting from beginning)
4. For each section:
   - Wait for page to load completely
   - Take screenshot to verify current alignment status
   - If not aligned:
     - Find and click main text editor (large content area)
     - Select all text (`Ctrl+A`)
     - **Apply RTL alignment using `Ctrl+Shift+R`** (PREFERRED METHOD)
     - Verify RTL alignment applied correctly
   - For interactive components (flip cards, info tabs, accordions):
     - Click each individual textbox
     - Apply `Ctrl+A` then `Ctrl+Shift+R`
     - Repeat for all textboxes in component
   - Navigate to next section
5. Continue until all sections processed

### Batch Operations
- Process courses sequentially to avoid session issues
- Return to course list between edits using navigation
- Verify each change before proceeding to next course
- For text alignment: Apply to all courses requiring Hebrew RTL display
- **Use keyboard shortcut `Ctrl+Shift+R` for consistent RTL operations**
- **Take screenshots to verify alignment status before attempting changes**

## Platform-Specific Notes

### Interface Behavior
- Some navigation requires UI clicks rather than direct URLs
- Settings menu appears as dropdown after icon click
- Edit mode replaces course display with form interface
- Save action returns to course information view
- **Keyboard shortcuts work more reliably than toolbar buttons**
- RTL alignment essential for Hebrew text readability
- **Page loading time varies - always wait for complete loading**

### Element Stability Patterns
- **Settings Icon**: Consistently positioned between "Unpublish" and "AI Tutor"
- **Course Title**: Always first major textbox in edit mode
- **Editor Button**: Always at bottom of course information section
- **Keyboard Shortcuts**: `Ctrl+Shift+R` works consistently across all text editors
- **Save Buttons**: Always at bottom of forms
- **Navigation Menu**: Always in left sidebar

### Language Considerations
- Interface may display in Hebrew or English
- Button text may be bilingual
- Look for both Hebrew and English text patterns
- Course titles are primarily in Hebrew
- RTL alignment critical for Hebrew content readability

### Session Management
- Login session persists across page navigation
- Direct URL navigation works after login
- Logout occurs automatically after extended inactivity
- Re-login required if session expires during batch operations

### RTL Alignment Best Practices (UPDATED)
- **PRIMARY METHOD**: Always use `Ctrl+Shift+R` keyboard shortcut for Hebrew RTL
- **VERIFICATION**: Take screenshots to check alignment status before making changes
- **SCOPE**: Apply RTL to all Hebrew content sections (main text + interactive components)
- **EFFICIENCY**: Keyboard method is faster and more reliable than toolbar navigation
- **CONSISTENCY**: Use same method across all content types for uniform results
- **FOCUS**: Only align text editors/textboxes - skip games and non-text content
