# Introduction

## CI/CD - Serverless Ebook using Gitbook CLI, Github Pages, Github Actions CI/CD, and Calibre <a id="cicd---serverless-ebook-using-gitbook-cli-github-pages-github-actions-cicd-and-calibre"></a>

 In this tutorial we are going to create an ebook instance using Github, then publish it to the Github pages in an automated manner \(on every push to upstream\) managed by Github Actions, and it will not deploy only the web version, but the ebook files as wall \(in `.pdf`, `.epub`, and `.mobi` format\).

 For every incoming push to the upstream, Github Actions \(CI/CD\) will trigger certain processes \(like compiling and generating the ebook\), then the result will be pushed to the `gh-pages` branch, make it publicly accessible.

