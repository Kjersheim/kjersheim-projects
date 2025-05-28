# Redeploying Hugo Site After Making Changes

This process is used when I make changes to the content or layout of the Hugo project and want to regenerate the site and push the updated version to GitHub Pages (kjersheim.github.io).

## Steps:

1. From the root of the Hugo project (kjersheim-projects), run:
   hugo

   This generates the updated static site in the `public/` folder.

2. Change into the public directory:
   cd public

   (Note: This folder is a Git submodule linked to the kjersheim.github.io GitHub repository.)

3. Check for changes:
   git status

4. Stage all updated files:
   git add .

5. Commit the changes with a message:
   git commit -m "fixing images in dsa-app"

6. Push the changes to GitHub Pages:
   git push origin main

After this, the site at https://kjersheim.github.io will be updated shortly with the new content.

## Notes:
- Always run `hugo` **from the main project folder**, not from within `public`.
- The `public` folder must remain committed as a Git submodule.
- If images do not show, check that paths start with `/images/...` and not `images/...`.
