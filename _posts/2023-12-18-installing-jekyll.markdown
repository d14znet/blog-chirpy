---
layout: post
title:  "Installing Jekyll"
date:   2023-12-18 13:41:43 +0100
categories: jekyll update
---

# JEKYLL


1. Github
  * Create public repo on Github and initialize it with a main branch
  * Create GitHub Page

2. Local Repository
  * Create folder for local repository and clone the contents from the remote repo.

3. Install Jekyll and dependencies  
  * Jekyll is a Gem Package, that's why we need to install Gem (package manager) and then Jekyll (similar to pip/conda)
  * Also, it is recommendable to use an installation directory for the working user (root is not recommended).
      {% highlight bash %}
      sudo apt install ruby-full build-essential zlib1g-dev software-properties-common
      echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
      echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
      echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
      source ~/.bashrc
      gem install jekyll bundler
      {% endhighlight %}
  * If any problems with `make` crash the installation, you can also install Jekyll directly over `apt` with
      {% highlight bash %}
      sudo apt install jekyll
      {% endhighlight %}

4. Initialize Jekyll site
  * On local repo:
      {% highlight bash %}
      jekyll new . #(create new Jekyll site on current directory)
      jekyll new . #--force (if error prompt because folder is not empty)
      {% endhighlight %}
  * New files will be generated, next command can be found on Gemfiles files
      {% highlight bash %}
      bundle exec jekyll serve
      {% endhighlight %}
  * A URL for preview will be generated - if working via SSH, you can tunnel to it:
      {% highlight bash %}
      ssh -p <Port-No.> -L 4000:localhost:4000 user@<ip-of-jekyll-host> -i /path/to/pubkey.pub
      {% endhighlight %}
  * To do changes on the fly, restart bundle with this option (you might not see live changes through SSH-tunnel):
      {% highlight bash %}
      bundle exec jekyll serve --livereload
      {% endhighlight %}

5. Tweak config for GitHub Pages
  * Stop bundle (Ctrl + C), open Gemfiles file
  * Comment `gem "jekyll"`, uncomment `gem "github-pages"`
  * Update gem github-pages: `bundle update github-pages`
  * If not installed, `bundle install` then `bundle update`

**WHAT I DID INSTEAD:**
  * `gem install github-pages`, then `bundle install` then `bundle update`
  * Many dependency failures were displayed, so I installed what was missing - **some errors persisted though**
      {% highlight bash %}
      gem install eventmachine
      gem install http_parser.rb
      gem update
      apt install ruby-http
      gem update
      bundle update
      bundle install
      {% endhighlight %}

6. Commit changes and push to remote repo -> Initialize site
      {% highlight bash %}
      git add ./* #(adds all files on current directory, so make sure you're on the local repo)
      git commit -m "Initialize Jekyll site"
      git push
      {% endhighlight %}
  * Back on GitHub, the site will take some time to render
 
7. Adjust configuration
  * Open _config.yml, save changes, commit and push
      {% highlight bash %}
      base_url: "/repo-name"
      url: "https://username.github.io"
      {% endhighlight %}
  * Refresh URL to see changes - using incognito mode is recommended as **old site might be saved in cache!**
