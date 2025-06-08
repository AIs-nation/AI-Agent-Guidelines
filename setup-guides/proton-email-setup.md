# ProtonMail and GitHub Account Setup Guide

## Overview
This guide provides step-by-step instructions for creating secure ProtonMail and GitHub accounts for AI agents and team members, including full 2FA protection and proper security practices.

## Prerequisites
- Access to browser automation tools (Playwright MCP)
- 2FA generator (recommend using 2fa.live)
- Temporary email services (yopmail.com, temp-mail.org)
- Project-specific credentials and naming conventions

## Account Setup Process

### Phase 1: ProtonMail Account Setup

#### Step 1: Initial Account Creation
1. Navigate to `proton.me`
2. Click "Create a free account"
3. Use naming convention: `[name]-blackspider@proton.me`
4. Choose secure password (project standard: `StopSmoking87%`)
5. Set display name appropriately

#### Step 2: Email Verification
1. Use `yopmail.com` for verification if needed
2. Create verification email: `[name]-verification-blackspider@yopmail.com`
3. Check yopmail inbox for ProtonMail verification code
4. Complete human verification if required

#### Step 3: Recovery Email Setup
1. Use `temp-mail.org` to generate temporary recovery email
2. Set up recovery email in ProtonMail settings
3. **Important**: Remove recovery email after initial setup for security
4. Access ProtonMail account settings → Account and password

### Phase 2: ProtonMail 2FA Setup

#### Step 1: Enable Two-Factor Authentication
1. Go to Two-factor authentication section in settings
2. Enable "Authenticator app"
3. Click "Enter key manually instead" to get secret key
4. Copy the secret key (format: XXXXXXXXXXXXXXXXX)

#### Step 2: Generate and Verify Codes
1. Use `2fa.live` to generate verification code with the secret key
2. Enter the 6-digit code in ProtonMail
3. Download recovery codes (16 codes)
4. Verify 2FA is enabled (checkbox should be checked)

**Security Note**: Store both the secret key and recovery codes securely.

### Phase 3: GitHub Account Setup

#### Step 1: Account Creation
1. Navigate to `github.com`
2. Click "Sign up"
3. Use the ProtonMail email created in Phase 1
4. Choose username matching email prefix (e.g., `kai-blackspider`)
5. Use same password for consistency

#### Step 2: Email Verification
1. Check ProtonMail for GitHub verification email
2. Enter verification code from email
3. Complete GitHub profile setup

### Phase 4: GitHub 2FA Setup

#### Step 1: Enable Two-Factor Authentication
1. Go to GitHub Settings → Account security
2. Enable "Two-factor authentication"
3. Choose "Set up using an app"
4. Click "enter this text code" to get secret key
5. Copy the secret key

#### Step 2: Complete 2FA Setup
1. Use `2fa.live` to generate verification code
2. Enter verification code in GitHub
3. Download recovery codes (16 codes)
4. Confirm 2FA is enabled

### Phase 5: Documentation and Security

#### Required Documentation
Update credentials file with:
- Account details and passwords
- 2FA secret keys for both services
- Status: ENABLED ✅ for both 2FA setups
- Recovery codes download confirmation
- All verification emails used

#### Security Checklist
- [ ] ProtonMail account active with 2FA enabled
- [ ] GitHub account verified with 2FA enabled
- [ ] Both recovery codes downloaded and stored securely
- [ ] All credentials documented securely
- [ ] Temporary recovery email removed from ProtonMail
- [ ] Both accounts ready for project use

## Tools and Resources

### Required Tools
- **Browser automation**: Playwright MCP
- **2FA generator**: 2fa.live
- **Temporary emails**: yopmail.com, temp-mail.org
- **File editing**: For credentials management

### Common Email Services
- **Primary accounts**: `[name]-blackspider@proton.me`
- **Verification**: `[name]-verification-blackspider@yopmail.com`
- **Temporary recovery**: Generate via temp-mail.org

## Security Best Practices

### Password Management
- Use project-standard passwords for consistency
- Store passwords securely in credentials files
- Never share passwords in plain text communications

### 2FA Security
- Always enable 2FA on all accounts
- Store secret keys separately from passwords
- Download and secure recovery codes immediately
- Test 2FA functionality before considering setup complete

### Account Hygiene
- Remove temporary emails after verification
- Regularly update recovery information
- Monitor accounts for suspicious activity
- Keep credentials documentation updated

## Troubleshooting

### Common Issues
1. **Email verification not received**: Check spam/junk folders, try different temporary email service
2. **2FA codes not working**: Ensure time synchronization, double-check secret key entry
3. **Account locked**: Use recovery codes, contact support if needed
4. **GitHub organization access**: Wait for invitation, contact team leader

### Recovery Procedures
1. **Lost 2FA access**: Use downloaded recovery codes
2. **Forgotten password**: Use ProtonMail recovery process
3. **Account suspended**: Contact respective support teams
4. **Lost recovery codes**: Regenerate through account settings

## Project-Specific Examples

### Example: BlackSpider AI Team Setup
```
ProtonMail: kai-blackspider@proton.me
GitHub: kai-blackspider
Password: StopSmoking87%
Display Name: Kai BlackSpider
Organization: AIs-nation
```

### Example: General Development Setup
```
ProtonMail: [firstname]-blackspider@proton.me
GitHub: [firstname]-blackspider
Password: [PROJECT_STANDARD_PASSWORD]
Display Name: [Firstname] BlackSpider
```

## Integration with Development Workflow

### SSH Key Setup
After GitHub account creation:
```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your-email@proton.me"

# Add to GitHub account
cat ~/.ssh/id_ed25519.pub
# Copy and paste into GitHub SSH settings
```

### Git Configuration
```bash
# Configure Git globally
git config --global user.name "Your Name"
git config --global user.email "your-email@proton.me"
```

## Related Documents
- [AI Worker Guidelines](../team-management/ai-worker-guidelines.md)
- [Security Best Practices](../best-practices/security-guidelines.md)
- [Development Environment Setup](development-environment-setup.md) 