+++
title = "Making a Website with Zola, Github Pages, and Github Actions"
date = 2022-02-26

[taxonomies]
categories = ["Meta"]
tags = ["zola", "github actions", "website"]
+++
Making a website in the modern era is not easy to do for free. I did it using Zola, Github Pages, and Github Actions.
<!-- more -->
I've always wanted to have a personal website where I can upload what I do on my local computer to access remotely and have the world see. But it always seemed like too much of a hassle to set up and I didn't have the capital to invest in a website that I didn't need. However, when I started going back to university in-person this year, I found it much easier to take notes by typing them instead of using OneNote as I was accustomed, as the amount of code I had to write was drastically increased. Needing a way to access them remotely with a nice view, I thought a blog would be a good way to do that in addition to all the other things I had always wanted a website for. So, I embarked on a journey to create a website.

## Github Pages

A website is no use unless we have somewhere to put it. [Github Pages](https://pages.github.com/) is a service offered by Github since 2008 that allows you to host your own website from a Github repository. You get one free website per Github account, which is called [username].github.io. All we have to do to enable it is create a repository named [username].github.io and enable Github Pages in the settings!

```bash
# should be above 2.28 to enable default branch name change
git --version

mkdir [username].github.io
cd [username].github.io

# personal git config
git config --global user.name "NAME"
git config --global user.email "EMAIL"
git config --global init.defaultBranch "main"
git init

gh repo create [username].github.io --public --source=. --remote-upstream
```

If you're using [Visual Studio Code](https://code.visualstudio.com/) as your editor, there's a nicer way to do this than through the command line. After installing [the Github extension](https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-pull-request-github), go to the Source Control button on the sidebar. There should be a button labeled "Publish to Github" which allows you to interactively initialize a Git repository in the current folder and publish it to Github.

## Github Actions

We might have created our Github page, but we need a way to get all of the code from our repository to the website. This is called **deployment**. Luckily, we have a way to automatically deploy our website through **Github Actions**. [Github Actions](https://github.com/features/actions) is another service offered by Github since 2019 that gives you free CI in public repositories. Although the free tier is [fairly limited](https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions#included-storage-and-minutes) at 500 MB and 2000 minutes per month, it should be more than enough for a static blog that is not deployed very often.

There is an action automatically created for our Github page called `pages-build-deployment` which, as the name implies, builds and deploys the page you've created on push. The way that I organized my code, which is probably the simplest option, is that I hosted my code at the `main` branch and had a `gh-pages` branch that hosted the actual website which was built from the `main` branch. If you want to do the same, go to `Settings` -> `Pages` and make sure that the build target is the `gh-pages` branch at the root.

For now, this won't do anything because we don't have a `gh-pages` branch or anything in our `main` branch. So how do we *actually* make our website?

## Zola

[Zola](https://www.getzola.org/) is a static site generator written in [Rust](https://www.rust-lang.org/), and is one of the fastest out there. I decided to choose it for my website. If you'd prefer a different generator, this is where this guide diverges for you. There are plenty of tutorials for Hugo websites or others, but I have found a lack of Zola guides so I decided to create this.

To start, [install Zola on your system](https://www.getzola.org/documentation/getting-started/installation/). The documentation on the website is pretty stellar so I would recommend reading that to get a quick understanding on how to use Zola. I will explain the parts that I personally found difficult or unclear.

```bash
zola init
zola build # unnecessary as serve will also automatically build it
zola serve
```

Now you can see your new website at `127.0.0.1:1111`!

## Zola Deployment

Although we can build your website very simply on our local machine, it would be preferable to automatically build the website when we publish content so we don't have to mess around with all of that. The [Zola-approved way](https://www.getzola.org/documentation/deployment/github-pages/) to do this is by using [zola-deploy-action](https://github.com/shalzz/zola-deploy-action). All of you have to do is click the `New Workflow` button on the `Actions` page from your Github repository and follow the link to `set up a workflow yourself`, then copy-paste this into it:

```yaml
# On every push this script is executed
on: push
name: Build and deploy GH Pages
jobs:
  build:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: checkout
        uses: actions/checkout@v2
      - name: build_and_deploy
        uses: shalzz/zola-deploy-action@v0.14.1
        env:
          # Target branch
          PAGES_BRANCH: gh-pages
          # Provide personal access token
          TOKEN: ${{ secrets.TOKEN }}
```

However, I wanted more configuration and control over my action. Github Actions operates using Jekyll by default, so unless you add a `.nojekyll` file to the build branch it will run unnecessary steps to build a Jekyll theme. In order to reduce complexity, I decided to make a similar action that adds that command. If you're comfortable with the workflow as provided, then skip the next section.

## Creating a Github Action

An action of the type we want here consists of three files: `action.yaml`, `Dockerfile`, and `entrypoint.sh`. Let's break these down.

- `action.yaml`: the configuration file that tells Github Actions what to do
- `Dockerfile`: the configuration file for the Docker container that Github Actions will set up
- `entrypoint.sh`: the shell script that will execute the commands we want

`action.yaml` is simple. It should follow this general format:

```yaml
# action.yaml
name: 'ACTION_NAME'
description: 'DESC'
author: 'NAME'
runs:
  using: 'docker'
  image: 'Dockerfile'
```

The Dockerfile is more complex and has many more options. I kept mine simple to what is needed, you may have your own preference for Docker iamges.

```Dockerfile
# any Docker image is fine, I prefer debian
from debian:stable-slim
MAINTAINER NAME <EMAIL>

# for github actions
LABEL "com.github.actions.name"="ACTION_NAME"
LABEL "com.github.actions.description"="DESC"

# locale, I am in the U.S. so I use en_US
ENV LC_ALL C.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8

# standard apt-get + wget and git for getting and building
RUN apt-get update && apt-get install -y wget git

# get zola on the docker image
RUN wget -q -O - \
"https://github.com/getzola/zola/releases/download/v0.15.3/zola-v0.15.3-x86_64-unknown-linux-gnu.tar.gz" \
| tar xzf - -C /usr/local/bin

COPY entrypoint.sh /entrypoint.sh

# give the entrypoint executable permissions
RUN chmod +x entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```

`entrypoint.sh` is where all the magic happens. Fundamentally, all that it does is call `zola build` on the `main` branch, which builds your website inside the `public` directory (you can configure this if you so wish). It then commits those website files to the `gh-pages` branch.

We need one more thing before we can create `entrypoint.sh`; a token. We need to authorize our action to be able to push to `gh-pages`. You can create a token by going to [this page](https://github.com/settings/tokens) and creating a new token with at least `repo` rights. You can add the token to your repository by going to `Settings` -> `Secrets` -> `Actions` and creating a new repository secret called `TOKEN` (or any other name you like).

With that token, this is the basic necessities for `entrypoint.sh`:

```bash
#!/bin/bash
set -e
set -o pipefail

main() {
    git config --global url."https://".insteadOf git://
    git config --global url."$GITHUB_SERVER_URL/".insteadOf "git@github.com":

    # update git submodules (important if you have themes)
    git submodule update --init --recursive

    zola build

    cd public

    # if you want to add any commands do it here e.g. `touch .nojekyll`

    git init
    git config user.name "GitHub Actions"
    git config user.email "github-actions-bot@users.noreply.github.com"
    git add .

    git commit -m "Deploy ${GITHUB_REPOSITORY} to ${GITHUB_REPOSITORY}:gh-pages"
    git push --force "https://${GITHUB_ACTOR}:${TOKEN}@github.com/${GITHUB_REPOSITORY}.git" master:gh-pages
}

main "$@"
```

With all of these settings, this is how your workflow should look:

```yaml
# .github/workflows/main.yml
on: push
name: Build and deploy GH Pages
jobs:
  build:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: checkout
        uses: actions/checkout@v2
      - name: build-and-deploy
        uses: ./ # wherever your action is in relation to the root of the repo
        env:
          TOKEN: ${{secrets.TOKEN}}
```

## Themes

Zola requires a [Tera](https://tera.netlify.app/) template to render your site for the base site `index.html` as well as `page.html` for page-specific settings. You can also use [Sass](https://sass-lang.com/) stylesheets if you enable it in your `config.toml`. **Themes** are a convenient way to have those built for you so you can get a website looking nice without excessive fiddling. I decided to use the [after-dark](https://github.com/getzola/after-dark) theme which is based on the Hugo theme of the same name. Since I wanted to make my own modifications to it, [I forked it](https://github.com/sharifhsn/after-dark) and added the changes I wanted. The easiest way to add a theme for Github pages is to use submodules:

```bash
git submodule add https://github.com/getzola/after-dark.git themes/after-dark
```

The deploy action will take care of updating the submodule as necessary. Just add the theme to your `config.toml` file and voila!

Although I like the `after-dark` theme, I might change to a different theme or make my own in the future to accommodate my goals for this website. If that happens, I'll detail that process in another post.

## $\KaTeX$

Most of the lecture notes I write incorporate [$\KaTeX$](https://katex.org/) in some way. I find it an expressive way to write formulas and math expressions when reviewing for tests. `after-dark` does not provide $\KaTeX$ support by default, which is part of the reason I forked it.

My preferred option for $\KaTeX$ rendering would be server-side, as I don't plan on pushing very often (perhaps once per day) and Zola compilation is extremely quick. However, after doing some research into [previous attempts](https://github.com/getzola/zola/pull/1073), I decided it wasn't feasible for now. Perhaps in the future I'll take a stab at implementing it myself, but for now I'll settle for client-side.

[The $\KaTeX$ docs](https://katex.org/docs/browser.html) give a pretty good description on how to incorporate it into your website. In Tera, all you have to do is enclose those stylesheets/scripts into CSS/JS blocks, respectively. My inspiration came from [this pull request](https://github.com/getzola/after-dark/pull/22). I modified the standard `auto-render.min.js` script to add standard $\KaTeX$ \$ \$ tags.
