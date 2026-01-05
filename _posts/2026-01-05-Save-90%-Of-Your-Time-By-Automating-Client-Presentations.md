---
layout: post
title: Save 90% Of Your Time By Automating Recurring Client Presentations
author: Rik Groenewoud
tags: microsoft powershell slidev automation
---

## Intro

If you've ever worked in as a Service Manager or similar role, you know the drill. Every two weeks, there's another operational meeting. Every month, another steering committee meeting. And before each meeting, someone (usually you) spends hours:

- Manually pulling data from Azure DevOps
- Copying work item statuses into a slide deck
- Calculating metrics and resolve times
- Formatting slides to match brand guidelines
- Proofreading for typos
- Etc etc.

Multiply this by let's say 7 clients with both monthly and two-weekly meetings, and you're spending over 20 hours every month on repetitive presentation work. There has to be a better way.

## The Solution

I built a complete automation system that transforms meeting preparation from a multi-hour manual process into a one-click operation. Here's what it does:

### Automated Data Collection

Instead of manually querying Azure DevOps and calculating metrics, PowerShell scripts automatically fetch:

- Active work items and their current status
- 3-month trend data for incidents, changes, and feature requests
- Average resolve times (excluding weekends for accuracy)
- Team capacity and velocity metrics

### Dynamic Presentation Generation

A Node.js script reads configuration files and generates presentation content using the [Slidev](https://sli.dev/) framework. This means:

- Templates that maintain consistent structure and branding
- Dynamic content generated from real data, not manually typed
- Professional slides with proper formatting, icons, and trend indicators
- Version control for all presentations (they're just markdown!)

### Azure Pipeline Orchestration

Everything runs automatically via Azure Pipelines with a simple UI:

1. Select your customer from a dropdown
2. Enter comma-separated attendee names
3. Add any team updates or opportunities
4. Click "Run"

Within minutes, you get:

- A professionally formatted PDF presentation
- Source markdown files for any manual tweaks
- All artifacts stored and version-controlled

## How it works

The system supports two distinct meeting types:

### Two-Weekly Operational Meetings

Focus: Day-to-day execution and tactical updates

Automated Content:

- Current sprint work items
- Blockers and dependencies
- Recent achievements
- Upcoming work for next 2 weeks
- Risks and mitigations
- Action items with owners

Pipeline Parameters: Customer name, attendees, and AOB items

### Monthly Steering Committee Meetings

Focus: Strategic overview and trend analysis

Automated Content:

- Team updates from the month
- Strategic opportunities
- Monthly metrics (incidents, changes, feature requests)
- 3-month trend analysis with visual charts
- Resolve time averages with trend indicators
- Executive-level insights

Pipeline Parameters: Customer name, attendees, team updates, and opportunities

## Technical Details

### PowerShell Data Fetchers

- Query Azure DevOps Work Item Tracking API
- Calculate business-day resolve times
- Generate JSON data files with structured metrics

### Node.js Presentation Generators

- Read JSON configuration and data files
- Apply business logic for status icons and trend indicators
- Generate Slidev-compatible markdown

### Slidev Templates

- Define slide structure and layouts using markdown
- Include placeholders for dynamic content (e.g., `{{CUSTOMER_NAME}}`, `{{WORK_ITEMS}}`)
- Reference custom theme for branding
- Use Slidev's built-in layouts (cover, two-column, image-left, etc.)

### Custom Slidev Theme

- Brand-specific layouts (cover, two-column, image-left/right, etc.)
- Consistent color schemes and typography
- Reusable Vue components for complex slides

### Azure Pipelines

- Orchestrate the entire workflow
- Manage parameters and secure credentials
- Generate PDF exports
- Publish artifacts for download

### Data Flow

```
[Azure DevOps]
    ↓
[PowerShell Scripts] → JSON Data Files
    ↓
[Node.js Generator] + [Config Files] + [Template]
    ↓
[Generated Markdown]
    ↓
[Slidev] → PDF Export
    ↓
[Azure Pipeline Artifacts]
```

## Benefits

### Time Savings

Before: 1 hour per presentation × 3 presentations per month × 7 clients = 21 hours/month
After: 5 minutes per presentation × same volume = ~2 hours/month

That's 90%+ time savings (19 hours per month) that can be reinvested in actual client work.

### Consistency and Quality

The automation ensures consistency across all presentations. Every slide follows the same professional structure, metrics are always calculated the same way, and brand guidelines are enforced automatically. No more typos or formatting inconsistencies.

### Real-Time Data

Since presentations are generated from live data, they always reflect the latest work item states. There's no risk of showing outdated information - all metrics are calculated directly from Azure DevOps data at the time of generation.

### Version Control and Auditability

All presentation source files are stored in Git, providing a complete historical record of what was presented when. This makes it easy to track changes and improvements over time, and ensures reproducible builds.

### Scalability

Adding new clients requires minimal configuration since templates can be reused across any number of customers. The pipeline infrastructure supports unlimited parallel runs, making it easy to scale as your client base grows.

### Developer-Friendly

Everything is code - Infrastructure as Code for presentations! This makes it easy to customize and extend. A local development workflow is available for quick iterations, and the system has full testing capabilities.

## Lessons learned

### Start with Templates

We began by manually creating the "perfect" presentation for one client, then worked backward to templatize it. This is much easier than trying to design templates in the abstract.

### Configuration Over Code

By keeping customer-specific details in JSON config files rather than hardcoded, we made the system vastly more maintainable. Adding a new customer is now a 5-minute configuration task, not a code change.

### PowerShell for Azure DevOps

While Node.js is great for presentation generation, PowerShell excels at Azure DevOps integration. The `Invoke-RestMethod` cmdlet with PAT token authentication made data fetching straightforward.

### Slidev is a Hidden Gem

We evaluated several presentation tools (Google Slides API, PowerPoint automation, LaTeX Beamer) but Slidev offered the best balance. It provides a developer-friendly workflow using markdown and Vue, produces beautiful output with minimal effort, and has excellent theming and customization options. The PDF export quality is also outstanding.

### Cache Aggressively in Pipelines

The first version of our pipeline took 4-5 minutes. By adding npm caching and optimizing dependency installation, we got it down to under 2 minutes.

## Conclusion

Automating recurring presentations might seem like a small optimization, but the impact compounds quickly. When you're managing 21 client meetings per month (7 customers × 3 meetings each), every minute of preparation time matters.

More importantly, automation frees yourself (or your team) to focus on high-value work: solving complex problems, building relationships, and delivering business impact. Nobody became a consultant to manually copy data between systems.

By treating presentations as code, we brought software engineering best practices (version control, testing, CI/CD) to a traditionally manual process. The result is presentations that are faster to create, more consistent, more accurate, and infinitely more scalable.

If you're spending hours each week on repetitive presentation work, ask yourself: could this be automated? The answer is almost always yes. And the ROI is better than you think.

**Want to know more see some of the actual code that makes this happen? Feel free to contact me!**

## Further reading on Slidev

- [Slidev Documentation](https://sli.dev/)
- [Slidev Guide - Getting Started](https://sli.dev/guide/)
- [Slidev Themes](https://sli.dev/themes/gallery.html)
- [Slidev Exporting](https://sli.dev/guide/exporting.html)
- [Slidev Custom Layouts](https://sli.dev/custom/)

<script src="https://giscus.app/client.js"
        data-repo="RikGr/cloudwoud"
        data-repo-id="R_kgDOHLlC9w"
        data-category="Announcements"
        data-category-id="DIC_kwDOHLlC984CO_2O"
        data-mapping="pathname"
        data-reactions-enabled="0"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="light"
        data-lang="en"
        crossorigin="anonymous"
        async>
</script>
