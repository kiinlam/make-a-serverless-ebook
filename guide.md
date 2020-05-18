# Guide

**Create a Github repo**

 First, create a new repo in your Github account, it can be a private one or public, doesn't matter. Just for the sake of this tutorial, I am going to pick `softwareengineering` as the repo name.

![](https://i.imgur.com/diIHwxE.png)

**Create a new Gitbook project**

 Next, use `gitbook` command line to initialize a new project, name it anyhing. Here I'll use `softwareengineering`, the same one as the git repo name.

After the project setup is finished, try to test it locally.

```text
gitbook init softwareengineering
cd softwareengineering
gitbook serve
```

![](https://i.imgur.com/99Q5kvv.png)

As we can see from image above, the web version of the book is running up.

Next, we are going to use Github Action plugin [peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages) to automate pushing resources from git repo server to the `gh-pages`.

To make this scenario happen, first, generate new key pair using `ssh-keygen` command below. We will use the keys as Github deploy key.

```text
ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f gh-pages -N ""
# You will get 2 files:
#   gh-pages.pub (public key)
#   gh-pages     (private key)
```

The above command generates two files:

* `gh-pages.pub` file as the public key
* `gh-pages` file as the private key

Upload these two files into repo's project keys and secret menu respectively. To do that, open the repo, click **Settings**, then do follow the steps below:

![](https://i.imgur.com/t8RVwN7.png)

**Create Github workflow CI/CD file for generating the web version of the ebook**

\*\*\*\*

Now we are going to make Github able to automatically deploy the web version of the ebook on every push. And we want that to be applied into the first push as well.

Create a new workflow file named `deploy.yml`, place it in `<yourproject>/.github/workflows`, then fill it with the configuration below:

```text
# file ./softwareengineering/.github/workflow/deploy.yml

name: 'deploy website and ebooks'

on:
  push:
    branches:
      - master

jobs:
  job_deploy_website:
    name: 'deploy website'
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1
    - uses: actions/setup-node@v1
      with:
        node-version: '10.x'
    - name: 'Installing gitbook cli'
      run: npm install -g gitbook-cli
    - name: 'Generating distributable files'
      run: |
        gitbook install
        gitbook build
    - uses: peaceiris/actions-gh-pages@v2.5.0
      env:
        ACTIONS_DEPLOY_KEY: ${{ secrets.ACTIONS_DEPLOY_KEY }}
        PUBLISH_BRANCH: gh-pages
        PUBLISH_DIR: ./_book
```

In summary, the workflow above will do these things sequentially:

* Trigger this workflow on every push happens on `master` branch.
* Install `nodejs`.
* Install `gitbook` CLI.
* Build the project.
* use `peaceiris/actions-gh-pages` plugin to deploy the built result to `gh-pages` branch. The Github deploy key that we just uploaded is used by this plugin.

**Push project to Github repo**

```text
cd softwareengineering

# ignore certain directory
touch .gitignore
echo '_book' >> .gitignore

# init git repo
git init
git add .
git commit -m "init"
git remote add origin git@github.com:novalagung/softwareengineering.git

# push
git push origin master
```

Navigate to browser, open your Github repo, click `Actions`, watch a workflow process that currently is running.

![](https://i.imgur.com/SZfwqZs.png)

After the workflow is complete, then try to open in the browser the following URL.

```text
# https://<github-username>.github.io/<repo-name>
https://novalagung.github.io/softwareengineering/
```

![](https://i.imgur.com/HzCygaX.png)

 If you are still not sure about what is the valid URL, open **Settings** menu of your Github repo then scrolls down a little bit until **Github Pages** section appears. The Github Pages URL will appear there.

![](https://i.imgur.com/eD5BmPv.jpg)

**Modify the workflow file to be able to generate the ebook files**

Ok, now we will modify the workflow so it will be able to generate the ebook files \(`.pdf`, `.epub`, and `.mobi`\), not just the web version.

Do open the previous `deploy.yml` file, add a new job called `job_deploy_ebooks`.

```text
# file ./softwareengineering/.github/workflow/deploy.yml

name: 'deploy website and ebooks'

on:
  push:
    branches:
      - master

env:
  ebook_name: 'softwareengineeringtutorial'

jobs:
  job_deploy_website:
    # ...
  job_deploy_ebooks:
    name: 'deploy ebooks'
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1
    - uses: actions/setup-node@v1
      with:
        node-version: '10.x'
    - name: 'Installing gitbook cli'
      run: npm install -g gitbook-cli
    - name: 'Installing calibre'
      run: |
        sudo -v
        wget -nv -O- https://download.calibre-ebook.com/linux-installer.sh | sudo sh /dev/stdin
    - name: 'Preparing for ebooks generations'
      run: |
        gitbook install
        mkdir _book
    - name: 'Generating ebook in pdf'
      run: gitbook pdf ./ ./_book/${{ env.ebook_name }}.pdf
    - name: 'Generating ebook in epub'
      run: gitbook epub ./ ./_book/${{ env.ebook_name }}.epub
    - name: 'Generating ebook in mobi'
      run: gitbook mobi ./ ./_book/${{ env.ebook_name }}.mobi
    - uses: peaceiris/actions-gh-pages@v2.5.0
      env:
        ACTIONS_DEPLOY_KEY: ${{ secrets.ACTIONS_DEPLOY_KEY }}
        PUBLISH_BRANCH: ebooks
        PUBLISH_DIR: ./_book
```

The previous `job_deploy_website` is responsible for generating the web-based version of the ebook. This newly created `job_deploy_ebooks` has different purpose, which is to generate the files version of the ebook \(`.pdf`, `.epub`, `.mobi`\). The generated files later will be pushed to a branch named `ebooks`. The processes will be done by **Calibre**.

Ok, now let's push recent changes into upstream.

```text
git add .
git commit -m "update"
git push origin master
```

![](https://i.imgur.com/iXd7bnr.png)

After the process complete, the ebooks will be available for download in these following URLs. Please adjust it to follow your Github profile and repo name.

```text
https://github.com/novalagung/softwareengineering/raw/ebooks/softwareengineeringtutorial.pdf
https://github.com/novalagung/softwareengineering/raw/ebooks/softwareengineeringtutorial.epub
https://github.com/novalagung/softwareengineering/raw/ebooks/softwareengineeringtutorial.mobi
```

FYI! Since the ebook files are accessible through Github direct link, this means the visibility of the repo needs to be public \(not private\). If you want the repo to be in private but keep the files accessible, then do push the files into `gh-pages` branch.



