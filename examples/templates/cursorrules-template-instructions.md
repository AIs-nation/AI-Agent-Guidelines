# CursorRules Template Instructions

## Overview
This template provides a comprehensive configuration structure for AI agents using the `.cursorrules` format. It includes all essential components for professional AI agent behavior, communication standards, and project-specific requirements.

## How to Use This Template

### 1. Copy the Template
Copy `cursorrules-template.json` to your project root and rename it to `.cursorrules`.

### 2. Replace Placeholders
Replace all bracketed placeholders `[PLACEHOLDER_NAME]` with your specific values:

#### Role Configuration
- `[AI_AGENT_NAME]` - The name/identity of your AI agent (e.g., "Kai", "Nova", "Alex")
- `[ROLE_TITLE]` - The agent's role (e.g., "Full Stack Engineer", "Team Leader", "QA Engineer")

#### Automation Settings
- `[AUTOMATION_COMMAND]` - Any required automation scripts or commands
- `[FREQUENCY]` - How often to run automation (e.g., "end of every prompt", "daily")
- `[EXCEPTION_RULES]` - Any exceptions to the automation rules
- `[CRITICAL_REQUIREMENTS]` - Critical automation requirements

#### Project Context
- `[PROJECT_NAME]` - Your project name (e.g., "AI Learning Management System")
- `[PROJECT_STAGE]` - Current stage (e.g., "Stage 1", "MVP", "Beta")
- `[FRONTEND_TECHNOLOGY]` - Frontend tech stack (e.g., "Next.js 14", "React", "Vue.js")
- `[BACKEND_TECHNOLOGY]` - Backend tech stack (e.g., "Express.js", "Django", "FastAPI")
- `[DATABASE_TECHNOLOGY]` - Database solution (e.g., "SQLite via Prisma ORM", "PostgreSQL", "MongoDB")

#### Services and Integrations
- `[AI_SERVICE_1]`, `[AI_SERVICE_2]` - AI services used (e.g., "Anthropic Claude", "OpenAI")
- `[API_SERVICE]` - Additional API services (e.g., "Unsplash API", "Stripe API")

#### Development Sessions
- `[PROJECT_FRONTEND]` - Frontend session name (e.g., "lms_frontend", "app_frontend")
- `[PROJECT_BACKEND]` - Backend session name (e.g., "lms_backend", "api_backend") 
- `[FRONTEND_DESCRIPTION]` - Description of frontend session purpose
- `[BACKEND_DESCRIPTION]` - Description of backend session purpose

#### Features and Requirements
- `[STAGE_NAME]` - Current stage name
- `[STAGE_DESCRIPTION]` - Description of current stage requirements
- `[FEATURE_1]`, `[FEATURE_2]` - Major features being worked on
- `[FEATURE_DESCRIPTION]` - Description of feature requirements
- `[STEP_1]`, `[STEP_2]`, `[STEP_3]` - Process steps for features
- `[DATA_STRUCTURE]` - Data structure organization
- `[TYPE_1]`, `[TYPE_2]`, `[TYPE_3]` - Content or component types
- `[REQUIREMENTS_DESCRIPTION]` - Detailed requirements
- `[CURRENT_PRIORITY]` - Current development focus

#### Issues and Priorities
- `[ISSUE_CATEGORY]` - Category of critical issues
- `[ISSUE_DESCRIPTION]` - Description of critical issues
- `[PRIORITY_DESCRIPTION]` - Priority resolution approach

#### UI Requirements
- `[COMPONENT_1]`, `[COMPONENT_2]`, `[COMPONENT_3]` - UI components
- `[STYLE_REQUIREMENT_1]`, `[STYLE_REQUIREMENT_2]`, `[STYLE_REQUIREMENT_3]` - Style requirements

#### Communication Settings
- `[PRIORITY_CHANNEL_1]`, `[PRIORITY_CHANNEL_2]` - High priority communication channels
- `[UPDATE_FREQUENCY]` - How often to provide updates
- `[UPDATE_FORMAT]` - Format for updates (e.g., "short bullet points")
- `[ESCALATION_RULES]` - When and how to escalate issues

#### Documentation
- `[PROJECT_SPECIFIC_EXAMPLE]` - Example of project-specific documentation needs

### 3. Customize Sections

#### Remove Unused Sections
If certain sections don't apply to your project, you can remove them:
- `automation` - If no automation scripts are needed
- `communication_requirements` - If using default communication
- `ui_requirements` - For backend-only projects

#### Add Custom Sections
You can add project-specific sections as needed:
```json
"custom_requirements": {
  "security": "specific security requirements",
  "performance": "performance benchmarks",
  "compliance": "regulatory compliance needs"
}
```

### 4. Validation
After customization:
1. Validate JSON syntax using a JSON validator
2. Ensure all placeholders have been replaced
3. Test with your AI agent to verify behavior

## Best Practices

### Keep It Updated
- Update project context as development progresses
- Adjust priorities based on current needs
- Add new requirements as they emerge

### Be Specific
- Use concrete, measurable requirements
- Provide specific examples where helpful
- Include exact technology versions when relevant

### Maintain Consistency
- Use consistent naming conventions
- Keep similar projects aligned in structure
- Share common patterns across team

## Example Usage

For a simple web application:
```json
{
  "role": {
    "identity": "Alex - Full Stack Developer",
    "communication": {
      "greeting_options": [
        "This is Alex, I understand your goal",
        "This is Alex, I need clarification on your goal"
      ]
    }
  },
  "project_context": {
    "current_project": "E-commerce Platform - MVP",
    "technology_stack": {
      "frontend": "React 18",
      "backend": "Node.js with Express",
      "database": "PostgreSQL"
    }
  }
}
```

## Related Files
- [AI Team Leader Guidelines](../team-management/ai-team-leader-guidelines.md)
- [AI Worker Guidelines](../team-management/ai-worker-guidelines.md)
- [Communication Protocols](communication-protocols-template.md) 