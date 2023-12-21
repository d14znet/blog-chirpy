---
layout: post
title:  "Setting up a Jekyll Blog with Chirpy Theme"
date:   2023-12-21 19:33:43 +0100
categories: how-to jekyll
tags: ["raspberry pi", "how to", "blog", "jekyll"]
---

# Setting up a Jekyll Blog with Chirpy Theme

For months I was looking for a way to collect and centralize all the technical documentation that I've been gathering during my vocational training on IT.
Apart from that, I wanted to start doing projects on my own to gain more knowledge about topics that I'm interested in - networking, containers, server setup and automatization.
As a way to document my progress in projects, gather instructions and technical docs, I decided to set up an easy blog.

After doing some research I became interested in Jekyll, which is a Ruby-based framework to serve blogs. It seemed pretty easy to setup and configure, giving me the possibility to self-host and serve my content to the Internet, or to combine it with GitHub Pages.

I will be using the later for now - GitHub Pages offers hosting and also automatized actions to build and serve Jekyll content. Self-hosting might be a nice project for later on.

Let's see if this project lives up to my expectations and the blog serves as a collection point for my projects' progress.


## Disclaimer & Credits

> This blog wouldn't exist without the help provided by these tutorials:
>
> * Bill Raymond - [Up and Running with GitHub Pages, Part 3, Blogging with Jekyll](https://www.youtube.com/watch?v=EmSrQCDsMv4)
> * Jekyll's Official Documentation - [Installation Guide](https://jekyllrb.com/docs/installation/)
> * Cotes Chung - [Getting Started with Chirpy Theme](https://chirpy.cotes.page/posts/getting-started/)
>


## Instructions

### Create and initialize GitHub Repo
* Create **public repo** on Github and initialize it with a main branch
* Create GitHub Page: Click on Settings and then Pages (on the left menu).
  * Under **Build and deployment**, choose GitHub Actions as Source.
* On your working host, create a folder for a local repository and clone the contents from the remote repo.

### Install Jekyll and its dependencies  
Jekyll is a **Gem Package**, that's why we need to install Gem (package manager) and then Jekyll (similar to `pip`/`conda` on Python).
Following the instructions on [Jekyll's Official Documentation](https://jekyllrb.com/docs/installation/other-linux/), it is recommendable to use an installation directory for the working user (root is not recommended).
```console
sudo apt install ruby-full build-essential zlib1g-dev software-properties-common
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Afterwards you should `exit` the session and login again so that the changes to the .bashrc take place. Run then `gem install jekyll bundler`

#### Troubleshooting
I had some trouble at this point, since my system wasn't able to install all dependencies and Jekyll couldn't be installed that way. It turns out you can also install Jekyll via Package Manager: `sudo apt install jekyll`.

### Initialize Jekyll site
* On your local repo, run:
```console
jekyll new . #(this creates a new Jekyll site on current directory)
jekyll new . --force #(if the directory is not empty, Jekyll will throw an error - use then the --force option)
```
* Jekyll will start generating files. To be able to browse your site, you have to serve it using `bundle exec jekyll serve`. You can find this command under `Gemfiles` textfile.
* Your command prompt will generate an URL for preview - `http://127.0.0.1:4000/`. If you're working via SSH as I was, you can use a local tunnel to preview your content. Once the tunnel is established, open `http://localhost:4000/` on your local browser.
```console
ssh -p <Port-No.> -L 4000:localhost:4000 user@<ip-of-jekyll-host>
```
* If you want to see your changes live (without having to reload the page every time), restart the bundle with `bundle exec jekyll serve --livereload` - **Note**: This doesn't work if you set up a SSH tunnel. I guess it will work if you set up 2 tunnels, as Jekyll uses an extra port to serve live content.

### Set up GitHub Pages config
* Stop bundle (Ctrl + C), open `Gemfiles` file
* Comment `gem "jekyll"`, uncomment `gem "github-pages"`
* Update gem github-pages: `bundle update github-pages`
* If needed, install bundle updates: `bundle install` then `bundle update`

#### Troubleshooting
This is again a point where I stuck for a while, as my system wasn't once again able to fullfill dependencies.
I looked at the errors carefully and tried to manually install all gems that were missing.
Some error messages persisted though, but I was able to tweak my Jekyll config to work with GitHub Actions.
```console
gem install github-pages
bundle install
bundle update
# Install will fail listing which gems were missing - I tried to install them manually:
gem install eventmachine
gem install http_parser.rb
gem update
apt install ruby-http
gem update
bundle update
bundle install
```

### Commit changes and push to remote repo
Once you have commited and push changes, GitHub will need some time to render your site live.
At this point, you will be able to see your blog at your GitHub Pages URL (`username.github.io/repo-name`).
```console
git add ./* #(adds all files on current directory, so make sure you're on the local repo)
git commit -m "Initialize Jekyll site"
git push
```
  
### Adjust Jekyll configuration
* Open `_config.yml` and adjust the following, then commit and push to remote repo:
```console
base_url: "/repo-name"
url: "https://username.github.io"
```
* Refresh URL on your browser to see changes - using incognito mode is recommended as **old site might be saved in cache!**
  
### Create & publish first post
* You will find an example post under `_posts/` on your local repository. You can use it as template to start writing your first posts. Remember to rename the file as recommended on the *Welcome to Jekyll* post - ``YYYY-MM-DD-your-post-name.markdown`
* Every time you run `bundle exec jekyll serve --livereload`, Jekyll will render the markdown language to display HTML.

**IMPORTANT**: As we tweaked Jekyll's configuration to make it compatible with GitHub Pages, you might have problems when trying to preview the site on your local machine. If this happens, simply revert the steps from "Set up GitHub Pages config" and make sure to add the `Gemfiles` file to .gitignore.


## Adding a Theme

With everything set up and running, only a nice theme was missing - I particularly liked the Chirpy Theme a lot for its compact design and cleanliness.
To install it, I followed the instructions from Cotes Chung's Site, [Getting Started with Chirpy Theme](https://chirpy.cotes.page/posts/getting-started/).
I went for the **Chirpy Starter** method - even though I already had set up a repository for my blog, I created a new one and imported the config, Gemfile and posts from the prior repo. 
I preferred keeping things separated, since I might probably break a lot of things while learning :) this way I might reckon faster what went wrong and will be able to restore things faster.

The result of Cotes Chung's documentation is the blog you're currently reading, so I didn't break that many things in the end.


## To-Dos

At the time you're reading this, all these features/changes need to be done yet:

* Set up the `about` site.
* Figure out how to upload content and use it on posts (images).
* I also have no clue how the tags actually work.
