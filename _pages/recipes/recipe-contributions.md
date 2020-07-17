---
layout: inner
title: Grove Recipe Contributions
lead_text: ''
permalink: /recipes/recipe-contributions/
---

# Grove welcomes recipe contributions

Have you done something you're happy with, or something that's highly reusable that others could benefit from?
Please submit it for inclusion in these docs!

These docs are mainly written in Markdown, with a bit of additional markup and configuration required so it will
be displayed properly and link-able.  The steps are very straightforward and detailed below.  

## How-To

To add a page into the Recipes section, first open `_data/sidenav.yml`.  

Add a new value to the `childPages` array of the JSON object with a value of `text:Recipes`.  For example, 

      { url: '/recipes/awesome-feature/', text: 'My awesome feature' }

can be added to the array:

    - {
      text: 'Recipes',
      childPages: [
        { url: '/recipes/recipe-contributions/', text: 'Add your recipe!' },
        { url: '/recipes/awesome-feature/', text: 'My awesome feature' }
      ]
    }

Next, create the empty file `_pages/recipes/awesome-feature.md`.  The filename should match the `url` you defined.

Lastly, in the markdown file you just created, add the following to the top of the file:
  ```
    ---
    layout: inner
    title: Marianne's Awesome Recipe
    lead_text: ''
    permalink: /recipes/awesome-recipe/
    ---
  ```

Ensure you update the `title` to something meaningful.  The `permalink` must match the `url` you previously defined.

Everything after the second `---` should be written in Markdown.  Feel free to take a look at some of the samples 
in the [React Guide](/guides/react) for inspiration. 

## Submitting a Pull Request

In case you need a referesher on Forking projects on GitHub, we've got you covered.

### Fork Grove

Fork the project [on GitHub](https://github.com/marklogic-community/grove/fork) and clone your copy.

```sh
git clone git@github.com:username/grove.git
cd grove
git remote add upstream git://github.com/marklogic-community/grove.git
```

All documentation changes go into the docs branch. 

### Create a branch for your changes

Okay, so you have decided to add some docs. Create a feature branch
and start hacking:

```sh
git checkout -b my-feature-branch -t origin/docs
```

### Rebase your repo

Use `git rebase` (not `git merge`) to sync your work from time to time.

```sh
git fetch upstream
git rebase upstream/docs
```

### Push your changes

```sh
git push origin my-feature-branch
```

### Submit the pull request

Go to https://github.com/username/grove and select your feature 
branch. Click the 'Pull Request' button and fill out the form.

Pull requests are usually reviewed within a few days. If you get comments
that need to be to addressed, apply your changes in a separate commit and push 
that to your feature branch. Post a comment in the pull request afterwards; 
GitHub does not send out notifications when you add commits to existing pull 
requests.

That's it! Thank you for your contribution!

### After your pull request is merged

After your pull request is merged, you can safely delete your branch and pull the changes
from the main (upstream) repository:

* Delete the remote branch on GitHub either through the GitHub web UI or your local shell as follows:

    ```shell
    git push origin --delete my-feature-branch
    ```

* Check out the develop branch:

    ```shell
    git checkout develop -f
    ```

* Delete the local branch:

    ```shell
    git branch -D my-feature-branch
    ```

* Update your develop with the latest upstream version:

    ```shell
    git pull --ff upstream develop
    ```