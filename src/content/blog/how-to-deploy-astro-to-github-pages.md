---
title: "How to Deploy Your Astro Portfolio to GitHub Pages: A Step-by-Step Guide"
description: "A Step-by-Step Guide to Deploying Your Astro Portfolio to GitHub Pages"
date: 2026-03-23
image: "/blogs/githubpages.png"
tags: ["Astro", "Web Dev", "GitHub Pages"]
---

# Deploy Your Astro Portfolio to GitHub Pages: A Step-by-Step Guide

So you’ve built a stunning portfolio with Astro, fast, modern, and perfect for showcasing your work. Now it’s time to get it live for the world to see. And what better place than GitHub Pages? It’s free, easy, and integrates perfectly with your Git workflow.

In this guide, I’ll walk you through deploying your Astro site to GitHub Pages using GitHub Actions (automated builds on every push). By the end, your portfolio will be live at https://yourusername.github.io (or https://yourusername.github.io/repo-name).

What You’ll Need

- A GitHub account (free)
- Your Astro project ready locally
- Git installed and configured
- Node.js (v18 or later)

**Step 1: Prepare Your Astro Project for GitHub Pages**

GitHub Pages serves static files. The good news: Astro outputs a static site by default, no extra adapter needed. But we do need to tell Astro where it will live.
Open astro.config.mjs and add the site and (optionally) base fields:

```js
import { defineConfig } from "astro/config";
export default defineConfig({
  site: "https://yourusername.github.io",
  base: "/your-repo-name", // remove this line if using a user/organization site
});
```

When to use base?

- User / Organization site (username.github.io): no base, just set site.
- Project site (username.github.io/my-portfolio): set base: '/my-portfolio'.

Keep your repo name in sync with base. If your repo is my-portfolio, then base: '/my-portfolio'.

Now test locally, run

```
npm run build
```

It will generate files in dist/ (default). Nothing breaks? Good.

**Step 2: Create the GitHub Actions Workflow**

This is the magic step. Every time you push to main (or your default branch), GitHub will build your site and deploy it to the gh-pages branch.
Create this file in your project:
.github/workflows/deploy.yml

```
name: Deploy Astro site to GitHub Pages
on:
  push:
    branches: [main]
  workflow_dispatch:
permissions:
  contents: read
  pages: write
  id-token: write
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - name: Install dependencies
        run: npm ci
      - name: Build
        run: npm run build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: dist/
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

A few notes:

- If your build output folder is not dist (e.g., you changed it to build), update the path under upload-pages-artifact.
- If your default branch is master, change branches: [ main ] to master.
  Commit and push this file to your repo.

**Step 3: Enable GitHub Pages for Your Repo**

Now go to your repository on GitHub.com:

1. Click Settings → Pages (in the left sidebar under “Code and automation”).
2. Under “Build and deployment” → Source, select GitHub Actions.
   That’s it, GitHub will now use the workflow we just added.
   (If you see an option for “Deploy from a branch” instead, make sure you have the .github/workflows/deploy.yml file present, the Actions method should appear automatically.)

**Step 4: Push and Watch It Deploy**

Push your code to GitHub:

```
git add .
git commit -m "Ready for GitHub Pages"
git push origin main
```

Now go to your repo on GitHub → Actions tab. You’ll see a workflow run. Click on it to watch the build and deploy steps.

Once the green checkmark appears, your portfolio is live!
Visit: https://yourusername.github.io (or https://yourusername.github.io/your-repo-name)

Troubleshooting Common Issues

**My site shows a 404 or broken assets (CSS/images missing)**

- Check base: If you are using a project site (username.github.io/repo), make sure base is set correctly. Then in your components, always use relative or absolute paths from root. Astro’s `<Image />` and asset handling work automatically with the base config but if you have hardcoded /style.css, change it to relative paths or use `import.meta.env.BASE_URL`.

- Try rebuilding locally with `npm run build` and test with `npx serve dist` if it works there, it’s a GitHub Pages path issue.

**GitHub Actions fails on “Upload artifact”**

- Ensure you have the correct permissions in the workflow (the permissions block is required).

- Check that the build step actually creates the dist/ folder.

**The site shows a blank page**

- Open browser DevTools → Console. Look for 404 errors on JS/CSS. That usually points to a base misconfiguration.

Still stuck?

- Astro’s official guide: <span style="color:blue">[Deploy an Astro site to GitHub Pages](https://docs.astro.build/en/guides/deploy/github/)</span>

- GitHub Pages docs: <span style="color:blue">[Configuring a publishing source for your GitHub Pages site](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)</span>

**Bonus: Custom Domain**

Want a custom domain like www.yourname.com?

- Add a CNAME file to your public/ folder with your domain name.
  · Then in GitHub repo Settings → Pages → Custom domain, enter the domain.
  · Update your DNS records (A/ALIAS/CNAME) with your provider.

**Wrapping Up**

That’s it, your Astro portfolio is now deployed on GitHub Pages with continuous deployment. Every git push automatically rebuilds and redeploys your site. No FTP, no server costs, no headaches.

Go share your portfolio link with pride. And if you found this useful, feel free to share it with another Astro builder.

Happy deploying!
