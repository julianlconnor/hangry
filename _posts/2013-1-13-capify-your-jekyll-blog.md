---
layout: post
title: Capify Your Jekyll Blog
---

In this tutorial I am going to show you how to use [Capistrano](https://github.com/capistrano/capistrano) to deploy your Jekyll blog to an AWS instance (or any server for that matter). I realized that I wanted full control of my blog (none of this CMS nonsense) and decided that a flow where I could edit posts locally, push them to github, and then use Capistrano to deploy these changes to the "`cloud`" would be pretty cool.. so I did it. ;)

In this tutorial I will cover:

* Installing Capistrano
* Configuring Capistrano
* Deploying Your Blog via Capistrano

Things I will not cover:
    
* Serving your site via nginx / apache.
* Configuring Jekyll

As stated in the [Capistrano](https://github.com/capistrano/capistrano) docs, Capistrano (aka Cap) is a tool written in Ruby to help manage the deployment of webapps. With this said, everything I'm going to note in this tutorial can be found in the [Capistrano Wiki](https://github.com/capistrano/capistrano/wiki).

Firstly, we'll want to install Capistrano and navigate to our project.
{% highlight bash %}
gem install capistrano
cd PATH_TO_YOUR_PROJECT
{% endhighlight %}

If you use [RVM](https://rvm.io/), you will want to make sure you're using the version of Ruby you'd like to use and you're currently using the proper gemset.

Now that we're inside of our project, let's capify it by typing `capify .`.
{% highlight bash %}
capify .
[add] writing './Capfile'
[add] making directory './config'
[add] writing './config/deploy.rb'
[done] capified!
{% endhighlight %}
As we can see, this command generated two files (`Capfile`, `config/deploy.rb`) and a directory (config). The `Capfile` is the main file that capistrano automatically loads and in our case it is very minimal. All it does in this scenario is load the `config/deploy.rb` file. The `deploy.rb` file contains all the contents that are pertinent to this tutorial.

Let's look at the contents of `config/deploy.rb`.
{% highlight ruby %}
set :application, "hangry" # Sets the name of our application, we could have more than one blog for example.
set :repository,  "git@github.com:muffs/hangry.git" # Where can Capistrano find your application on github?

set :scm, :git # What type of version control do you use? This could also be `git`, `mercurial`, `subversion`, etc..
set :deploy_to, "/home/ubuntu/blog" # Where on your server do you want to place your project?
set :user, "ubuntu" # What user should Capistrano SSH as?

role :blog, "12.12.12.12" # Where is your server? lol

ssh_options[:forward_agent] = true # This tells Capistrano to use your local keys for github rather than keys found on the server.
{% endhighlight %}

Once we're all set with your `deploy.rb` file, we need to run two commands in our shell.
{% highlight bash %}
cap deploy:setup
cap deploy
{% endhighlight %}

`cap deploy:setup` quite literally sets up the directories used for deployment. Capistrano manages deployments in an interesting way by simply changing symlinks to allow for easy versioning. A great explanation of the directory structure can be found [here](https://github.com/capistrano/capistrano/wiki/2.x-from-the-beginning). `cap deploy` is the command that: accesses github, clones your target repository, and then moves the `current` symlink to point to your most recent version. Pretty slick process.

Now you should be all set, from here you can make changes to your blog on your local machine, push those changes to github, and use `cap deploy` to have your changes properly represented on your server.

Julian
