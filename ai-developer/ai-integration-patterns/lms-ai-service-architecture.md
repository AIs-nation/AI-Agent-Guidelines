# LMS AI Service Architecture and Integration Patterns

## Claude API Integration for Educational Content Generation

### Production-Ready Claude Service Implementation
**Critical for LMS course generation with sophisticated content quality**

✅ **Good Example: Robust Claude Service Architecture**
```typescript
// services/claudeService.ts
import Anthropic from '@anthropic-ai/sdk';
import { rateLimit } from '../utils/rateLimit';
import { validateCourseContent } from '../utils/contentValidation';

interface CourseGenerationConfig {
  maxTokens: number;
  temperature: number;
  topP: number;
  retryAttempts: number;
  timeoutMs: number;
}

export class ClaudeService {
  private client: Anthropic;
  private config: CourseGenerationConfig;

  constructor() {
    this.client = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
      timeout: 30000 // 30 second timeout
    });

    this.config = {
      maxTokens: 4000,
      temperature: 0.7,
      topP: 0.9,
      retryAttempts: 3,
      timeoutMs: 30000
    };
  }

  @rateLimit(10, 60000) // 10 requests per minute
  async generateCourseOutline(prompt: string, difficulty: string): Promise<CourseOutline> {
    const systemPrompt = this.buildSystemPrompt('course_outline', difficulty);
    
    try {
      const response = await this.executeWithRetry(async () => {
        return await this.client.messages.create({
          model: 'claude-3-5-sonnet-20241022',
          max_tokens: this.config.maxTokens,
          temperature: this.config.temperature,
          top_p: this.config.topP,
          system: systemPrompt,
          messages: [
            {
              role: 'user',
              content: this.formatCoursePrompt(prompt, difficulty)
            }
          ]
        });
      });

      const courseOutline = this.parseCourseOutline(response.content[0].text);
      
      // Validate generated content quality
      const validation = await validateCourseContent(courseOutline);
      if (!validation.isValid) {
        throw new Error(`Course outline validation failed: ${validation.errors.join(', ')}`);
      }

      return courseOutline;
    } catch (error) {
      console.error('Claude course outline generation failed:', error);
      throw new Error(`Course outline generation failed: ${error.message}`);
    }
  }

  @rateLimit(15, 60000) // 15 requests per minute for lesson content
  async generateLessonContent(
    courseContext: CourseContext,
    lessonTitle: string,
    lessonOrder: number
  ): Promise<LessonContent> {
    const systemPrompt = this.buildSystemPrompt('lesson_content', courseContext.difficulty);
    
    try {
      const response = await this.executeWithRetry(async () => {
        return await this.client.messages.create({
          model: 'claude-3-5-sonnet-20241022',
          max_tokens: 6000, // More tokens for detailed lesson content
          temperature: 0.6, // Slightly lower for consistency
          top_p: 0.85,
          system: systemPrompt,
          messages: [
            {
              role: 'user',
              content: this.formatLessonPrompt(courseContext, lessonTitle, lessonOrder)
            }
          ]
        });
      });

      const lessonContent = this.parseLessonContent(response.content[0].text);
      
      // Ensure lesson has required educational components
      if (!this.validateLessonStructure(lessonContent)) {
        throw new Error('Generated lesson does not meet educational standards');
      }

      return lessonContent;
    } catch (error) {
      console.error('Claude lesson generation failed:', error);
      throw new Error(`Lesson content generation failed: ${error.message}`);
    }
  }

  // Advanced streaming for real-time course generation
  async *generateCourseStream(
    prompt: string,
    difficulty: string,
    onProgress?: (stage: string, progress: number) => void
  ): AsyncGenerator<CourseGenerationUpdate, void, unknown> {
    try {
      // Stage 1: Course Outline
      onProgress?.('Generating course outline', 10);
      yield { stage: 'outline', status: 'in_progress', data: null };
      
      const outline = await this.generateCourseOutline(prompt, difficulty);
      yield { stage: 'outline', status: 'complete', data: outline };
      onProgress?.('Course outline generated', 25);

      // Stage 2: Module Details
      onProgress?.('Generating module details', 40);
      yield { stage: 'modules', status: 'in_progress', data: null };
      
      const modules = await this.generateModuleDetails(outline);
      yield { stage: 'modules', status: 'complete', data: modules };
      onProgress?.('Module details generated', 60);

      // Stage 3: Lesson Content
      onProgress?.('Generating lesson content', 75);
      yield { stage: 'lessons', status: 'in_progress', data: null };
      
      const lessons = await this.generateAllLessons(outline, modules);
      yield { stage: 'lessons', status: 'complete', data: lessons };
      onProgress?.('Lesson content generated', 90);

      // Stage 4: Final Assembly
      onProgress?.('Assembling course', 95);
      const completeCourse = this.assembleCourse(outline, modules, lessons);
      yield { stage: 'complete', status: 'complete', data: completeCourse };
      onProgress?.('Course generation complete', 100);

    } catch (error) {
      yield { stage: 'error', status: 'error', error: error.message };
      throw error;
    }
  }

  private async executeWithRetry<T>(operation: () => Promise<T>): Promise<T> {
    let lastError: Error;

    for (let attempt = 1; attempt <= this.config.retryAttempts; attempt++) {
      try {
        return await Promise.race([
          operation(),
          new Promise<never>((_, reject) =>
            setTimeout(() => reject(new Error('Operation timeout')), this.config.timeoutMs)
          )
        ]);
      } catch (error) {
        lastError = error;
        
        if (attempt === this.config.retryAttempts) {
          break;
        }

        // Exponential backoff with jitter
        const delay = Math.min(1000 * Math.pow(2, attempt - 1), 10000) + Math.random() * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        
        console.warn(`Claude API attempt ${attempt} failed, retrying in ${delay}ms:`, error.message);
      }
    }

    throw lastError;
  }

  private buildSystemPrompt(type: 'course_outline' | 'lesson_content', difficulty: string): string {
    const basePrompt = `You are an expert educational content creator specializing in comprehensive, engaging course development. Your role is to create high-quality educational materials that are pedagogically sound, technically accurate, and appropriately structured for ${difficulty} level learners.`;

    const typeSpecificPrompts = {
      course_outline: `
        When creating course outlines:
        - Structure content with clear learning objectives
        - Ensure logical progression from basic to advanced concepts
        - Include diverse learning activities (reading, exercises, projects)
        - Provide realistic time estimates for each component
        - Create engaging, descriptive titles that convey value
        - Consider prerequisite knowledge and skill building
        
        Required JSON format:
        {
          "title": "Course Title",
          "description": "Detailed course description",
          "difficulty": "${difficulty}",
          "estimatedHours": number,
          "lessons": [
            {
              "title": "Lesson Title",
              "order": number,
              "estimatedMinutes": number,
              "learningObjectives": ["objective1", "objective2"]
            }
          ]
        }
      `,
      lesson_content: `
        When creating lesson content:
        - Start with clear learning objectives
        - Provide comprehensive explanations with examples
        - Include practical exercises and real-world applications
        - Structure content with logical sections and subsections
        - Ensure appropriate depth for the specified difficulty level
        - Include interactive elements and knowledge checks
        
        Required JSON format:
        {
          "title": "Lesson Title",
          "learningObjectives": ["objective1", "objective2"],
          "sections": [
            {
              "title": "Section Title",
              "type": "TEXT|EXERCISE|QUIZ|VIDEO_SCRIPT",
              "content": "Detailed content",
              "order": number,
              "estimatedMinutes": number
            }
          ],
          "keyTakeaways": ["takeaway1", "takeaway2"]
        }
      `
    };

    return basePrompt + typeSpecificPrompts[type];
  }

  private formatCoursePrompt(prompt: string, difficulty: string): string {
    return `
      Create a comprehensive course outline for: "${prompt}"
      
      Difficulty Level: ${difficulty}
      
      Requirements:
      - 6 well-structured lessons with logical progression
      - Each lesson should have 2 main sections
      - Include realistic time estimates
      - Ensure content depth appropriate for ${difficulty} level
      - Focus on practical, applicable knowledge
      - Include diverse learning activities
      
      Please generate a complete course outline following the specified JSON format.
    `;
  }

  private parseCourseOutline(response: string): CourseOutline {
    try {
      // Extract JSON from response (handle markdown formatting)
      const jsonMatch = response.match(/```(?:json)?\s*([\s\S]*?)\s*```/) || [null, response];
      const jsonString = jsonMatch[1] || response;
      
      const parsed = JSON.parse(jsonString);
      
      // Validate required fields
      if (!parsed.title || !parsed.lessons || !Array.isArray(parsed.lessons)) {
        throw new Error('Invalid course outline structure');
      }

      return parsed as CourseOutline;
    } catch (error) {
      console.error('Failed to parse course outline:', error);
      throw new Error('Invalid JSON response from Claude API');
    }
  }

  private validateLessonStructure(lesson: LessonContent): boolean {
    return (
      lesson.title &&
      lesson.sections &&
      Array.isArray(lesson.sections) &&
      lesson.sections.length >= 2 &&
      lesson.sections.every(section => 
        section.title && section.content && section.type && section.order !== undefined
      )
    );
  }
}

// Type definitions
interface CourseOutline {
  title: string;
  description: string;
  difficulty: string;
  estimatedHours: number;
  lessons: Array<{
    title: string;
    order: number;
    estimatedMinutes: number;
    learningObjectives: string[];
  }>;
}

interface LessonContent {
  title: string;
  learningObjectives: string[];
  sections: Array<{
    title: string;
    type: 'TEXT' | 'EXERCISE' | 'QUIZ' | 'VIDEO_SCRIPT';
    content: string;
    order: number;
    estimatedMinutes: number;
  }>;
  keyTakeaways: string[];
}

interface CourseGenerationUpdate {
  stage: string;
  status: 'in_progress' | 'complete' | 'error';
  data?: any;
  error?: string;
}
```

❌ **Bad Example: Poor AI Service Implementation**
```typescript
// Poor implementation - no error handling, validation, or rate limiting
export class BadClaudeService {
  async generateCourse(prompt: string) {
    const response = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: { 'Authorization': `Bearer ${process.env.ANTHROPIC_API_KEY}` },
      body: JSON.stringify({ prompt })
    });
    
    return response.json(); // No validation or error handling
  }
}
```

### DALL-E 3 Integration for Course Thumbnails
✅ **Good Example: Professional Image Generation Service**
```typescript
// services/dalleService.ts
import OpenAI from 'openai';
import { imageOptimization } from '../utils/imageProcessing';

export class DalleService {
  private client: OpenAI;
  private cache: Map<string, string> = new Map();

  constructor() {
    this.client = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
      timeout: 60000 // 60 second timeout for image generation
    });
  }

  async generateCourseThumbnail(
    courseTitle: string,
    courseDescription: string,
    difficulty: string
  ): Promise<string> {
    const cacheKey = this.generateCacheKey(courseTitle, difficulty);
    
    // Check cache first
    if (this.cache.has(cacheKey)) {
      return this.cache.get(cacheKey)!;
    }

    try {
      const prompt = this.buildImagePrompt(courseTitle, courseDescription, difficulty);
      
      const response = await this.client.images.generate({
        model: 'dall-e-3',
        prompt,
        size: '1024x1024',
        quality: 'hd',
        style: 'natural',
        response_format: 'url',
        n: 1
      });

      if (!response.data || response.data.length === 0) {
        throw new Error('No image generated by DALL-E 3');
      }

      const imageUrl = response.data[0].url!;
      
      // Optimize and process image
      const optimizedUrl = await imageOptimization.processAndStore(
        imageUrl,
        {
          width: 400,
          height: 300,
          quality: 85,
          format: 'webp'
        }
      );

      // Cache the result
      this.cache.set(cacheKey, optimizedUrl);
      
      return optimizedUrl;
    } catch (error) {
      console.error('DALL-E 3 thumbnail generation failed:', error);
      
      // Return fallback image for course category
      return this.getFallbackThumbnail(courseTitle, difficulty);
    }
  }

  private buildImagePrompt(title: string, description: string, difficulty: string): string {
    const difficultyStyles = {
      'BEGINNER': 'friendly, approachable, bright colors',
      'INTERMEDIATE': 'professional, balanced, sophisticated colors',
      'EXPERT': 'technical, advanced, premium design aesthetic'
    };

    const style = difficultyStyles[difficulty] || difficultyStyles['INTERMEDIATE'];

    return `
      Create a professional, modern course thumbnail image for an educational platform.
      
      Course: "${title}"
      Context: ${description.substring(0, 200)}
      
      Style requirements:
      - ${style}
      - Clean, minimalist design suitable for online learning
      - No text overlays (text will be added separately)
      - Professional educational aesthetic
      - High-quality, engaging visual that represents the subject matter
      - Suitable for use as a course card thumbnail
      - Modern flat design or subtle gradients
      - Relevant icons or symbols related to the course topic
    `;
  }

  private generateCacheKey(title: string, difficulty: string): string {
    return `${title.toLowerCase().replace(/\s+/g, '-')}-${difficulty}`;
  }

  private getFallbackThumbnail(title: string, difficulty: string): string {
    // Return category-based fallback thumbnails
    const category = this.categorizeByTitle(title);
    return `/images/fallbacks/${category}-${difficulty.toLowerCase()}.webp`;
  }

  private categorizeByTitle(title: string): string {
    const categories = {
      'programming': ['code', 'programming', 'development', 'software', 'web', 'app'],
      'data-science': ['data', 'machine learning', 'ai', 'analytics', 'statistics'],
      'business': ['business', 'management', 'leadership', 'marketing', 'finance'],
      'design': ['design', 'ui', 'ux', 'graphics', 'visual'],
      'general': []
    };

    const titleLower = title.toLowerCase();
    
    for (const [category, keywords] of Object.entries(categories)) {
      if (keywords.some(keyword => titleLower.includes(keyword))) {
        return category;
      }
    }
    
    return 'general';
  }
}
```

### Server-Sent Events for Real-Time Course Generation
✅ **Good Example: Production SSE Implementation**
```typescript
// routes/courseGeneration.ts
import { Router } from 'express';
import { ClaudeService } from '../services/claudeService';
import { DalleService } from '../services/dalleService';
import { CourseRepository } from '../repositories/courseRepository';

const router = Router();
const claudeService = new ClaudeService();
const dalleService = new DalleService();
const courseRepo = new CourseRepository();

router.post('/generate-stream', async (req, res) => {
  const { title, description, difficulty } = req.body;

  // Validate input
  if (!title || !difficulty) {
    return res.status(400).json({ error: 'Title and difficulty are required' });
  }

  // Set up SSE headers
  res.writeHead(200, {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive',
    'Access-Control-Allow-Origin': '*',
    'Access-Control-Allow-Headers': 'Cache-Control'
  });

  // Keep connection alive
  const keepAlive = setInterval(() => {
    res.write('data: {"type":"ping"}\n\n');
  }, 30000);

  let courseId: string | null = null;

  try {
    // Send initial status
    res.write(`data: ${JSON.stringify({
      type: 'status',
      stage: 'initializing',
      message: 'Starting course generation...',
      progress: 0
    })}\n\n`);

    // Generate course content with streaming updates
    const courseGenerator = claudeService.generateCourseStream(
      title,
      difficulty,
      (stage: string, progress: number) => {
        res.write(`data: ${JSON.stringify({
          type: 'progress',
          stage,
          progress,
          message: `${stage}... ${progress}%`
        })}\n\n`);
      }
    );

    let generatedCourse: any = null;

    for await (const update of courseGenerator) {
      res.write(`data: ${JSON.stringify({
        type: 'update',
        stage: update.stage,
        status: update.status,
        data: update.data
      })}\n\n`);

      if (update.stage === 'complete' && update.status === 'complete') {
        generatedCourse = update.data;
      }

      if (update.status === 'error') {
        throw new Error(update.error);
      }
    }

    // Generate thumbnail in parallel with course saving
    const [savedCourse, thumbnailUrl] = await Promise.all([
      courseRepo.createCourse({
        ...generatedCourse,
        description: description || generatedCourse.description
      }),
      dalleService.generateCourseThumbnail(title, description, difficulty)
    ]);

    courseId = savedCourse.id;

    // Update course with thumbnail
    await courseRepo.updateCourse(courseId, { imageUrl: thumbnailUrl });

    // Send completion message
    res.write(`data: ${JSON.stringify({
      type: 'complete',
      courseId,
      message: 'Course generation completed successfully!'
    })}\n\n`);

  } catch (error) {
    console.error('Course generation failed:', error);
    
    // Send error message
    res.write(`data: ${JSON.stringify({
      type: 'error',
      message: error.message || 'Course generation failed'
    })}\n\n`);
  } finally {
    clearInterval(keepAlive);
    res.end();
  }
});

// Graceful shutdown handling
process.on('SIGTERM', () => {
  // Close all SSE connections gracefully
  console.log('Gracefully shutting down SSE connections...');
});

export { router as courseGenerationRouter };
```

## Key AI Integration Takeaways

### Critical AI Service Patterns ✅

1. **Robust Error Handling**
   - Implement retry mechanisms with exponential backoff
   - Provide fallback responses for service failures
   - Validate AI-generated content before persistence
   - Handle rate limiting and timeout scenarios

2. **Content Quality Assurance**
   - Build comprehensive validation for generated content
   - Implement content moderation and safety checks
   - Ensure educational value and accuracy standards
   - Maintain consistent formatting and structure

3. **Performance Optimization**
   - Use caching for frequently requested content
   - Implement streaming for real-time updates
   - Optimize API usage with request batching
   - Monitor and optimize response times

4. **Production Reliability**
   - Configure proper timeouts and connection pooling
   - Implement comprehensive logging and monitoring
   - Build graceful degradation for service outages
   - Design for horizontal scaling under load

5. **Cost Management**
   - Implement rate limiting and usage tracking
   - Cache expensive operations appropriately
   - Optimize prompts for token efficiency
   - Monitor and alert on usage thresholds

### AI Integration Anti-Patterns to Avoid ❌

1. **No Error Recovery** - Not handling API failures or providing fallback content
2. **Unvalidated Content** - Accepting AI responses without quality validation
3. **Blocking Operations** - Not using streaming for long-running generation tasks
4. **Missing Rate Limits** - Not implementing proper API usage controls
5. **Poor Monitoring** - Insufficient logging and alerting for AI service health

AI service integration is fundamental to LMS success, requiring robust architecture patterns that ensure reliability, quality, and performance under production conditions. 