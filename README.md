# Reference

- https://www.docsy.dev/docs/adding-content/content/


# Initial Setup

```bash
hugo new site technical-learning
cd technical-learning/
git init
git submodule add https://github.com/google/docsy.git themes/docsy
echo 'theme = "docsy"' >> config.toml
git submodule update --init --recursive

hugo server
```

# Subsequent setup

```bash
git clone <project>
git submodule update --recursive

hugo server
```