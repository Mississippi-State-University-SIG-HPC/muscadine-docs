# Getting Started

This template provides a ready-to-use documentation setup using **MyST-flavored Markdown** and GitLab Pages.

## Create a New Repository

1. Navigate to **GitLab → New Project → Create from template → Instance**
2. Select **this template**  
3. Name your repository and create it

## Populate Your Documentation

Place your documentation files under the `docs/` directory.

**NOTE:**

For details on syntax features such as directives, roles, and math support, see the [MyST Syntax Guide](https://mystmd.org/guide/quickstart-myst-markdown)

## Build and Preview Locally

You can build and preview your docs before pushing changes:

```bash
make html
python3 -m http.server -d _build/html
```

Then open your browser to [http://localhost:8000](http://localhost:8000).

## Automatic Build and Deployment

When you push updates to your repository:

* The GitLab CI/CD pipeline will automatically build your documentation
* The site will be published via **GitLab Pages** at

  ```
  https://<group>.gitlab.io/<project>
  ```
