# GitHub Pages Deployment Guide for maybonmusic.com

## ✅ Completed Setup

1. **SvelteKit Configuration**: Configured for static export with GitHub Pages adapter
2. **GitHub Actions**: Automated deployment workflow created
3. **CNAME File**: Custom domain configuration added
4. **Build Scripts**: Updated package.json with deployment scripts

## 🚀 Next Steps to Deploy

### 1. Push to GitHub Repository

```bash
# Initialize git repository (if not already done)
git init

# Add all files
git add .

# Commit files
git commit -m "Initial commit - Maybon countdown site"

# Add GitHub repository as remote (replace YOUR_USERNAME)
git remote add origin https://github.com/EivindHacker/Maybon-Homepage.git

# Push to main branch
git push -u origin main
```

### 2. Enable GitHub Pages

1. Go to your GitHub repository settings
2. Scroll down to **"Pages"** in the left sidebar
3. Under **"Source"**, select **"GitHub Actions"**
4. Your site will deploy automatically when you push to main

### 3. Configure DNS at one.com

Log into your one.com control panel and set up these DNS records:

#### For maybonmusic.com (root domain)

```
Type: A
Name: @ (or leave blank)
Value: 185.199.108.153
TTL: 3600

Type: A
Name: @ (or leave blank)  
Value: 185.199.109.153
TTL: 3600

Type: A
Name: @ (or leave blank)
Value: 185.199.110.153
TTL: 3600

Type: A
Name: @ (or leave blank)
Value: 185.199.111.153
TTL: 3600
```

#### For <www.maybonmusic.com> (subdomain)

```
Type: CNAME
Name: www
Value: EivindHacker.github.io
TTL: 3600
```

### 4. Configure Custom Domain in GitHub

1. Go to your repository settings
2. Scroll to **"Pages"**
3. Under **"Custom domain"**, enter: `maybonmusic.com`
4. Check **"Enforce HTTPS"** (after DNS propagates)

## 🔧 Local Development

```bash
# Start development server
npm run dev

# Build for production (test deployment)
npm run build

# Preview production build
npm run preview
```

## 🌐 DNS Propagation

- DNS changes can take 24-48 hours to propagate globally
- You can check propagation status at: <https://dnschecker.org>
- Your site will be available at:
  - `https://maybonmusic.com`
  - `https://www.maybonmusic.com`
  - `https://YOUR_USERNAME.github.io/maybon` (GitHub URL)

## 📂 Project Structure

```
maybon/
├── .github/workflows/deploy.yml  # Auto-deployment
├── src/
│   ├── routes/
│   │   ├── +layout.ts           # Prerender config
│   │   └── +page.svelte         # Main countdown page
│   └── app.html
├── static/
│   ├── CNAME                    # Custom domain
│   └── img/                     # Assets
├── svelte.config.js             # Static adapter config
└── package.json                 # Build scripts
```

## 🎯 Automatic Deployment

Every time you push to the `main` branch:

1. GitHub Actions builds your site
2. Deploys to GitHub Pages
3. Updates live at maybonmusic.com

## 🐛 Troubleshooting

### Site not loading

- Check DNS propagation
- Verify CNAME file exists in `/static/`
- Check GitHub Pages settings

### Build errors

- Run `npm run build` locally first
- Check GitHub Actions logs
- Ensure all dependencies are installed

### Custom domain issues

- Verify DNS records at one.com
- Wait for DNS propagation
- Check GitHub Pages custom domain setting

## 📱 Performance Notes

- Video autoplay may not work on mobile (browser limitations)
- Site is optimized for fast loading
- All assets are statically generated
