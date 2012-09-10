---
layout: post
title: How I use chef-solo with ubuntu 12.04
category: ubuntu chef chef-solo devops
---

Having recently started working with [chef](http://www.opscode.com/chef/) I have come up with a simple method of kickstarting use of it in the small
end of town. This really is designed for those of use managing a large number of very similar servers, or a small number
of simple servers.

After watching this [great presentation](http://www.youtube.com/watch?v=he7vxhm6v64&feature=youtu.be) on chef by [@benr](https://twitter.com/benr) from [joyent](http://joyent.com/), I decided it was time to roll up my sleeves and
get started with chef. To do this I took some of his advice and my meager Unix knowledge and crafted a simple bootstrap method
for my development [Ubuntu](http://ubuntu.com) systems, which I will describe in this post.

Firstly I get myself an ubuntu development system which I use to build my develop the profile I will use for this system
in the future. If this is done locally I grab the ubuntu install CD for the server version and follow the default
installation options with the only service I install being openssh. If your using a VPS this is typically what you get out
of the box.

Because I am doing this on a local server I normally use `ssh-copy-id` to copy over my ssh public key the easy mode way.

ssh-copy-id admin@ubuntuserver

Next I run my [bootstap](https://gist.github.com/3328844) script to install my environment on the server, note the link below
is retrieving a specific revision based on the raw link in the gist this may change based on my updates.

{% highlight bash %}
ssh admin@ubuntuserver -t -C 'curl https://raw.github.com/gist/3328844/2f4d74d49f8f7a2cd0b7a83a23fafe75d21241cf/gistfile1.sh | sudo bash'
{% endhighlight %}

In the next few steps I export my chef-solo template project to the system and build up a recipe for producing this
type of system. You may do this differently, I have a script with all these commands in it but I have exploded it
for this example.

I start my dev cycle by generating a new ssh key pair and upload that to github or bitbucket depending who you use.

{% highlight bash %}
ssh admin@ubuntuserver -t -C "ssh-keygen -t rsa -b 4096 && cat ~/.ssh/id_rsa.pub"
{% endhighlight %}

Make the chef directory and chown it for my admin user.

{% highlight bash %}
ssh admin@ubuntuserver -t -C "sudo su - -c '(mkdir /var/chef && chown admin:admin /var/chef)'"
{% endhighlight %}

Clone my git project whilst retaining ownership of the files by my admin user.

{% highlight bash %}
ssh admin@ubuntuserver -t -C "git clone git@github.com:wolfeidau/chef-solo-base.git /var/chef"
{% endhighlight %}

Initialise the sub modules and update them, this is something I always forget unless I have a script to follow..

{% highlight bash %}
ssh admin@ubuntuserver -t -C "cd /var/chef && git submodule init && git submodule update"
{% endhighlight %}

Now you can run chef-solo just to ensure it is all running as expected.

ssh admin@ubuntuserver -t -C 'sudo chef-solo -c /var/chef/solo.rb -j /var/chef/node.json'

In my example I have commented out a whole section of example code which bootstraps my CI server using chef, this is
gives me some starting points. Note that I am not an authority on either chef or ruby so my scrappy sysadmin code may
make some people grimace but it is a starting point.

Next some rules I try and live by when using chef:

1. Keep it simple, if a recipe looks complicated or you don't understand it don't use it.
2. Don't use chef as an alternate package manager, make packages using fpm and install them.
3. Please reread #1.

To add more recipes from [opscode-cookbooks](https://github.com/opscode-cookbooks) account simply navigate to the base of your project and run something like the
following example.

{% highlight bash %}
git submodule add https://github.com/opscode-cookbooks/openssl.git cookbooks/openssl
{% endhighlight %}
