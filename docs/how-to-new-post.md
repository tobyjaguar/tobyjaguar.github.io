# How to Add a New Post

1. Create a markdown file in `content/posts/` named `YYYY-MM-DD-slug.md`:
   ```
   content/posts/2026-04-17-my-new-post.md
   ```

2. Add YAML frontmatter at the top:
   ```yaml
   ---
   title: "My New Post"
   date: 2026-04-17
   ---

   Your content here...
   ```

3. If you have images, put them in `static/assets/posts/<slug>/` and reference them as:
   ```markdown
   ![alt text](/assets/posts/my-new-post/image.jpg)
   ```

4. Preview locally:
   ```bash
   hugo server -D
   ```

5. Commit and push — the site updates automatically.
