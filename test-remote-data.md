# Testing Remote Data Configuration

## Testing Commands

### Development (Local API)
```bash
# Start local API server first
cd ../
python -m http.server 8000 --directory api

# Test development build (in another terminal)
cd site
hugo server -D --config config/development/hugo.toml --environment development

# Access: http://localhost:1313
```

### Production (Remote API)
```bash
# Test production build with remote data
hugo --config config/production/hugo.toml --environment production

# Or test locally with production config
hugo server -D --config config/production/hugo.toml --environment production
```

### Environment Testing
```bash
# Check configuration for development
hugo config --config config/development/hugo.toml

# Check configuration for production
hugo config --config config/production/hugo.toml
```

## Expected Behavior

### With use_remote_data = false (Development)
- Site uses JavaScript to fetch data from local API
- Initial page shows loading spinner
- Data loads via client-side fetch requests

### With use_remote_data = true (Production)
- Hugo fetches data during build time
- Initial page shows server-rendered posts
- Additional pages load via JavaScript for "Load more"

## Validation Checklist

- [ ] Development config uses localhost:8000
- [ ] Production config uses https://api.popefeed.com
- [ ] Server-side rendering works with remote data
- [ ] Client-side loading still works for additional pages
- [ ] Error handling works for failed remote requests
- [ ] Environment switching works correctly

## Troubleshooting

### If remote data fails to load:
1. Check if https://api.popefeed.com is accessible
2. Verify JSON structure matches expected format
3. Check Hugo warnings during build
4. Ensure HTTPS is used (Hugo rejects HTTP in production)

### If build is slow:
1. Hugo caches remote resources automatically
2. Subsequent builds should be faster
3. Check network connectivity

### If JavaScript features don't work:
1. Verify window.PopeFeedConfig is set correctly
2. Check browser console for errors
3. Ensure API URLs are accessible from browser