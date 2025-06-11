# Deployment Guide 🚀

This guide covers multiple deployment options for your My Bathroom Mats UK e-commerce application.

## 🌟 Vercel (Recommended)

Vercel offers the best experience for React applications with automatic deployments.

### Quick Deploy

1. Push your code to GitHub
2. Go to [vercel.com](https://vercel.com)
3. Click "New Project"
4. Import your GitHub repository
5. Deploy! ✨

### Manual Setup

```bash
# Install Vercel CLI
npm install -g vercel

# Login to Vercel
vercel login

# Deploy from project directory
vercel

# For production deployment
vercel --prod
```

### Environment Variables (Vercel)

In your Vercel dashboard, add these environment variables:

```
VITE_SHOPIFY_STORE_URL=your-store.myshopify.com
VITE_SHOPIFY_STOREFRONT_TOKEN=your_storefront_access_token
VITE_APP_URL=https://your-domain.vercel.app
```

## 🔥 Netlify

Great for static sites with form handling capabilities.

### Drag & Drop Deploy

1. Build your project: `npm run build`
2. Go to [netlify.com](https://netlify.com)
3. Drag the `dist/` folder to deploy area

### Git Integration

1. Connect your GitHub repository
2. Set build command: `npm run build`
3. Set publish directory: `dist`
4. Deploy automatically on push

### Netlify Configuration

Create `netlify.toml` in project root:

```toml
[build]
  command = "npm run build"
  publish = "dist"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[build.environment]
  NODE_VERSION = "18"
```

## ☁️ AWS S3 + CloudFront

For enterprise-grade hosting with CDN.

### Build and Upload

```bash
# Build project
npm run build

# Install AWS CLI
pip install awscli

# Configure AWS credentials
aws configure

# Create S3 bucket
aws s3 mb s3://mybathroommats-uk

# Upload files
aws s3 sync dist/ s3://mybathroommats-uk --delete

# Set up static website hosting
aws s3 website s3://mybathroommats-uk --index-document index.html --error-document index.html
```

## 🐳 Docker Deployment

For containerized deployments.

### Dockerfile

```dockerfile
# Build stage
FROM node:18-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Serve stage
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### nginx.conf

```nginx
events {}
http {
    include /etc/nginx/mime.types;

    server {
        listen 80;
        root /usr/share/nginx/html;
        index index.html;

        location / {
            try_files $uri $uri/ /index.html;
        }
    }
}
```

### Deploy with Docker

```bash
# Build image
docker build -t mybathroommats-uk .

# Run container
docker run -p 80:80 mybathroommats-uk
```

## 🎯 GitHub Pages

Free hosting for public repositories.

### Setup

1. Install gh-pages: `npm install --save-dev gh-pages`
2. Add to package.json:

```json
{
  "homepage": "https://yourusername.github.io/mybathroommats-uk",
  "scripts": {
    "predeploy": "npm run build",
    "deploy": "gh-pages -d dist"
  }
}
```

3. Deploy: `npm run deploy`

### Update vite.config.ts

```typescript
export default defineConfig({
  plugins: [react()],
  base: "/mybathroommats-uk/",
});
```

## 🏢 Traditional Web Hosting

For shared hosting providers like cPanel.

### Steps

1. Build project: `npm run build`
2. Upload contents of `dist/` folder to public_html
3. Ensure server supports SPA routing

### .htaccess (Apache)

```apache
Options -MultiViews
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.html [QSA,L]
```

## 🔧 Environment Configuration

### Production Environment Variables

```bash
# Shopify Integration
VITE_SHOPIFY_STORE_URL=mybathroommats.myshopify.com
VITE_SHOPIFY_STOREFRONT_TOKEN=your_token_here

# Analytics
VITE_GA_TRACKING_ID=G-XXXXXXXXXX
VITE_HOTJAR_ID=your_hotjar_id

# Features
VITE_ENABLE_CART=true
VITE_ENABLE_WISHLIST=true
VITE_ENABLE_REVIEWS=true

# API URLs
VITE_API_URL=https://api.mybathroommats.uk
VITE_CHECKOUT_URL=https://checkout.mybathroommats.uk
```

## 📊 Performance Optimization

### Build Optimization

```json
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          router: ['react-router-dom'],
          ui: ['@radix-ui/react-dialog', '@radix-ui/react-dropdown-menu']
        }
      }
    }
  }
})
```

### Image Optimization

- Use WebP format with fallbacks
- Implement lazy loading
- Add responsive image sizes
- Compress images before deployment

## 🔐 Security Headers

### Netlify \_headers

```
/*
  X-Frame-Options: DENY
  X-XSS-Protection: 1; mode=block
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  Content-Security-Policy: default-src 'self'; img-src 'self' data: https:; script-src 'self' 'unsafe-inline' https://cdn.shopify.com
```

## 📈 Monitoring

### Add monitoring to your deployed app:

- **Google Analytics**: Track user behavior
- **Sentry**: Error monitoring
- **Hotjar**: User experience insights
- **PageSpeed Insights**: Performance monitoring

## 🚀 CI/CD Pipeline

### GitHub Actions (.github/workflows/deploy.yml)

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Setup Node.js
        uses: actions/setup-node@v2
        with:
          node-version: "18"

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID }}
          vercel-project-id: ${{ secrets.PROJECT_ID }}
          vercel-args: "--prod"
```

## 📱 Mobile App (Future)

Consider these options for mobile apps:

- **React Native**: Share code between web and mobile
- **Capacitor**: Turn your web app into native mobile apps
- **PWA**: Progressive Web App for app-like experience

---

Choose the deployment method that best fits your needs and budget. Vercel is recommended for most use cases due to its simplicity and performance.
