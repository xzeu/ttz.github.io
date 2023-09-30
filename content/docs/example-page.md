---
weight: 999
title: "Example Page"
description: ""
icon: "article"
date: "2023-09-28T08:19:11+08:00"
lastmod: "2023-09-28T08:19:11+08:00"
draft: true
toc: true
---
# Quickstart

A guide to getting up and running with Lotus Docs.

## Requirements

- git
- Go ≥ v1.19
- Hugo ≥ v0.100.0 (Extended Version)

## Install Hugo

Install the Hugo CLI, using the specific instructions for your operating system below:

```bash
sudo apt install hugo
```

## Manual Installation

The Hugo GitHub repository contains pre-built versions of the Hugo command-line tool for various operating systems, which can be found on the Releases page

For more instruction on installing these releases, refer to Hugo’s documentation

### Create a New Lotus Docs Site

With Hugo installed, create a new Hugo project using the `hugo new` command:

```bash
hugo new site my-docs-site && cd my-docs-site
```

Now initialize your project as a Hugo Module using the `hugo mod init` command:

```bash
hugo mod init my-docs-site
```

You can now choose your preferred method for adding the Lotus Docs theme to your new site from the options below:

```toml
baseURL = 'http://example.org/'
languageCode = 'en-us'
title = 'My New Hugo Site'

[module]
    [[module.imports]]
        path = "github.com/colinwilson/lotusdocs"
        disable = false
    [[module.imports]]
        path = "github.com/gohugoio/hugo-mod-bootstrap-scss/v5"
        disable = false
```

### Create New Content

Navigate to the root of your Hugo project and use the `hugo new` command to create a file in the `content/docs` directory:

```bash
hugo new docs/example-page.md
```

进行编辑
