## Getting started

[quick-start](https://gohugo.io/getting-started/quick-start/)

**References:**
- [beautifulhugo  theme](https://github.com/halogenica/beautifulhugo)

## Add another blog post:

To create a new blog post, run:
```script
hugo new posts/how-to-create-react-app.md
```

## Install notes:

_Install on windows_

```
choco install hugo -confirm
choco install hugo-extended -confirm
```

if you get this after `hugo server`:

```
Error: error building site: assemble: "C:\Users\garra\source\garrardkitchen\blog\content\posts\error-NETSDK1045.md:11:1": failed to extract shortcode: template for shortcode "hint" not found
```

then run this:

```
git submodule update --init --recursive
```