# Release Notes

**Last Updated**: 09/01/2026
**Version**: 1.4.0
**Maintained By**: Development Team
**Language**: British English (en_GB)
**Timezone**: Europe/London

---

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Latest Release](#latest-release)
  - [Version 1.4.0 - 09 January 2026](#version-140---09-january-2026)
- [Previous Releases](#previous-releases)
  - [Version 1.3.1 - 29 December 2025](#version-131---29-december-2025)
  - [Version 1.3.0 - 28 December 2025](#version-130---28-december-2025)
  - [Version 1.2.0 - 24 December 2025](#version-120---24-december-2025)
  - [Version 1.1.0 - 24 December 2025](#version-110---24-december-2025)
  - [Version 1.0.0 - 21 December 2025](#version-100---21-december-2025)
- [Need Help?](#need-help)


---

## Latest Release

### Version 1.4.0 - 09 January 2026

#### What's New

**Portable Plugin Tools**

Your projects are now fully self-contained! When you initialise a new project with `/init`, all the Python plugin tools are automatically copied into your project's `.claude/plugins/` folder.

This means:

- **Work anywhere** - Your project works even without the Syntek Dev Suite installed globally
- **Share with your team** - Everyone gets the same plugin versions, no setup required
- **Customise freely** - Modify plugins for project-specific needs without affecting other projects
- **Version stability** - Your project keeps working even if the main plugin updates

#### How It Works

When you run `/init`, the system now:

1. Creates the `.claude/plugins/` directory in your project
2. Copies all 14 Python plugin tools
3. Makes them executable automatically

Your project's `CLAUDE.md` will include full documentation on how to use these plugins.

#### For Existing Projects

Want to add plugin tools to an existing project? Simply run:

```bash
mkdir -p .claude/plugins/
cp /path/to/syntek-dev-suite/plugins/*.py .claude/plugins/
chmod +x .claude/plugins/*.py
```

---

## Previous Releases

### Version 1.3.1 - 29 December 2025

#### What's Fixed

**Works Anywhere Now**

This bug fix release makes the plugin work properly no matter where you install it. All hardcoded paths have been replaced with dynamic detection.

- **Installation flexibility** - Install the plugin anywhere on your system
- **Cross-platform compatibility** - Better support for Linux and macOS
- **Portable configuration** - All file paths are now relative

---

### Version 1.3.0 - 28 December 2025

#### What's New

**Project Management Integration**

Connect your Claude Dev Team to your favourite project management tool. Whether you use ClickUp, Linear, Jira, GitHub Projects, or any of the other 10 supported platforms, you can now:

- Automatically sync your user stories and tasks with your PM tool
- Create and update issues directly from Claude Code
- Track sprint progress across your entire team
- Link commits to specific issues for better traceability
- Customise field mappings to match your workflow

Simply run `/syntek-dev-suite:pm-setup` to get started, and the system will detect your PM tool automatically.

**Improved Navigation**

We've refreshed all the documentation README files throughout the project to make it easier to:
- Find the right agent for your task
- Understand what each plugin does
- Navigate the codebase structure
- Discover examples and templates

#### Coming Soon

In our next release, we're working on:
- Enhanced reporting capabilities for sprint metrics
- Dashboard visualisations for agent performance
- Integration with more communication tools (Slack, Teams, Discord)

---

## Previous Releases

### Version 1.2.0 - 24 December 2025

#### Highlights

**Smart Version Management**

We've introduced a powerful Version Agent that handles all your versioning needs:

- **Automatic version bumping** - Just tell it what changed (feature, fix, or breaking change) and it updates all version files
- **Consistent documentation** - Maintains technical changelogs, developer summaries, and user-facing release notes
- **Metadata tracking** - Every documentation file now includes version info, last updated date, and language settings

Run `/version bump minor` to increment your version and update all documentation in one go.

**Enhanced Markdown Editing**

Documentation just got better with:
- Automatic table of contents generation
- Smart table formatting
- Consistent headers across all files
- British English spell checking
- VS Code extension integration for a smooth editing experience

Your documentation stays professional and consistent without extra effort.

---

### Version 1.1.0 - 24 December 2025

#### Highlights

**Browser Testing Made Easy**

Testing web applications is now seamless with integrated Chrome support:

- **One-click browser launch** - The system auto-detects Chrome on your machine
- **Live debugging** - See console errors and inspect the DOM directly from Claude Code
- **Automated testing** - Works with Playwright, Cypress, Puppeteer, and more
- **Visual verification** - Compare your running app against design mockups

Perfect for catching bugs early and ensuring your app looks great across all browsers.

---

### Version 1.0.0 - 21 December 2025

#### Highlights

**Complete Development Suite**

The first major release brings everything you need for full-stack development:

- **30 Specialised Agents** - From planning to deployment, there's an agent for every task
- **Self-Learning System** - The agents get smarter over time based on your feedback
- **Multi-Stack Support** - Laravel, Django, React, React Native, and more
- **80+ Code Examples** - Real-world examples for common development patterns

**Smart Environment Detection**

The suite automatically detects:
- Your programming language and framework
- Database type and ORM
- Docker and DDEV configurations
- Git repository settings
- Environment variables

No configuration files to write - it just works.

**Team Collaboration**

Built for teams with:
- Shared learning across the entire team
- Consistent British English documentation
- Standardised code patterns
- Section README files for easy onboarding

Get new team members productive on day one.

#### What Users Are Saying

"The self-learning system is brilliant - the agents actually improve the more we use them!"

"Finally, an AI dev tool that understands British English properly. Colour, not color!"

"The Section README files saved us hours during onboarding. New developers know exactly where to find things."

---

## Need Help?

Visit our documentation at `/docs` or run `/help` for a list of all available commands.

For issues and feature requests, please visit our [GitHub repository](https://github.com/Syntek-Studio/syntek-dev-suite).
