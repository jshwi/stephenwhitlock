---
title: Create Static Blog Site
pubDatetime: 2025-12-18T05:05:00
slug: create-static-blog-site
tags:
  - blog
  - astro
  - astro-paper
  - markdown
description: Create Static Blog Site With Astro Paper
---

For most accurate replication of this post, use the ref this was modelled on

## Initialise

```shell
$ npm create astro@latest -- --template satnaing/astro-paper#aad5ac6
```

## Configure

If you're using a custom domain use that, otherwise use your github pages domain, in my case jshwi.github.io

This will be your default config for all your posts

```diff file=src/config/config.ts
@@ -1,5 +1,5 @@
 export const SITE = {
-  website: "https://astro-paper.pages.dev/", // replace this with your deployed domain
+  website: "https://www.stephenwhitlock.com/",
   author: "Sat Naing",
   profile: "https://satnaing.dev/",
   desc: "A minimal, responsive and SEO-friendly Astro blog theme.",
```

In Astro Paper, the author key is purely metadata unless you explicitly use it in your theme or components

```diff file=src/config.ts
@@ -1,6 +1,6 @@
 export const SITE = {
   website: "https://www.stephenwhitlock.com/",
-  author: "Sat Naing",
+  author: "Stephen Whitlock",
   profile: "https://satnaing.dev/",
   desc: "A minimal, responsive and SEO-friendly Astro blog theme.",
   title: "AstroPaper",
```

Your personal/portfolio website URL which is used for better SEO. Just put this website for now

```diff file=src/config.ts
@@ -1,7 +1,7 @@
 export const SITE = {
   website: "https://www.stephenwhitlock.com/",
   author: "Stephen Whitlock",
-  profile: "https://satnaing.dev/",
+  profile: "https://www.stephenwhitlock.com/",
   desc: "A minimal, responsive and SEO-friendly Astro blog theme.",
   title: "AstroPaper",
   ogImage: "astropaper-og.jpg",
```

In Astro Paper, the description (or desc) in config is global site metadata. You'll find this one your RSS feed.

```diff file=src/config.ts
@@ -2,7 +2,7 @@ export const SITE = {
   website: "https://www.stephenwhitlock.com/",
   author: "Stephen Whitlock",
   profile: "https://www.stephenwhitlock.com/",
-  desc: "A minimal, responsive and SEO-friendly Astro blog theme.",
+  desc: "Stephen's Blog",
   title: "AstroPaper",
   ogImage: "astropaper-og.jpg",
   lightAndDarkMode: true,
```

This will show up on your navbar as the title of your site (your site's name)

```diff file=src/config.ts
@@ -3,7 +3,7 @@ export const SITE = {
   author: "Stephen Whitlock",
   profile: "https://www.stephenwhitlock.com/",
   desc: "Stephen's Blog",
-  title: "AstroPaper",
+  title: "Stephen Whitlock",
   ogImage: "astropaper-og.jpg",
   lightAndDarkMode: true,
   postPerIndex: 4,
```

This will point you to the edit screen of your repo on github

```diff file=src/config.ts
@@ -14,7 +14,7 @@ export const SITE = {
   editPost: {
     enabled: true,
     text: "Edit page",
-    url: "https://github.com/satnaing/astro-paper/edit/main/",
+    url: "https://github.com/jshwi/stephenwhitlock/edit/master/",
   },
   dynamicOgImage: true,
   dir: "ltr",
```

In Astro Paper, timezone is purely for date correctness. Without a timezone (or with the wrong one):

- Posts can appear one day early/late

- Sorting can be subtly wrong

- RSS readers may show incorrect publish times

- Midnight posts are the most common victims

```diff file=src/config.ts
@@ -19,5 +19,5 @@ export const SITE = {
   dynamicOgImage: true,
   dir: "ltr", // "rtl" | "auto"
   lang: "en", // html lang code. Set this empty and default will be "en"
-  timezone: "Asia/Bangkok", // Default global timezone (IANA format) https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
+  timezone: "Australia/Sydney",
 } as const;
```

Update socials, if you post on `X` edit that, I don't

```diff file=src/constants.ts
@@ -19,25 +19,19 @@ interface Social {
 export const SOCIALS: Social[] = [
   {
     name: "GitHub",
-    href: "https://github.com/satnaing/astro-paper",
+    href: "https://github.com/jshwi",
     linkTitle: `${SITE.title} on GitHub`,
     icon: IconGitHub,
   },
-  {
-    name: "X",
-    href: "https://x.com/username",
-    linkTitle: `${SITE.title} on X`,
-    icon: IconBrandX,
-  },
   {
     name: "LinkedIn",
-    href: "https://www.linkedin.com/in/username/",
+    href: "https://au.linkedin.com/in/stephen-whitlock/",
     linkTitle: `${SITE.title} on LinkedIn`,
     icon: IconLinkedin,
   },
   {
     name: "Mail",
-    href: "mailto:yourmail@gmail.com",
+    href: "mailto:s.whitlock@live.com",
     linkTitle: `Send an email to ${SITE.title}`,
     icon: IconMail,
   },
```

This is displayed on your home page

```diff file=src/pages/index.astro
@@ -25,7 +25,7 @@ const recentPosts = sortedPosts.filter(({ data }) => !data.featured);
   <main id="main-content" data-layout="index">
     <section id="hero" class="pt-8 pb-6">
       <h1 class="my-4 inline-block text-4xl font-bold sm:my-8 sm:text-5xl">
-        Mingalaba
+        stephenwhitlock.com
       </h1>
       <a
         target="_blank"
@@ -43,19 +43,7 @@ const recentPosts = sortedPosts.filter(({ data }) => !data.featured);
       </a>

       <p>
-        AstroPaper is a minimal, responsive, accessible and SEO-friendly Astro
-        blog theme. This theme follows best practices and provides accessibility
-        out of the box. Light and dark mode are supported by default. Moreover,
-        additional color schemes can also be configured.
-      </p>
-      <p class="mt-2">
-        Read the blog posts or check
-        <LinkButton
-          class="underline decoration-dashed underline-offset-4 hover:text-accent"
-          href="https://github.com/satnaing/astro-paper#readme"
-        >
-          README
-        </LinkButton> for more info.
+        Stephen's Blog
       </p>
       {
         // only display if at least one social link is enabled
```

And your about page

```diff file=src/pages/about.md
@@ -3,35 +3,4 @@ layout: ../layouts/AboutLayout.astro
 title: "About"
 ---

-AstroPaper is a minimal, accessible and SEO-friendly blog theme built with [Astro](https://astro.build/) and [Tailwind CSS](https://tailwindcss.com/).
-
-![Astro Paper](public/astropaper-og.jpg)
-
-AstroPaper provides a solid foundation for blogs, or even portfolios\_ with full markdown support, built-in dark mode, and a clean layout that works out-of-the-box.
-
-The blog posts in this theme also serve as guides, docs or example articles\_ making AstroPaper a flexible starting point for your next content-driven site.
-
-## Features
-
-AstroPaper comes with a set of useful features that make content publishing easy and effective:
-
-- SEO-friendly
-- Fast performance
-- Light & dark mode
-- Highly customizable
-- Organizable blog posts
-- Responsive & accessible
-- Static search with [PageFind](https://pagefind.app/)
-- Automatic social image generation
-
-and so much more.
-
-## Show your support
-
-If you like [AstroPaper](https://github.com/satnaing/astro-paper), consider giving it a star ⭐️.
-
-Found a bug 🐛 or have an improvement ✨ in mind? Feel free to open an [issue](https://github.com/satnaing/astro-paper/issues), submit a [pull request](https://github.com/satnaing/astro-paper/pulls) or start a [discussion](https://github.com/satnaing/astro-paper/discussions).
-
-If you find this theme helpful, you can also [sponsor me on GitHub](https://github.com/sponsors/satnaing) or [buy me a coffee](https://buymeacoffee.com/satnaing) to show your support — every penny counts.
-
-Kyay zuu! 🙏🏼
+Stephen's Blog
```

## Deploy

This will be your deployer so that every push to `master` will update your site (https://docs.astro.build/en/guides/deploy/github/)

```diff file=.github/workflows/deploy.yml
@@ -0,0 +1,42 @@
+name: Deploy to GitHub Pages
+
+on:
+  # Trigger the workflow every time you push to the `main` branch
+  # Using a different branch name? Replace `main` with your branch’s name
+  push:
+    branches: [ master ]
+  # Allows you to run this workflow manually from the Actions tab on GitHub.
+  workflow_dispatch:
+
+# Allow this job to clone the repo and create a page deployment
+permissions:
+  contents: read
+  pages: write
+  id-token: write
+
+jobs:
+  build:
+    runs-on: ubuntu-latest
+    steps:
+      - name: Checkout your repository using git
+        uses: actions/checkout@v5
+      - name: Install, build, and upload your site
+        uses: withastro/action@v5
+        # with:
+          # path: . # The root location of your Astro project inside the repository. (optional)
+          # node-version: 24 # The specific version of Node that should be used to build your site. Defaults to 22. (optional)
+          # package-manager: pnpm@latest # The Node package manager that should be used to install dependencies and build your site. Automatically detected based on your lockfile. (optional)
+          # build-cmd: pnpm run build # The command to run to build your site. Runs the package build script/task by default. (optional)
+        # env:
+          # PUBLIC_POKEAPI: 'https://pokeapi.co/api/v2' # Use single quotation marks for the variable value. (optional)
+
+  deploy:
+    needs: build
+    runs-on: ubuntu-latest
+    environment:
+      name: github-pages
+      url: ${{ steps.deployment.outputs.page_url }}
+    steps:
+      - name: Deploy to GitHub Pages
+        id: deployment
+        uses: actions/deploy-pages@v4
```

And finally, for quality assurance (don't include the node stuff if you use `pnpm`)

```diff file=.github/workflows/ci.yml
@@ -4,12 +4,12 @@ permissions:
   contents: read

 on:
+  push:
+    branches:
+      - master
   pull_request:
-    types:
-      - opened
-      - edited
-      - synchronize
-      - reopened
+    branches:
+      - master
   workflow_call:

 jobs:
@@ -26,25 +26,21 @@ jobs:
       - name: "☁️ Checkout repository"
         uses: actions/checkout@v4

-      - name: "📦 Install pnpm"
-        uses: pnpm/action-setup@v4
-        with:
-          version: 10.11.1
-
-      - name: Use Node.js ${{ matrix.node-version }}
-        uses: actions/setup-node@v4
+      - name: Setup Node
+        id: setup-node
+        uses: actions/setup-node@v6.0.0
         with:
           node-version: ${{ matrix.node-version }}
-          cache: "pnpm"
+          cache: npm

       - name: "📦 Install dependencies"
-        run: pnpm install
+        run: npm install

       - name: "🔎 Lint code"
-        run: pnpm run lint
+        run: npm run lint

       - name: "📝 Checking code format"
-        run: pnpm run format:check
+        run: npm run format:check

       - name: "🚀 Build the project"
-        run: pnpm run build
+        run: npm run build
```
