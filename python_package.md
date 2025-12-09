# Create and Publish a Python Package

## 1. Create the Package Template

```bash
uvx cookiecutter gh:scientific-python/cookie
cd PACKAGE-NAME
```

## 2. Initialize Git and Push to GitHub

```bash
git init
git add .
git commit -m "feat: initial commit"

gh auth login
gh repo create
git push --set-upstream origin main
```

## 3. Configure TestPyPI Publishing

On https://test.pypi.org → **Publishing**:

- **Provider:** GitHub\
- **PyPI project name:** \[PACKAGE-NAME\]\
- **Owner:** USERNAME\
- **Repository name:** \[PACKAGE-NAME\]\
- **Workflow name/path:** cd.yml\
- **Environment name:** pypi

## 4. Create a Release

```bash
git tag -a v0.1.0 -m "v0.1.0"
gh release create v0.1.0 -t "v0.1.0" -n "First TestPyPI release"
```

## 5. Set Up Documentation with Read the Docs

On https://app.readthedocs.org/dashboard/ → **Add project**:

- **Repository:** USERNAME/PACKAGE-NAME

This automatically triggers a documentation build.

## 6. Set Up Code Coverage with Codecov

On https://app.codecov.io/:

1.  Configure the repository: `USERNAME/PACKAGE-NAME`
2.  Copy the **CODECOV_TOKEN**
3.  In GitHub:\
    **Settings → Secrets and variables → Actions → New repository
    secret**
    - **Name:** `CODECOV_TOKEN`\
    - **Value:** the copied token

Add these lines to your `README.md`:

```markdown
[![Coverage][coverage-badge]][coverage-link]
[coverage-badge]: https://codecov.io/gh/USERNAME/PACKAGE_NAME/branch/main/graph/badge.svg
[coverage-link]: https://codecov.io/gh/USERNAME/PACKAGE_NAME
```

## 7. Add discussion feature for the repository

Enable the **Discussions** feature in the repository settings.
