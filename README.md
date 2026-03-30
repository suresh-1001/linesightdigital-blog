# LineSight Digital Blog

Hugo-powered blog at [blog.linesightdigital.com](https://blog.linesightdigital.com)

## Adding a New Post

```bash
hugo new posts/your-post-slug.md
```

Then edit `content/posts/your-post-slug.md`. Set `draft: false` when ready to publish.

## Adding a Post Image

1. Add image to `static/images/` (PNG or JPG, ~1200×630px recommended)
2. Reference in front matter: `image: "/images/your-image.png"`

## Local Development

```bash
hugo server -D
# Visit http://localhost:1313
```

## Deploy

Push to `main` branch — GitHub Actions builds and deploys automatically.

## Structure

```
content/posts/     ← blog posts (markdown)
static/images/     ← post images
assets/css/        ← stylesheet
layouts/           ← HTML templates
```
