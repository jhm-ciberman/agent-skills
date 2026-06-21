# Write documentation in Laravel's expressive style

Write documentation (or any explanatory text) using the same expressive, warm, and clear writing style found in Laravel's official documentation.

**REQUIRED ARGUMENT:** A description of what to document. Examples:
- `Write docs for our authentication system`
- `Document the PaymentService class`
- `Write a PR description for the refactored queue system`
- `Explain how our caching layer works`

If no argument is provided, show an error and stop:
```
Error: You must describe what you want documented.
Usage: /artisan-docs-writer <description of what to document>
```

The user's request: $ARGUMENTS

## Step 1: Read Laravel documentation to absorb the writing style

You MUST read real Laravel documentation pages using `curl` to absorb the writing style firsthand. Do NOT use the WebFetch tool. It uses an intermediate AI model that would lose the literal prose style. Use raw `curl` and read the full output yourself.

**CRITICAL: Always read the FULL curl output. Do NOT use `head`, `tail`, or any form of truncation. Do NOT paginate. You need the complete prose to capture the style.**

Read generously. Pick at least 6 to 10 pages from the index below. More is better. Context window is not a concern. The more Laravel prose you absorb, the more natural the output will sound.

Choose pages that are relevant to the topic the user wants documented and that showcase a variety of documentation flavors (conceptual, practical, example-heavy, long-form, concise). Use your judgment for every request. There is no fixed set of pages that must always be read.

Run the curl commands in parallel. Each URL is formed by appending `.md` to the path. Example:
```bash
curl -s https://laravel.com/docs/12.x/middleware.md
```

### Full Laravel 12.x Documentation Index

Use this index to choose which pages to read. All URLs follow the pattern `https://laravel.com/docs/12.x/<slug>.md`

**Prologue:**
- `releases` : Release Notes
- `upgrade` : Upgrade Guide
- `contributions` : Contribution Guide

**Getting Started:**
- `installation` : Installation
- `configuration` : Configuration
- `ai` : Agentic Development
- `structure` : Directory Structure
- `frontend` : Frontend
- `starter-kits` : Starter Kits
- `deployment` : Deployment

**Architecture Concepts:**
- `lifecycle` : Request Lifecycle
- `container` : Service Container
- `providers` : Service Providers
- `facades` : Facades

**The Basics:**
- `routing` : Routing
- `middleware` : Middleware
- `csrf` : CSRF Protection
- `controllers` : Controllers
- `requests` : Requests
- `responses` : Responses
- `views` : Views
- `blade` : Blade Templates
- `vite` : Asset Bundling
- `urls` : URL Generation
- `session` : Session
- `validation` : Validation
- `errors` : Error Handling
- `logging` : Logging

**Digging Deeper:**
- `artisan` : Artisan Console
- `broadcasting` : Broadcasting
- `cache` : Cache
- `collections` : Collections
- `concurrency` : Concurrency
- `context` : Context
- `contracts` : Contracts
- `events` : Events
- `filesystem` : File Storage
- `helpers` : Helpers
- `http-client` : HTTP Client
- `localization` : Localization
- `mail` : Mail
- `notifications` : Notifications
- `packages` : Package Development
- `processes` : Processes
- `queues` : Queues
- `rate-limiting` : Rate Limiting
- `search` : Search
- `strings` : Strings
- `scheduling` : Task Scheduling

**Security:**
- `authentication` : Authentication
- `authorization` : Authorization
- `verification` : Email Verification
- `encryption` : Encryption
- `hashing` : Hashing
- `passwords` : Password Reset

**Database:**
- `database` : Getting Started
- `queries` : Query Builder
- `pagination` : Pagination
- `migrations` : Migrations
- `seeding` : Seeding
- `redis` : Redis
- `mongodb` : MongoDB

**Eloquent ORM:**
- `eloquent` : Getting Started
- `eloquent-relationships` : Relationships
- `eloquent-collections` : Collections
- `eloquent-mutators` : Mutators / Casts
- `eloquent-resources` : API Resources
- `eloquent-serialization` : Serialization
- `eloquent-factories` : Factories

**AI:**
- `ai-sdk` : AI SDK
- `mcp` : MCP
- `boost` : Boost

**Testing:**
- `testing` : Getting Started
- `http-tests` : HTTP Tests
- `console-tests` : Console Tests
- `dusk` : Browser Tests
- `database-testing` : Database
- `mocking` : Mocking

**Packages:**
- `billing` : Cashier (Stripe)
- `cashier-paddle` : Cashier (Paddle)
- `dusk` : Dusk
- `envoy` : Envoy
- `fortify` : Fortify
- `folio` : Folio
- `homestead` : Homestead
- `horizon` : Horizon
- `mix` : Mix
- `octane` : Octane
- `passport` : Passport
- `pennant` : Pennant
- `pint` : Pint
- `precognition` : Precognition
- `prompts` : Prompts
- `pulse` : Pulse
- `reverb` : Reverb
- `sail` : Sail
- `sanctum` : Sanctum
- `scout` : Scout
- `socialite` : Socialite
- `telescope` : Telescope
- `valet` : Valet

## Step 2: Analyze the style patterns

After reading the pages, internalize these style patterns (which you'll see confirmed in the text):

- **Conversational but authoritative.** Laravel docs speak directly to "you" as a developer. They say "you may", "you should", "of course", "in fact". Warm and confident, never stiff or robotic.
- **Progressive disclosure.** Start with the simplest use case, then layer complexity. Don't frontload every option. The reader should feel guided, not overwhelmed.
- **Practical code examples.** Prose and code alternate naturally. Explain a concept in 1 to 3 sentences, then show it. Don't write long blocks of prose without code, and don't dump code without context.
- **Explains the "why".** Don't just show *what* to do. Explain *why* it works that way. Give the reader mental models, not just instructions.
- **Clear structure.** Use a table of contents at the top. Use headings and subheadings generously. Each section should be self-contained enough to link to.
- **Tips and warnings.** Use `> [!NOTE]`, `> [!TIP]`, and `> [!WARNING]` blockquotes for important callouts. Sparingly, not on every paragraph.
- **Cross-references.** Link to related topics naturally within the prose (e.g., "as mentioned in the documentation on [views](/docs/...)").
- **Concise but complete.** Every sentence earns its place. No filler, no redundancy, but nothing important is left out either.
- **No em dashes.** Never use em dashes (the long "—" character). Use regular dashes sparingly, or restructure the sentence instead. Follow the Laravel docs' punctuation style closely.

## Step 3: Examine the user's codebase

Before writing anything, read the relevant source code thoroughly. You can't document what you don't understand.

1. **Understand the full system.** Don't just read the specific files being documented or changed. Explore the surrounding codebase to understand the broader architecture, how components relate to each other, and the conventions the project follows. Context matters.
2. **For PR descriptions,** read the diff *and* the parts of the codebase the changes interact with. A good PR description explains the change in context, not in isolation.
3. **Look at existing documentation** in the project to match any conventions already in place (file structure, naming, etc.).
4. **Identify the audience.** Is this for end-users, API consumers, or fellow developers? Adjust depth accordingly.

## Step 4: Ask questions if anything is unclear

Before writing, use `AskUserQuestion` to clarify anything that is not straightforward. This includes:

- Questions about the architecture or how components fit together.
- What exactly should be documented and what can be left out.
- Who the target audience is.
- Whether certain design decisions were intentional or accidental.
- The desired level of detail or formality.
- Anything ambiguous about the user's request.

Do not guess. Ask. It is better to ask one or two good questions upfront than to rewrite the entire document later.

## Step 5: Write the documentation

Write the requested documentation applying the Laravel style. Follow these rules:

1. **Start with a clear introduction** that tells the reader what this thing is and why they'd care, in 2 to 3 sentences max.
2. **Use practical, runnable examples.** Don't show abstract pseudocode. Show real code that someone could paste and adapt.
3. **Build from simple to advanced.** The first example should be the most common use case. Edge cases and advanced configuration come later.
4. **Use the Laravel voice.** Write like you're a knowledgeable friend explaining something. Direct, warm, and clear. Avoid corporate jargon, passive voice, and unnecessary formality.
5. **Format for scannability.** Use headings, short paragraphs (2 to 4 sentences), and code blocks. A developer scanning the page should be able to find what they need quickly.

## Important rules

- Do NOT mention that you read Laravel docs or that you're imitating a style. The output should simply *be* well-written documentation.
- Do NOT copy Laravel-specific content. Only absorb the *writing style* and *structural patterns*.
- Do NOT over-explain. Trust that the reader is a developer who understands basic concepts.
- Do NOT add meta-commentary like "this documentation covers..." Just start writing the actual docs.
- Do NOT use em dashes. Ever. Restructure the sentence or use periods, commas, or parentheses instead.
- The output format depends on what the user asked for. If they want Markdown docs, produce Markdown. If they want a PR description, produce a PR description. If they want inline code comments, produce comments. Always in the Laravel voice.
