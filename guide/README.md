# diagmatrix's SQL Style Guide

This is my SQL style guide for development. These guidelines are meant to inform, not dictate. Every project comes with its own constraints: efficiency concerns, business rules, specific contexts... There will be times when following a rule to the letter does more harm than good.

Most of the rules come from other SQL style guides, where a decision has been made with some reasoning, and no other guide contradicts it. Those rules are as-is, with no modifications planned. Others, however, come mostly from my opinion or are not a consensus between the guides.

## Rules structure

 1. [Formatting](formatting.md)
 2. [Keywords](keywords.md)
 3. [Naming](naming.md)
 4. [Aliasing](aliasing.md)
 5. [Comments](comments.md)
 6. [Data types](data-types.md)
 7. [Constraints and keys](constraints-keys.md)
 8. [JOIN clauses](joins.md)
 9. [Views](views.md)
 10. [Subqueries and CTEs](subqueries-ctes.md)
 11. [Queries](queries.md)

## Sources

### Celko

*Joe Celko's SQL Programming Style* is a seminal, vendor-agnostic textbook that laid the foundational philosophy for modern SQL formatting and best practices. Published in 2005, it remains highly respected for its deep dive into the theoretical and structural aspects of writing clean code, even if some of its specific efficiency considerations or procedural advice feel slightly dated in the context of modern database architectures. Often cited as the direct predecessor to many of today’s web-based frameworks, it is widely utilized by database administrators and backend developers seeking a rigorous, historically grounded approach to database design that emphasizes logic and standardized typography over automated tooling.

**You can find the source here:** https://poornaprajnalibrary.com/admin/uploads/cmbooks/Joe%20Celko%27s%20SQL%20Programming%20Style%20%28The%20Morgan%20Kaufmann%20Series%20in%20Data%20Management%20Systems%29%20%28%20PDFDrive%20%29.pdf

### Holywell

Simon Holywell's SQL Style Guide is a widely adopted, vendor-agnostic framework designed to promote consistency, readability, and maintainability in collaborative database development. Created as an easily accessible, web-based successor to Joe Celko’s textbook standards, it is extensively utilized across the data industry by startups, enterprise teams, and open-source projects, often serving as a foundational baseline for corporate guidelines. Its significant popularity is reflected in its high search visibility, translations into over a dozen languages, ongoing community debates regarding its polarising formatting rules (such as "river alignment"), and the development of dedicated automated linters built to enforce its specific conventions.

**You can find the source here:** https://www.sqlstyle.guide/

### PostgreSQL

The PostgreSQL implicit style guide is an unofficial but highly influential set of conventions derived directly from the consistent formatting found within the official PostgreSQL documentation. Rather than a published rulebook, it serves as a pragmatic, dialect-specific baseline that native developers naturally adopt. Its popularity stems from its absolute alignment with the database's own ecosystem, providing a reliable, friction-free reference point for engineers building applications, writing tutorials, or developing extensions specifically tailored to the nuances of the Postgres environment.

**You can find the source here:** https://www.postgresql.org/docs/current/sql.html

### PG.ai

The Postgres.ai Style Guide is a modern, dialect-specific framework tailored specifically for the PostgreSQL ecosystem, with a strong emphasis on database-as-code principles and CI/CD integration. Designed for large-scale, automated environments, it moves beyond basic formatting to include rules that optimize for query performance, safe database migrations, and seamless collaboration in Git-based workflows. It is frequently adopted by advanced database engineering teams and DBAs who require a rigorous standard that addresses both the visual readability of SQL and the strict operational realities of managing high-traffic, production-grade Postgres deployments.

**You can find the source here:** https://postgres.ai/rules/rules-pages

### Mozilla

Mozilla’s SQL Style Guide is a specialized framework embedded within their broader data documentation, designed primarily to handle the complexities of large-scale telemetry and analytical data warehousing. The guide is heavily focused on readability and maintainability for massive analytical queries. It is particularly popular among data engineers and analysts working in modern cloud data environments (such as BigQuery), serving as a stellar example of how a major open-source organization structures its internal data pipelines and querying practices.

**You can find the source here:** https://docs.telemetry.mozilla.org/concepts/sql_style

### Brooklyn

The Brooklyn Data Co. SQL Style Guide is a prominent, modern framework heavily oriented towards analytics engineering and the modern data stack, particularly within dbt (data build tool) ecosystems. It champions pragmatic readability and developer experience, advocating for accessible conventions. Widely adopted by data consultancies and agile startups, it serves as a highly practical, analytics-first baseline that prioritizes clean visual hierarchy and consistency for teams building robust data warehouses in platforms like Snowflake or BigQuery.

**You can find the source here:** https://github.com/brooklyn-data/co/blob/main/sql_style_guide.md

### GitLab

The GitLab SQL Style Guide is a highly opinionated and rigorously enforced framework developed to maintain consistency across one of the world’s largest open-source, Postgres-backed monolithic applications. Focused heavily on performance and the avoidance of common database anti-patterns, it mandates strict formatting rules, naming conventions, and query structures optimized for massive codebases and continuous integration. Because it is actively used to govern global community contributions via merge requests, it is widely referenced by backend engineers and enterprise teams seeking a battle-tested standard for managing complex database interactions in highly collaborative, high-velocity environments.

**You can find the source here:** https://handbook.gitlab.com/handbook/enterprise-data/platform/sql-style-guide/

### Mazur

Matt Mazur's SQL Style Guide is a highly influential, independently created framework that prioritizes human readability and a clean visual hierarchy for analytical query writing. Born from practical experience in data analytics, it focuses on sensible, easy-to-follow formatting rules without the overwhelming rigidity of some enterprise-level guides. Its accessibility and focus on the "visual shape" of the code have made it a favorite among data scientists, analysts, and indie developers who want a lightweight, elegant standard that makes complex data wrangling scripts instantly understandable to anyone reviewing the code.

**You can find the source here:** https://github.com/mattm/sql-style-guide

### Supabase

The Supabase Postgres SQL Style Guide is a modern, highly pragmatic framework uniquely designed to be ingested as a system prompt by AI coding assistants (such as GitHub Copilot, Cursor, ...) to enforce consistent styling during automated code generation. Tailored specifically for the Supabase ecosystem, it champions accessible, developer-friendly conventions that prioritize readability over raw performance. Its popularity is surging among full-stack developers and agile teams utilizing backend-as-a-service (BaaS) platforms, as it serves a powerful dual purpose: acting as a clean, readable rulebook for human engineers while actively preventing AI tools from generating chaotic, non-standard database migrations and queries in Postgres environments.

**You can find the source here:** https://supabase.com/docs/guides/getting-started/ai-prompts/code-format-sql
