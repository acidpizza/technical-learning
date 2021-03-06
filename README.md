# Reference

- [docsy documentation](https://www.docsy.dev/docs/adding-content/content/)
- [supported code languages](https://gohugo.io/content-management/syntax-highlighting/)
- [supported code styles](https://xyproto.github.io/splash/docs/longer/all.html)

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

# CI

```bash
# Get package-lock.json needed for CI
docker run --rm -it -v /home/user/code/technical-learning:/code node:14 /bin/bash
npm i --package-lock-only
```