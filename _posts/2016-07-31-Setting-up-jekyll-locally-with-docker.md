---
title: Setting up Jekkyl in Docker on Windows
layout: post
comments: true
---

As you may have noticed, I have started a blog (again). This time I decided to go
with jekyll. In this article I'm explaining how to run jekyll on Windows using Docker
and why you might want to do it this way.

If you're running different platform, you will find it useful too.

<!--MORE-->

## What and why?

Jekyll is static-site generator - it will generate a static site using some clever
text transformation and templating on your source files. Why use static site anyway,
you might ask. There are many answers, but for me it's simplicity, security and
an opportunity to learn a popular tool.

Jekyll is written in ruby and requires ruby to run. You can install ruby
on your machine to play with jekyll and your site, but that's not the path I've chosen.
I decided to go with Docker.

## Why Docker?

Docker is a container platform, that allows you to run pieces of software in clean,
isolated and consistent micro-environemnts. If you don't know what it is, you miss
a big time and I truly encourage you to check it out and get an opinion.

I decided to run jekyll using Docker because that's a perfect solution to keep
my machine clean. I don't have to install ruby on my box, I don't need to
worry about dependencies and maybe ruby-versions in the future. I have a requirements
that Dockers was created for and it would be uniwse not to take advantage of that.
Plus I've never used Docker before and it's always great to learn new things.

## Guide

Okay, now to the point. So what I currently have is clean Windows 10 box with
Docker installed. You should get yourself a Docker installation to and I'm not
explaining how - it's easy and cross-platform, so just follow the Docker's page
and [get yourself a Docker installation](http://www.docker.com/products/overview).

### Running jekyll

I didn't know how to use jekyll before so I went to 
[jekyll quickstart](https://jekyllrb.com/docs/quickstart/) and got myself
up to date on the matter. The most important findings were:

* requires ruby
* there is a command line tool `jekyll`
* I'd like to start with `jekyll new myblog`

Since I didn't want to install ruby on my machine, I decided to use Docker.
The first obvious thing was to google how to use jekyll in docker and
no wonder there already was an
[article about this](https://github.com/jekyll/docker/wiki/Usage:-Running)
on jekyll's wiki page.

So let's try that. I'm using PowerShell console for all command line stuff.
I'm also running native Docker, so my command to run is:

{% highlight powershell %}
  > docker run --rm --label=jekyll --volume=$(pwd):/srv/jekyll -it -p 127.0.0.1:4000:4000 jekyll/jekyll
{% endhighlight %}

What that does is:

* start a new container using image jekyll/jekyll
* bind current working directory to `/srv/jekyll` inside container
* bind localhost:4000 to port 4000 inside container
* delete the container after it's ran (`--rm` switch does this)

Try that out:

    Unable to find image 'jekyll/jekyll:latest' locally
    latest: Pulling from jekyll/jekyll

    ce1e95d87833: Pull complete
    8cfe8fca8736: Pull complete
    060434a1fb0d: Pull complete
    Digest: sha256:c552e5a74a9a6f233a6523b4488c77c53acb0b0d424e878f305ae06e45f4dece
    Status: Downloaded newer image for jekyll/jekyll:latest

    Configuration file: none
                Source: /srv/jekyll
           Destination: /srv/jekyll/_site
     Incremental build: disabled. Enable with --incremental
          Generating...
                        done in 0.012 seconds.
     Auto-regeneration: enabled for '/srv/jekyll'
    Configuration file: none
        Server address: http://0.0.0.0:4000/
      Server running... press ctrl-c to stop.


And it works! I can navigate to http://localhost:4000/ and see... blank page.
That's ok since I have started the server in some empty directory and default
action that this container runs is `jekyll serve` - serve the page over http.

To create
a simple site structure i need to run `jekyll new myblog`. But since I'm using
Docker it gets a little bit longer, because I need to prefix it with the
"docker starter":

{% highlight powershell %}
  > docker run --rm --label=jekyll --volume=$(pwd):/srv/jekyll -it -p 127.0.0.1:4000:4000 jekyll/jekyll jekyll new myblog
{% endhighlight %}

### Making that easy

That's lengthy and I'm lazy, so I naturally want to shorten this. Little bit of
googling more and I know I need to create a powershell function for this. To make
it available for me always I add it to my PowerShell_profile.ps1, which get's loaded
every time I open console (make sure that your execution policy allows for that).
The file is located here:

And the function I added at the end of that file:

{% highlight powershell %}

function jekyll{
  docker run --rm --label=jekyll --volume=$(pwd):/srv/jekyll -it -p 127.0.0.1:4000:4000 jekyll/jekyll jekyll $args
} 

{% endhighlight %}

This will create powershell function `jekyll` that will run the jekyll inside
the container with any arguments you provide to the function. Basically it's
like I had that jekyll locally. Magic!

    > jekyll new myblog
    New jekyll site installed in /srv/jekyll/myblog.
    > dir
    
    
        Directory: C:\Projects\jekyll-start
    
    
    Mode                LastWriteTime         Length Name
    ----                -------------         ------ ----
    d-----       25.07.2016     22:53                myblog


Great! Now just run `jekyll serve` and go to the browser.

![Jekyll site opened in a browser](/assets/images/20160731_jekyll_ready.png)

### Problem with auto refresh

Jekyll's `serve` command should automatically refresh your
site whenevere there is change made to any of your files,
so you can just refresh browser and review your changes immediately.

Well, it stompted me two times:

1. It didn't refresh by itself because of Docker/Windows issue. Fixed that.
1. It didn't refresh for `_config.yml`. Didn't fixed that, it works this way,
you have to live with that and restart after each change to config file. No problem
if you know it.  

As for the first issue it is document on
[Running jekyll in docker](https://github.com/jekyll/docker/wiki/Usage:-Running)
wiki page. Files are on Windows, container runs in Linux VM and can't
be notified of NTFS files changes properly, so you have to resort to
polling for changes. Jekyll support's that and the easiest way for me
was to set `POLLING` environmental variable to `true`. I did that by
changing my function definition:

{% highlight powershell %}

function jekyll{
  docker run --rm --label=jekyll --volume=$(pwd):/srv/jekyll -it -p 127.0.0.1:4000:4000 -e POLLING=true jekyll/jekyll jekyll $args
} 

{% endhighlight %}

And now it's working again and I can forget about it, except that
`_config.yml` thing.

## Follow-up

I wish to host my site on github pages and there are two ways to do it:

* either let the github to run jekyll for you,
* or run jekyll for yourself and push the built site to github.

I will elaborate on this next time.

