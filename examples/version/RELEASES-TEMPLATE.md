# Release Notes Template

**Last Updated**: 28/12/2025
**Version**: 1.3.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Overview](#overview)
- [Template](#template)
- [Writing Guidelines](#writing-guidelines)
  - [Language Style](#language-style)
  - [What to Include](#what-to-include)
  - [Section Guidelines](#section-guidelines)
    - [What's New](#whats-new)
    - [Improvements](#improvements)
    - [Bug Fixes](#bug-fixes)
    - [Upcoming Features](#upcoming-features)
- [Examples](#examples)
  - [Example: Feature-Rich Release](#example-feature-rich-release)
  - [Example: Bug Fix Release](#example-bug-fix-release)
  - [Example: Major Release with Breaking Changes](#example-major-release-with-breaking-changes)
  - [Example: Security Update](#example-security-update)
- [Formatting Tips](#formatting-tips)
  - [Use Emoji Sparingly (Optional)](#use-emoji-sparingly-optional)
  - [Screenshots and GIFs](#screenshots-and-gifs)
  - [Version Naming](#version-naming)
- [Best Practices](#best-practices)


---

## Overview

The `RELEASES.md` file is **user-facing release notes** intended for end users, clients, and non-technical stakeholders. It focuses on:

- Features that affect the user experience
- Benefits and improvements users will notice
- Fixed issues that impacted users
- Upcoming features (roadmap hints)

This differs from `VERSION-HISTORY.md` (technical details) and `CHANGELOG.md` (developer summary).

---

## Template

Copy this template to your project root as `RELEASES.md`:

```markdown
# Release Notes

**Last Updated**: DD/MM/YYYY
**Version**: X.Y.Z
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

Welcome to our release notes! Here you'll find information about new features, improvements, and fixes in each version of the application.

---

## Table of Contents

- [Latest Release](#latest-release)
- [Previous Releases](#previous-releases)
- [Upcoming Features](#upcoming-features)

---

## Latest Release

### Version X.Y.Z - DD Month YYYY

#### What's New

**Feature Name**
Description of what users can now do and how it benefits them.

**Another Feature**
Description focusing on user value.

#### Improvements

- Improvement 1 in user-friendly language
- Improvement 2 focusing on user benefit

#### Bug Fixes

- Fixed an issue where [user-visible problem description]
- Resolved a problem that caused [user-visible symptom]

---

## Previous Releases

### Version X.Y.Z - DD Month YYYY

#### Highlights
- Feature highlight 1
- Feature highlight 2

---

## Upcoming Features

We're working on exciting new features:

- **Feature Name**: Brief description
- **Another Feature**: Brief description

*Features and timelines are subject to change.*
```

---

## Writing Guidelines

### Language Style

| Do                                        | Don't                           |
| ----------------------------------------- | ------------------------------- |
| Use simple, friendly language             | Use technical jargon            |
| Focus on user benefits                    | Describe implementation details |
| Write in second person ("You can now...") | Write in passive voice          |
| Use British English spelling              | Use American spelling           |
| Be concise and scannable                  | Write long paragraphs           |

### What to Include

| Include                        | Exclude                          |
| ------------------------------ | -------------------------------- |
| Features users interact with   | Internal refactoring             |
| Visible improvements           | Code structure changes           |
| Fixed issues users reported    | Technical debt fixes             |
| Performance gains users notice | Developer tooling changes        |
| Security fixes (high level)    | CVE numbers or technical details |

### Section Guidelines

#### What's New
- Lead with the most exciting feature
- Use descriptive headings for each feature
- Explain what users can do, not how it works
- Include screenshots if helpful

#### Improvements
- Focus on noticeable changes
- Quantify where possible ("40% faster")
- Group related improvements

#### Bug Fixes
- Describe the user-visible symptom
- Don't include file names or code references
- Phrase positively ("Fixed" not "Bug in...")

#### Upcoming Features
- Build excitement without over-promising
- Use "We're working on..." language
- Include disclaimer about changes

---

## Examples

### Example: Feature-Rich Release

```markdown
## Latest Release

### Version 2.1.0 - 15 January 2025

#### What's New

**Dark Mode**
You asked, we delivered! Dark mode is now available across the entire application. Switch between light and dark themes from your account settings, or let the app follow your device preferences automatically.

**Team Collaboration**
Invite your colleagues to collaborate on projects in real-time. You can now:
- Share dashboards with team members
- Set permissions (view, edit, or admin)
- See who's viewing a report in real-time
- Leave comments and annotations on charts

**Export to PDF**
Generate professional PDF reports with a single click. Your exported reports include:
- All charts and visualisations
- Company branding and logo
- Table of contents for easy navigation
- Print-optimised formatting

#### Improvements

- **Faster Loading**: Dashboards now load up to 50% faster, especially on mobile devices
- **Better Mobile Experience**: We've redesigned the mobile interface for easier navigation on smaller screens
- **Smarter Search**: Search now finds results as you type and suggests related items

#### Bug Fixes

- Fixed an issue where some charts weren't displaying correctly in Safari
- Resolved a problem that caused the app to log you out unexpectedly on mobile
- Fixed date filtering which was sometimes showing results from the wrong day

---
```

### Example: Bug Fix Release

```markdown
## Latest Release

### Version 2.0.1 - 5 January 2025

#### Improvements

- Faster response times when loading large reports

#### Bug Fixes

- Fixed an issue where scheduled reports weren't being sent at the correct time for users in certain time zones
- Resolved a problem that prevented some users from uploading files larger than 5MB
- Fixed the "Remember Me" option which wasn't working correctly on the login page
- Resolved an issue where chart colours would reset after refreshing the page

---
```

### Example: Major Release with Breaking Changes

```markdown
## Latest Release

### Version 3.0.0 - 1 February 2025

#### Important Update

This is a major update that includes significant improvements to how the app works. Please read the changes carefully.

**What You Need to Know**
- You'll need to log in again after this update
- Some settings may need to be reconfigured
- We recommend reviewing your notification preferences

#### What's New

**Completely Redesigned Dashboard**
We've rebuilt the dashboard from the ground up to make it more intuitive and powerful. The new design features:
- Drag-and-drop widget arrangement
- Customisable layouts for different screen sizes
- Quick actions menu for common tasks
- Improved data visualisation with interactive charts

**Enhanced Security**
We've upgraded our security to keep your data even safer:
- Automatic session timeout after inactivity
- Option to enable two-factor authentication
- Detailed login history in your account settings
- Improved password requirements for new accounts

**Powerful New Reporting**
Create more detailed reports with our new reporting engine:
- Combine data from multiple sources
- Schedule reports to run automatically
- Share reports with external stakeholders via secure links
- Export in multiple formats (PDF, Excel, CSV)

#### Improvements

- The entire app is now 40% faster
- Reduced mobile data usage by optimising how information is loaded
- Improved accessibility for screen readers and keyboard navigation

#### Bug Fixes

- Fixed all known issues from previous versions

#### Coming Soon

We're already working on the next set of features:
- **Mobile App**: Native iOS and Android apps for on-the-go access
- **API Access**: For developers who want to integrate with other tools
- **Advanced Analytics**: AI-powered insights and predictions

---
```

### Example: Security Update

```markdown
## Latest Release

### Version 2.0.2 - 10 January 2025

#### Security Update

This release includes important security improvements. We recommend all users update as soon as possible.

**What We've Done**
- Strengthened login protection against automated attacks
- Improved how we handle your session to prevent unauthorised access
- Added additional verification for sensitive account changes

**What You Should Do**
- No action required, but we recommend:
  - Reviewing your recent login activity in Account Settings
  - Ensuring your password is strong and unique
  - Considering enabling two-factor authentication if available

#### Bug Fixes

- Fixed an issue where password reset emails were sometimes delayed

---
```

---

## Formatting Tips

### Use Emoji Sparingly (Optional)
Some teams prefer emoji for visual scanning:
- 🆕 New features
- ⚡ Improvements
- 🐛 Bug fixes
- 🔒 Security updates

### Screenshots and GIFs
For major features, consider adding:
- Before/after comparisons
- Short animated GIFs showing the feature
- Annotated screenshots highlighting key areas

### Version Naming
Consider user-friendly version names for major releases:
- Version 3.0 "Phoenix"
- Version 4.0 "Aurora"

---

## Best Practices

1. **Write for your users**, not developers
2. **Lead with benefits**, not features
3. **Be honest** about bugs and issues
4. **Keep it scannable** with headers and bullets
5. **Update regularly** to build trust
6. **Use DD Month YYYY** date format (e.g., 15 January 2025)
7. **British English** spelling throughout
