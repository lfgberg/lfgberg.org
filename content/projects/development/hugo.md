---
title: "Creating a Hugo Site"
description: "The process I used to create my portfolio site"
date: 2023-08-29
categories: ["Development"]
tags: ["Web", "Hugo"]
---
This post will serve as a brief overview of why I chose to create a portfolio site, and how to create your own with zero hardware and a relatively low barrier to entry. I created my site using [Hugo](https://gohugo.io/), and the [Blowfish Theme](https://blowfish.page/). I'll be providing any resources I utilized, but will not be providing step-by-step instructions on configuration. My end goal with this project was to have a centralized platform to stand out when applying to jobs and internships by showcasing my projects and interests as well as coursework. As a student, this is an extremely helpful tool to show initiative and commitment to the field outside of the classroom, and I've found it to be an engaging low-stress project to work on.

## Goals

When choosing my tech stack for this portfolio I kept the above goals in mind:

1. No hardware required
2. Low to no cost
3. Simple formatting

## What and Why?

- Hugo
  - Hugo is a flexible framework for building websites
  - Utilizes markdown syntax, no web design skills needed
  - Easy to maintain
  - Has robust options for themes and shortcodes
  - Supported by GitHub and Cloudflare Pages
- Cloudflare Pages
  - Free environment to host and deploy a site
  - Syncs to a GitHub repo and auto-deploys
  - Works well with my domain already being managed by Cloudflare

## Setting Up The Site

### Buying a Domain (optional)

The only part of this project with any cost attached is purchasing a domain, I chose to purchase my domain from Cloudflare because I wanted the additional flexibility it could provide to build out things like custom email, etc. This project could easily be made free by omitting this step and using GitHub pages to create the entire thing for free. Feel free to read more about it [here](https://pages.github.com/).

### Setting up a Development Environment

[Install Hugo](https://gohugo.io/installation/) on your local machine, and use `hugo new site` to initialize a new project. Create a GitHub repository to use when developing the portfolio site. If utilizing GitHub pages, name the repository `username.github.io`, otherwise navigate to your Cloudflare Dashboard, and [set up a new page](https://developers.cloudflare.com/pages/framework-guides/deploy-a-hugo-site/) linked to your repository. Changes will be automatically deployed to your site as you commit them to the specified branch.

### Developing Content

To develop content for your site you'll first want to add a theme, this will do a lot of the leg work of creating the site for you. I chose to use the [Blowfish Theme](https://blowfish.page/docs/installation/), you can add this to your project by adding it as a git submodule in the themes folder with `git submodule add -b main https://github.com/nunocoracao/blowfish.git themes/blowfish`. At this point, you'll have to set up the configuration files for your site which can differ depending on your theme. The documentation for Blowfish is [here](https://blowfish.page/docs/installation/#set-up-theme-configuration-files). At this point you're ready to create content for your site with Hugo's markdown syntax, and regularly update it with new information.
