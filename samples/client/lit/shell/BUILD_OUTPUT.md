# A2UI Shell - Build Output Documentation

## Build Successfully Configured! ✅

The Vite build has been configured to generate a production-ready HTML file and assets in the `dist/` folder.

---

## Build Output Structure

```
dist/
├── index.html                       # Main HTML file (7.46 KB)
├── assets/
│   ├── shell-BCGIe2pp.js           # Bundled JavaScript (297 KB)
│   └── shell-BCGIe2pp.js.map       # Source map (1.1 MB)
├── hero.png                         # Light mode hero image (304 KB)
├── hero-dark.png                    # Dark mode hero image (255 KB)
└── sample/
    ├── city_skyline.jpg            # Sample image (285 KB)
    ├── forest_path.jpg             # Sample image (305 KB)
    └── scenic_view.jpg             # Sample image (194 KB)
```

**Total Size**: ~2.5 MB (including source maps)  
**Minified JS**: 297 KB (91 KB gzipped)

---

## What Was Changed

### 1. **vite.config.ts**

```typescript
build: {
  outDir: "dist",                    // Output directory
  emptyOutDir: true,                 // Clean before build
  rollupOptions: {
    input: entry,                    // index.html as entry point
    output: {
      entryFileNames: "assets/[name]-[hash].js",
      chunkFileNames: "assets/[name]-[hash].js",
      assetFileNames: "assets/[name]-[hash][extname]",
    },
  },
  target: "esnext",                  // Modern browsers
  minify: "esbuild",                 // Fast minification
  sourcemap: true,                   // Generate source maps
}
```

**Key Features**:
- ✅ Proper output directory structure
- ✅ Content hashing for cache busting
- ✅ Modern JavaScript (ES modules)
- ✅ Source maps for debugging
- ✅ esbuild minification (faster than terser)

### 2. **package.json**

Added new scripts:

```json
{
  "scripts": {
    "build:vite": "vite build",              // Build HTML + assets
    "build:all": "npm run build && npm run build:vite",  // TypeScript + Vite
    "preview": "vite preview"                 // Preview production build
  }
}
```

---

## How to Build

### Development Build (TypeScript only)
```bash
npm run build
```

**Output**: TypeScript compiled to `dist/` (no HTML)

### Production Build (HTML + Assets)
```bash
npm run build:vite
```

**Output**: Complete production build in `dist/`

### Full Build (TypeScript + HTML)
```bash
npm run build:all
```

**Output**: Everything built and ready for deployment

---

## Generated index.html

The built HTML file includes:

### Features
- ✅ **Inline CSS**: All theme variables and base styles
- ✅ **Google Fonts**: Material Icons + Outfit font
- ✅ **Bundled JS**: Single JavaScript file with hash
- ✅ **Self-contained**: No external dependencies (except fonts)
- ✅ **Production-ready**: Minified and optimized

### Script Tag
```html
<script type="module" crossorigin src="/assets/shell-BCGIe2pp.js"></script>
```

**Benefits**:
- `type="module"` - Modern ES modules
- `crossorigin` - Proper CORS handling
- Hash in filename - Cache busting

### Body Content
```html
<body>
  <a2ui-shell></a2ui-shell>
</body>
```

The custom element `<a2ui-shell>` is defined in the bundled JavaScript.

---

## JavaScript Bundle

### What's Included

The `shell-BCGIe2pp.js` bundle contains:

1. **Application Code**:
   - `app.ts` - Main component
   - `client.ts` - A2A client
   - All UI components from `ui/`
   - Configuration files

2. **Dependencies**:
   - `lit` - Web components framework
   - `@lit-labs/signals` - Reactivity
   - `@a2ui/lit` - A2UI renderer
   - `@a2a-js/sdk` - A2A protocol
   - All other npm packages

3. **Optimizations**:
   - Dead code elimination
   - Tree shaking
   - Minification
   - Code splitting (if needed)

### Bundle Analysis

```
Original Size:  ~800 KB (uncompressed)
Minified:       297 KB
Gzipped:        91 KB  ← What users download
```

**Performance**: Excellent for a full-featured app!

---

## Deployment Options

### Option 1: Static File Server

**Any static server works!**

```bash
# Serve the dist folder
cd dist
python -m http.server 8080
```

**Or use:**
- Nginx
- Apache
- IIS
- GitHub Pages
- Netlify
- Vercel

### Option 2: Vite Preview

```bash
npm run preview
```

**Output**:
```
  ➜  Local:   http://localhost:4173/
  ➜  Network: use --host to expose
```

### Option 3: CDN Deployment

Upload `dist/` folder to:
- AWS S3 + CloudFront
- Google Cloud Storage + CDN
- Azure Blob Storage + CDN
- Cloudflare Pages

**Important**: Set proper cache headers for hashed files!

---

## Cache Strategy

### Files with Hashes (Long Cache)

```
assets/shell-BCGIe2pp.js  → Cache: 1 year
hero.png                   → Cache: 1 year
hero-dark.png             → Cache: 1 year
```

**Why?** Hash changes when content changes

### Files without Hashes (Short Cache)

```
index.html  → Cache: No cache or 1 hour
```

**Why?** Always fetch latest HTML to get new hashes

### Recommended Headers

```nginx
# nginx.conf
location /assets/ {
  expires 1y;
  add_header Cache-Control "public, immutable";
}

location = /index.html {
  expires 1h;
  add_header Cache-Control "public, must-revalidate";
}
```

---

## Environment Variables

### Build Time

The build reads from `.env`:

```env
GEMINI_API_KEY=your_key_here
```

**Note**: Environment variables are **embedded** into the bundle at build time!

### Production Considerations

**⚠️ Security Warning**: 

Don't put sensitive data in `.env` for client builds!

```env
# ❌ DON'T DO THIS (will be in client bundle)
GEMINI_API_KEY=secret_key

# ✅ DO THIS (use a backend proxy)
VITE_API_URL=https://api.yourdomain.com
```

**Solution**: Use backend API instead of direct API keys in frontend.

---

## Testing the Build

### Step 1: Build
```bash
npm run build:vite
```

### Step 2: Verify Files
```bash
ls -lh dist/
# Should see: index.html, assets/, images
```

### Step 3: Preview
```bash
npm run preview
```

### Step 4: Open Browser
```
http://localhost:4173
```

### Step 5: Test Features
- ✅ Page loads
- ✅ Hero image displays
- ✅ Theme toggle works
- ✅ Query submission works
- ✅ A2UI rendering works
- ✅ Console has no errors

---

## Build Performance

### Metrics

```
Build Time:       3.69s
Modules:          211 transformed
Bundle Size:      297 KB (91 KB gzipped)
Source Map:       1.1 MB
```

### Optimization Tips

1. **Code Splitting**: Split vendor and app code
   ```typescript
   build: {
     rollupOptions: {
       output: {
         manualChunks: {
           vendor: ['lit', '@a2ui/lit'],
           a2a: ['@a2a-js/sdk'],
         },
       },
     },
   }
   ```

2. **Lazy Loading**: Import components on demand
   ```typescript
   const module = await import('./heavy-component.js');
   ```

3. **Image Optimization**: Use WebP/AVIF formats
   ```bash
   cwebp hero.png -o hero.webp
   ```

---

## Troubleshooting

### Issue: Build Fails

**Error**: `terser not found`

**Solution**: Changed to esbuild (already done)

### Issue: Large Bundle Size

**Check**: Run bundle analyzer
```bash
npm install --save-dev rollup-plugin-visualizer
```

Add to `vite.config.ts`:
```typescript
import { visualizer } from 'rollup-plugin-visualizer';

plugins: [
  visualizer({ open: true })
]
```

### Issue: Assets Not Loading

**Cause**: Incorrect base path

**Solution**: Set base in `vite.config.ts`
```typescript
export default {
  base: '/your-app/',  // If deployed to subdirectory
}
```

---

## Production Checklist

Before deploying to production:

- [ ] Build completes without errors
- [ ] All assets load correctly
- [ ] Console shows no errors
- [ ] API endpoints are configured
- [ ] Environment variables are safe
- [ ] Cache headers are set
- [ ] HTTPS is enabled
- [ ] CORS is configured
- [ ] Error monitoring is set up
- [ ] Analytics are configured
- [ ] Performance is tested
- [ ] Mobile/tablet tested
- [ ] Different browsers tested

---

## Next Steps

### For Development
```bash
npm run dev
```

### For Production
```bash
npm run build:vite
npm run preview  # Test locally
# Then deploy dist/ folder
```

### For Distribution
Package the `dist/` folder and distribute:
- Zip file
- Docker image
- Static hosting
- CDN

---

## File Sizes Breakdown

| File | Size | Gzipped | Purpose |
|------|------|---------|---------|
| `index.html` | 7.46 KB | 2.47 KB | Entry point |
| `shell-*.js` | 297 KB | 91 KB | Application code |
| `shell-*.js.map` | 1.1 MB | - | Debug source map |
| `hero.png` | 304 KB | - | Light theme image |
| `hero-dark.png` | 255 KB | - | Dark theme image |

**Total Download** (on first visit): ~400 KB (compressed)  
**Total Download** (cached): <10 KB (HTML only)

---

## Summary

✅ **Build Configured**: Vite properly generates HTML + assets  
✅ **Production Ready**: Minified, hashed, optimized  
✅ **Deploy Anywhere**: Static files work on any server  
✅ **Cache Friendly**: Hashed filenames for efficient caching  
✅ **Source Maps**: Included for debugging  
✅ **Performance**: 91 KB gzipped - excellent!  

**The A2UI Shell is now ready for production deployment!** 🚀

---

**Generated**: January 2026  
**Build Tool**: Vite 7.3.0  
**Target**: ESNext (Modern Browsers)


