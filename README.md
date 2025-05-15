# Portfolio Website - README

This README documents the quirks, setup steps, deployment process, and other critical notes required to maintain and deploy this portfolio website correctly.

---

## Dependencies & Compatibility Notes

### 1. **Node Version**

- **Use Node.js v18.x**
- Avoid Node 14 or below: causes issues with modern packages like `sharp`, and modules like `node:stream` won’t resolve properly.
- Use [nvm](https://github.com/nvm-sh/nvm) to switch versions:

  ```bash
  nvm install 18
  nvm use 18
  ```

### 2. **NPM Installation Caveats**

- Always install with:

  ```bash
  npm install --legacy-peer-deps
  ```

- Reason: some plugins like `gatsby-plugin-robots-txt@1.8.x` expect `gatsby@^5.0.0`, but this site uses `gatsby@3.15.0`, causing dependency resolution conflicts.

### 3. **Sharp Module Errors**

- If you encounter `Cannot find module '../build/Release/sharp-*.node'`:

  - Delete `node_modules` and `package-lock.json`
  - Reinstall:

    ```bash
    rm -rf node_modules package-lock.json
    npm install --legacy-peer-deps
    ```

- These errors should be already solved, though.

### 4. **Image Optimization Plugins**

- `gatsby-remark-images` warns about deprecated `tracedSVG` option — can be safely ignored or removed in `gatsby-config.js`.
- Basically, ignore all this.

### 5. **Changes to any part**

- Mostly, you'll be just updating the resume. Just make sure the new name of the resume is `cmd + shift + f`ed and updated as well.
- Other content update stuff just change the MD content. Or add, but dont delete.

---

## Local Development Setup

### 1. **Clone & Install**

```bash
// git clone and cd steps..  then use below commands
nvm use 18
npm install --legacy-peer-deps
```

### 2. **Run Locally**

```bash
npm run develop // or npm start as defined in the script in package.json
```

> Note: Stop the local dev server (`Ctrl+C`) before running production builds.

---

## Deployment Workflow

### TL;DR

```bash
git add .
git commit -m "update"
git push
gatsby build
npm run deploy
```

### 1. **gatsby build**

- Builds static files into the `/public` folder
- Files here will be published to the `gh-pages` branch by the next command

### 2. **npm run deploy**

```bash
"deploy": "gatsby build && gh-pages -d public" // in package.json
```

- Automatically builds and pushes contents of `/public` to the `gh-pages` branch.
- Ensure `gh-pages` is configured as the GitHub Pages source under **Settings > Pages**.

---

## Misc Gotchas

### 1. **Do not edit files in `/public` directly**

- They are regenerated each time you run `gatsby build`

### 2. **SVG Logo Pairing**

- `IconHex` and `IconLogo` must visually align
- If replacing, test their dimensions to maintain the correct header layout

---

## Credit

This site was originally forked from [bchiang7/v4](https://github.com/bchiang7/v4) and customized with:

- Custom branding
- Personal project showcase
- Node compatibility fixes
- Domain + GitHub Pages integration

---

## Final Notes

If something breaks in build:

```bash
rm -rf node_modules package-lock.json
npm install --legacy-peer-deps
```
