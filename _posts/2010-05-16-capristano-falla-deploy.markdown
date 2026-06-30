---
layout:     post
title:      "Capristano falla al realizar deploy"
date:       2010-05-16 01:32:00
author:     "Daniel Vela"
og_image: "/img/post-bg-05.jpg"
locale:       en
lang-ref:   capristano-falla-deploy
---

There is a lot of failures we can expect deploying with Capistrano.
I'm going to comment one of them. This failure appears executing a 'cap deploy' command, and it returns the following error:

{% highlight bash %}
executing "svn checkout -q -r4 svn+ssh://root@[server_name]/root/[...] /var/www/html/[...]/releases/20100515165617 && (echo 4 > /var/www/html/[...]/releases/20100515165617/REVISION)"  
servers: ["[server_name]"]  
[server_name] executing command  
** [[server_name] :: err] Address [ip.ip.ip.ip] maps to [server_name], but this does not map back to the address - POSSIBLE BREAK-IN ATTEMPT!  
** [[server_name] :: err] Permission denied, please try again.  
** [[server_name] :: err] Permission denied, please try again.  
** [[server_name] :: err] Permission denied (publickey,gssapi-with-mic,password).  
** [[server_name] :: err] svn: Connection closed unexpectedly  
command finished[/shell]  
{% endhighlight %}

Although, if we execute this command in the server,...

{% highlight bash %}
svn+ssh://root@[server_name]/root/[...] /var/www/html/[...]/releases/20100515165617 && (echo 4 > /var/www/html/[...]/releases/20100515165617/REVISION)[/shell]
..., funciona perfectamente.
{% endhighlight %}

The couse of this error is that **capristano** expects that the ssh protoocol authentications works unmanned using the public key of the user. But for some reason I don't know, the authentication fails.

To solve this problem we can use the following method:

In the file ***Capfile*** of our Rails project include the following line:

{% highlight bash %}
default_run_options[:pty] = true  
{% endhighlight %}

Setting this change and executing **cap deploy** capristano command ask us to type the password of the ssh connection with the server. It's not a pretty solution, but it works.
Al aplicar este cambio y ejecutar el **cap deploy** capristano nos pedirá que introduzcamos el pasword de nuestra conexión ssh con el servidor. No es una solución bonita, pero funciona.