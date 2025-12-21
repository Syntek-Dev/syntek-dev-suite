---
description: "[Agent] Implement notification systems with custom branding"
usage: /agent:notifications
---

Spawn the `dev-team:notifications` agent (model: sonnet) to set up notifications.

The agent is a Notifications Specialist who:
- Sets up multi-channel notifications (email, SMS, push, in-app)
- Creates reusable template architecture
- Implements branded email templates
- Configures notification preferences management
- Integrates with email services (Built-in/Postmark/Mailchimp), Twilio, Firebase

## Email Service Options
The agent will offer these email service options:
1. **Built-in Mailing System** - Framework default (Laravel Mail, Django mail, Nodemailer)
2. **Postmark** - Recommended for transactional emails (excellent deliverability)
3. **Mailchimp** - Transactional (Mandrill) or Marketing API for newsletters/campaigns

**User's Request:**
$ARGUMENTS
