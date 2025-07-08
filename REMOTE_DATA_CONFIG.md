# Hugo Remote Data Configuration

## Overview

This document describes how to configure Hugo to fetch data from the remote Pope Feed API at `https://api.popefeed.com` instead of using local API files.

## Remote API Structure

The remote API follows the same structure as the local API:

```
https://api.popefeed.com/
├── popes.json                     # List of all popes
├── popes/                         # Pope data and images
│   ├── francesco.jpg              # Official Vatican photos
│   ├── benedict-xvi.jpg
│   └── ...
├── documents/                     # Document metadata and PDFs
│   ├── document-id.json           # Document metadata
│   └── document-id/
│       ├── en.pdf                 # PDF files by language
│       ├── es.pdf
│       └── ...
├── posts/                         # Social media-style feed
│   ├── index.json                 # Pagination metadata
│   ├── page=1.json                # First page (latest documents)
│   ├── page=2.json                # Second page
│   └── ...
└── feed/                          # Language-specific feeds
    ├── en.json                    # English feed
    ├── es.json                    # Spanish feed
    └── ...
```

## Hugo Remote Data Fetching

Hugo provides built-in support for fetching remote data using:

1. **`getJSON` function** - For JSON data fetching
2. **`resources.GetRemote` function** - For remote resources (Hugo 0.91+)
3. **Data templates** - For processing fetched data

### Configuration Methods

#### Method 1: Using getJSON Function

```go-template
{{ $popes := getJSON "https://api.popefeed.com/popes.json" }}
{{ $posts := getJSON "https://api.popefeed.com/posts/page=1.json" }}
```

#### Method 2: Using resources.GetRemote (Preferred)

```go-template
{{ $popesResource := resources.GetRemote "https://api.popefeed.com/popes.json" }}
{{ $popes := $popesResource | transform.Unmarshal }}
```

#### Method 3: Configuration-based Data Sources

```toml
# hugo.toml
[params]
  api_base_url = "https://api.popefeed.com"

[params.remote_data]
  popes_url = "https://api.popefeed.com/popes.json"
  posts_url = "https://api.popefeed.com/posts/page=1.json"
  feed_en_url = "https://api.popefeed.com/feed/en.json"
```

## Configuration Changes Required

### 1. Hugo Configuration (hugo.toml)

```toml
# Current local configuration
[params]
  api_base_url = "http://localhost:8000"

# New remote configuration
[params]
  api_base_url = "https://api.popefeed.com"
  use_remote_data = true

[params.remote_data]
  popes_url = "https://api.popefeed.com/popes.json"
  posts_index_url = "https://api.popefeed.com/posts/index.json"
  posts_page_url = "https://api.popefeed.com/posts/page="
  feed_base_url = "https://api.popefeed.com/feed/"
```

### 2. Layout Templates

#### Current JavaScript-based approach:
```javascript
// In layouts/_default/baseof.html
const API_BASE_URL = '{{ .Site.Params.api_base_url }}';
fetch(`${API_BASE_URL}/posts/page=1.json`)
```

#### New Hugo template-based approach:
```go-template
<!-- In layouts/index.html -->
{{ $postsResource := resources.GetRemote .Site.Params.remote_data.posts_index_url }}
{{ $postsIndex := $postsResource | transform.Unmarshal }}

{{ $page1Resource := resources.GetRemote (printf "%spage=1.json" .Site.Params.remote_data.posts_page_url) }}
{{ $page1 := $page1Resource | transform.Unmarshal }}
```

### 3. Data Processing

```go-template
<!-- Process remote data -->
{{ range $page1.posts }}
  <article class="post-card">
    <div class="post-header">
      <img src="{{ .pope.avatar }}" class="pope-avatar" alt="{{ .pope.name }}">
      <div class="pope-info">
        <h3>{{ .pope.name }}</h3>
        <p>@{{ .pope.handle }}</p>
      </div>
    </div>
    <div class="post-content">
      <h2>{{ .document.title }}</h2>
      <p>{{ .document.excerpt }}</p>
    </div>
  </article>
{{ end }}
```

## Benefits of Remote Data Fetching

### 1. Build-time Data Fetching
- Data is fetched during Hugo build process
- Generates static HTML with embedded data
- No client-side API calls needed
- Better SEO and performance

### 2. Caching and Performance
- Hugo caches remote resources automatically
- Faster subsequent builds
- Reduced client-side JavaScript

### 3. Offline Capability
- Generated site works without API server
- Data is embedded in static files
- Better reliability

## Implementation Strategy

### Phase 1: Hybrid Approach
Keep both local and remote data support:

```go-template
{{ if .Site.Params.use_remote_data }}
  {{ $posts := getJSON .Site.Params.remote_data.posts_url }}
{{ else }}
  {{ $posts := getJSON (printf "%s/posts/page=1.json" .Site.Params.api_base_url) }}
{{ end }}
```

### Phase 2: Full Remote Migration
Replace all local data fetching with remote calls.

### Phase 3: Optimization
- Implement data caching strategies
- Add error handling for failed requests
- Progressive enhancement for client-side features

## Error Handling

```go-template
{{ $postsResource := resources.GetRemote .Site.Params.remote_data.posts_url }}
{{ if $postsResource.Err }}
  <div class="error">
    <p>Error loading posts: {{ $postsResource.Err }}</p>
  </div>
{{ else }}
  {{ $posts := $postsResource | transform.Unmarshal }}
  <!-- Render posts -->
{{ end }}
```

## Environment Configuration

### Development
```toml
# config/development/hugo.toml
[params]
  api_base_url = "http://localhost:8000"
  use_remote_data = false
```

### Staging
```toml
# config/staging/hugo.toml
[params]
  api_base_url = "https://staging.api.popefeed.com"
  use_remote_data = true
```

### Production
```toml
# config/production/hugo.toml
[params]
  api_base_url = "https://api.popefeed.com"
  use_remote_data = true
```

## Build Process Changes

### Current Process
1. Start local API server (`python -m http.server 8000 --directory api`)
2. Start Hugo server (`hugo server -D`)
3. Site fetches data from localhost via JavaScript

### New Process
1. Hugo fetches data from `https://api.popefeed.com` during build
2. Data is embedded in generated HTML
3. No need for local API server
4. Faster, more reliable site

## Security Considerations

### HTTPS Requirements
- Remote API must use HTTPS
- Hugo will reject HTTP requests in production builds

### CORS Not Required
- Hugo fetches data server-side during build
- No browser CORS restrictions

### Rate Limiting
- Consider API rate limits during builds
- Implement retry logic if needed

## Performance Impact

### Build Time
- Initial build: Slightly slower (remote fetching)
- Subsequent builds: Faster (Hugo caching)
- Data changes: Requires rebuild

### Runtime Performance
- Faster page loads (no API calls)
- Better SEO (server-side rendered content)
- Improved Core Web Vitals

## Migration Checklist

- [ ] Update hugo.toml with remote API URLs
- [ ] Modify layouts to use Hugo data fetching
- [ ] Add error handling for remote requests
- [ ] Test with remote API
- [ ] Update build/deployment scripts
- [ ] Update documentation
- [ ] Configure environment-specific settings

## Testing Strategy

### Local Testing
```bash
# Test with remote data
hugo server -D --config config/production/hugo.toml

# Test with local data
hugo server -D --config config/development/hugo.toml
```

### Build Testing
```bash
# Production build with remote data
hugo --minify --config config/production/hugo.toml

# Verify generated files contain data
grep -r "Pope Francis" public/
```

## Rollback Plan

If remote data fetching causes issues:

1. Revert hugo.toml changes
2. Re-enable JavaScript-based data fetching
3. Restart local API server
4. Rebuild and redeploy

The hybrid approach allows for quick switching between local and remote data sources.