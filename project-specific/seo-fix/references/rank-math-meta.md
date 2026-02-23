# Rank Math Meta Keys Reference

## Post/Page Meta (via REST API `meta` object)

| Key | Description | Notes |
|-----|-------------|-------|
| `rank_math_title` | SEO title | Supports `%sep%`, `%sitename%`, `%title%` variables |
| `rank_math_description` | Meta description | Keep 120-155 chars, no trailing newlines |
| `rank_math_focus_keyword` | Focus keywords | Comma-separated, primary first |
| `rank_math_robots` | Robot directives | Array: `index`, `noindex`, `follow`, `nofollow` |
| `rank_math_canonical_url` | Canonical URL | Leave empty for self-referencing |
| `rank_math_schema_Article` | Article schema JSON | For blog posts |
| `rank_math_schema_LocalBusiness` | Local business schema | For location/service pages |
| `rank_math_breadcrumb_title` | Breadcrumb title | Short version for breadcrumb trail |

## Rank Math Variables

| Variable | Expands To |
|----------|-----------|
| `%title%` | Post/page title |
| `%sep%` | Separator (configured in Rank Math settings) |
| `%sitename%` | Site name ("Outdoor Butler") |
| `%currentdate%` | Current date |
| `%currentyear%` | Current year |

## Common SEO Title Patterns

```
# Service pages
{Service Name} in {Location} %sep% %sitename%

# Location pages
Outdoor Cleaning Services in {City}, Alicante %sep% %sitename%

# Blog posts
{Question or Topic} %sep% %sitename%
```

## WordPress REST API Notes

- Rank Math registers its meta with `show_in_rest`, accessible via standard `meta` object
- Posts: `/wp/v2/posts/{id}` | Pages: `/wp/v2/pages/{id}`
- Meta update requires `edit_posts` capability (Application Password provides this)
- Always include `Content-Type: application/json` header
- Tag/category updates use term IDs, not names
