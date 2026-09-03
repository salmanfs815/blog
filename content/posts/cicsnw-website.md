---
title: "New West Masjid Website"
date: 2026-09-01
tags:
  - projects
  - web development
  - react
  - vite
  - typescript
---

The Canadian Islamic Cultural Society of New Westminster (CICSNW) is working to establish the first permanent masjid and Islamic centre in New Westminster, BC. The organization needed a website that could communicate the vision of the New West Masjid project, make fundraising information easy to understand, and provide the community with an accessible place to find prayer times, contact information, updates, and donation links.

This project became an opportunity to build something practical for a real community while focusing on the parts of web development that are easy to overlook: clarity, maintainability, accessibility, and deployment simplicity.

## The use case

The site had a few important jobs:

- Explain the long-term New West Masjid project and its immediate land-acquisition goals.
- Make donations easy to find and act on.
- Show fundraising progress and upcoming payment milestones transparently.
- Provide practical information for the current Taiba Musalla community, including prayer times, classes, contact details, and social links.
- Present project photos and frequently asked questions without requiring a developer to edit the application code for every content update.
- Work well on mobile devices, where many visitors would likely access the site.

The key design problem was not simply “make a website.” It was to make a fundraising page feel trustworthy and understandable. Visitors should quickly be able to answer three questions:

1. What is being built?
2. Why does it matter?
3. How can I help?

That guided the structure of the page. The fundraising call to action and high-level goal are immediately visible, followed by project context, payment milestones, sponsorship options, prayer times, gallery content, FAQs, and contact information.

## Ideation: designing around clarity and trust

For a community fundraising site, an attractive design alone is not enough. Donation pages need to earn trust by communicating concrete information clearly.

The initial concept was a single-page experience with section-based navigation. This approach fit the project well because the content is focused, visitors have a clear path through the page, and the organization does not need to maintain a complex multi-page content structure.

I organized the page around a few principles:

- Lead with the project’s purpose rather than implementation details.
- Put the donation call to action near the fundraising goal.
- Show the financial target and payment schedule in a digestible format.
- Use real project imagery and location information to make the initiative tangible.
- Keep the page navigable with both desktop and mobile menus.
- Avoid unnecessary dependencies or infrastructure.

The visual direction uses warm, understated colours and spacious layouts to make the fundraising information feel calm and legible rather than overly promotional. The content is intentionally direct: a visitor should not need to read several paragraphs before understanding the project.

## Tech stack

The website is built with:

- **React 19** for component-based UI development.
- **TypeScript** for safer application code and clearer data contracts.
- **Vite** for local development and optimized static production builds.
- **Plain CSS** for styling and responsive layouts.
- **react-slick** for the accessible image gallery carousel.
- **JSON content files** for FAQs and gallery entries.
- **Static hosting** for a low-maintenance deployment model.

I chose React because it provides a clean way to structure interactive UI elements such as the mobile menu, expandable FAQ section, scroll-to-top control, and responsive gallery. Although the site is primarily informational, these small interactions benefit from component state and predictable rendering.

TypeScript was useful because configuration drives several visible pieces of the interface, including financial totals, milestones, contact links, and external embeds. Strong typing helps catch mistakes before deployment, especially when values are supplied through environment variables.

Vite was a natural fit for this kind of project. It keeps local development fast, produces a static deployment bundle, and avoids introducing server infrastructure that the site does not need.

## Architecture: static by design

One of the most important architectural decisions was choosing a static-site model.

The website does not need user accounts, a database, payment processing logic, or a custom backend. Donations are handled by an external fundraising platform, while the site provides the project context and sends visitors to the donation destination.

The architecture is intentionally simple:

```text
Browser
  |
  v
Static HTML, CSS, JavaScript, and images
  |
  +-- React page components
  +-- Environment-based public configuration
  +-- JSON-managed FAQ and gallery content
  +-- External donation, map, social, and prayer-time services
```

This model has several benefits:

- Lower hosting and maintenance costs.
- Fewer security concerns than a custom server-side application.
- Easier deployment to platforms such as Cloudflare Pages, Netlify, Vercel, or GitHub Pages.
- Fast loading because the production build consists of static assets.
- A smaller operational burden for a community organization.

The main application component is responsible for composing the page sections, while `siteConfig.ts` centralizes configuration that may change over time. Gallery and FAQ content live in separate JSON files, so straightforward edits do not require modifying the React component itself.

## Configuration without a backend

Fundraising totals, milestone dates, contact information, sponsorship amounts, map links, social links, and prayer-time URLs are all read from `VITE_` environment variables at build time.

This creates a useful separation between the UI and organization-specific values. For example, changing the fundraising goal should not require searching through JSX markup or updating multiple hard-coded values.

The configuration layer also validates required values and numeric inputs early. If a critical value is missing or malformed, the build fails instead of silently deploying a broken donation page.

There is an important tradeoff here: Vite environment variables prefixed with `VITE_` are included in the client-side bundle. That means they are appropriate for public information such as a donation URL or contact email, but never for credentials, API keys, or private data.

## Responsive design and accessibility

The audience for this website is broad, so responsiveness and accessibility were treated as core requirements rather than finishing touches.

The layout adapts across desktop, tablet, and mobile screen sizes. The desktop navigation becomes a mobile menu, fundraising information stacks into a more readable layout, and the gallery adjusts the number of visible slides based on viewport width.

A few accessibility considerations included:

- Semantic sections, headings, navigation, lists, and buttons.
- Clear alternative text for all meaningful images.
- Descriptive labels for gallery controls and the mobile navigation toggle.
- A fundraising progress indicator with ARIA values and readable text.
- Keyboard support for closing the mobile menu with the Escape key.
- Visible focus states for keyboard navigation.
- Lazy-loaded gallery images to improve page performance.

The image gallery was a particularly interesting area. It needed to feel polished without becoming a barrier for keyboard users or users on slower connections. The carousel uses accessible controls and loads images on demand instead of forcing every image to download immediately.

## Challenges and decisions

### Keeping the fundraising data credible

Fundraising pages are sensitive to inconsistencies. A goal shown in one section should match the milestones and cost breakdown shown elsewhere.

To prevent drift, the site calculates the fundraising progress display from central configuration values. Currency formatting is handled with the browser’s `Intl.NumberFormat` API, which keeps Canadian dollar amounts consistent throughout the page.

### Supporting different deployment paths

The site may be hosted at a domain root or under a subdirectory, such as a GitHub Pages project URL. Hard-coded asset paths can break in the second case.

To avoid that, public assets are generated with Vite’s configured base path. This allows the same application source to work locally, at a normal production domain, or under a repository-specific GitHub Pages path.

### Balancing dynamic features with simple maintenance

It would have been possible to add a content-management system, database, or custom administrative dashboard. For the project’s needs, that would add cost and maintenance without much benefit.

Instead, the site uses structured JSON files for content that changes occasionally and environment variables for public configuration. This keeps the editing workflow understandable while preserving a clean codebase.

### Integrating prayer times responsibly

Prayer times are provided through external services. Rather than attempting to recreate an unreliable local scheduling system, the site embeds configured external prayer-time resources and provides a fallback link.

This keeps the website focused on its core responsibility while relying on a specialized source for time-sensitive information.

## What I learned

This project reinforced that the “right” technical solution is often the one with the smallest operational footprint.

A static React site may not sound as complex as a full-stack application, but it still requires thoughtful decisions around configuration, accessibility, responsive behaviour, content management, deployment paths, and external integrations.

A few takeaways stood out:

- **Good information architecture is product work.** The order of content has a direct impact on whether visitors understand the project and take action.
- **Static sites are powerful.** For many organizations, they provide the right balance of speed, cost, security, and maintainability.
- **Configuration deserves first-class treatment.** Centralized, validated configuration prevents small content changes from becoming code changes.
- **Accessibility improves the product for everyone.** Clear labels, responsive layouts, keyboard support, and readable visual hierarchy benefit all visitors.
- **Avoid building infrastructure without a real need.** A backend, CMS, or custom payment workflow would create more maintenance work without improving the core experience.

## Conclusion

The CICSNW website is designed to help turn a long-term community vision into a clear, actionable online experience. It provides a focused fundraising journey while also serving as a practical source of information for the current Taiba Musalla community.

The project is a reminder that software can be most valuable when it removes friction from something meaningful: helping a community communicate its goals, build trust, and bring people together around a shared purpose.

The site is live at [cicsnw.org](https://cicsnw.org). Source code is on [GitHub](https://github.com/salmanfs815/cicsnw-website).
