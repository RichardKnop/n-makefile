# n-makefile

## Installing

```bash
LATEST=$(curl -s https://api.github.com/repos/Financial-Times/n-makefile/tags | grep name | head -n 1 | sed 's/[," ]//g' | cut -d ':' -f 2)
curl -sL https://raw.githubusercontent.com/Financial-Times/n-makefile/$LATEST/Makefile > n.Makefile
echo "include n.Makefile" > Makefile
git add Makefile n.Makefile
git commit -m "Add build tools at version $LATEST"
git push
```

## Introduction

### Problems trying to solve over NBT (and OBT)

- Enormous dependency tree.  The `npm install` part of builds are often the slowest thing.  It's crazy that when you're building apps that don't and can never have front ends (APIs, Lambda functions) you still need to pull in webpack, Sass, etc just to get the toolchain.
- Everyone's code editors would be much happier if the dotfiles were in the place those expect them to be.  `n-makefile` puts `.editorconfig`, `scss-lint.yml`, etc in the root directory of projects.
- `make install` was dependent on `make install` having run before.  (`obt install` required `obt` to be there).
- Try to eliminate the need to add the custom `:node_modules/.bin` to everyone's `$PATH`.

### What this is not

- A complete rewrite in bash/Makefile.  The ‘heavy lifting’ should still be done by npm modules.  There'll probably be a bit **too** much implemented in bash/Makefile initially whilst we figure out exactly where to draw the line in what logic should be controlled by bash/Makefile and what should be handled by the npm libraries.

## Philosophy

- Always **default to doing nothing**.
- Each feature should work in **isolation**.
- Rely on **signals** and **intelligently infer** what to do.
- The developer can **always override**:1 anything.
- Unused features must not slow things down.

### Install

`make install` may pull in two types of thing.  Packages and dot files.

#### Packages

- Only try to install npm modules if there's a `package.json` file.
- Only try to install bower components if there's a `bower.json` file.
- Only try to install the `scss-lint` Ruby Gem if it's not already installed and if there are actually `*.scss` files in the project.

(We hope to get rid of scss-lint as soon as the Node port gains feature parity)

#### Dot files

- By default, no nothing.
- If `.env`, `.editorconfig`, `.scss-lint.yml`, or `.eslintrc.json` are commited to the repository don't overwrite them (default behaviour in Makefile)
- If `.env`, `.editorconfig`, `.scss-lint.yml`, or `.eslintrc.json` are not commited but **are listed in the `.gitignore` file** download them.

### Verify

- Only run the verify step **if** the relevant dot file exists.  E.g. only run `eslint` if `.eslintrc.json` exists.
- Run the linting tool against an appropriate subset of the files committed to the project.