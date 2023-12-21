---
layout: post
title:  "Testing out Chirpy"
date:   2023-12-19 20:42:43 +0100
categories: jekyll update
---

# Testing Post layouts

I want to test if Chirpy automatically creates a content table

## Chapter 1

Some text

1. Element one
2. Element two
3. Element three
  * Subelement 1
  * Subelement 2

## Chapter 2

This has 3 sub-elements

### Chapter 2.1

Some text, and some `in-line code`.

### Chapter 2.2

Some text, and now a code block follows:

{% highlight bash %}
sudo apt install ruby-full build-essential zlib1g-dev software-properties-common
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
gem install jekyll bundler
{% endhighlight %}

### Chapter 2.3

Now some *text* with **more formatting** ~strikethroughs~ and _no idea what this actually did tbh_.

- Unordered list 1
- Unordered list 2
  - Subelement 2.1
  - Subelement 2.2
- Unordered list 3

## Chapter 3

Some more code blocks:

```console
var="Hello"
echo $var
```

I think this is enough.

## Chapter 4

> But I also want to try out callout blocks.
> No idea if it work this way or now.
> Let's build to see.

# THE END

Bye
