---
layout: post
title:  "Inventing the Universe? Microframeworks"
date:   2016-08-17 00:00:00 +0930
---

ecently for a side project, I figured I would give the latest version of [http://www.slimframework.com/](Slim Framework<) a shot as I was building a small application where I didn't think I would need many 'full stack' framework features. For most projects, I generally reach for [http://laravel.com/](Laravel<).

The first thing I came up against was that I wanted a templating engine, Blade? nope, we're using slim! So off I went and pulled in the Slim-Twig package, great, I have templates. I was quite familiar with Twig and was quite simple to get setup.

My next job was fetching some data from an XML feed and mapping it to my own entities, it was rather slow to do this every page load so caching to the rescue! I reached for [http://github.com/doctrine/cache](Doctrine Cache<) and just used the filesystem driver, great success! Next?

I kept developing, then got sick of having to manually wire up my container services, this is a somewhat controversial this view but I like the auto wiring feature in the Laravel container, so I ventured out and ended up pulling in [http://github.com/PHP-DI/Slim-Bridge](PHP-DI Slim Bridge<) so that I could have some auto-wiring!

The next hurdle was authentication, I had a bit of a unique requirement where I was authenticating to an LDAP server, but to check if the credentials are valid, I need to try to 'bind' to the server, which is the term used for authenticating to an LDAP server. Authenticate to authenticate you say? Yup.

For this I ended up writing a light wrapper over [http://github.com/zendframework/zend-authentication](Zend Authentication<), in this case, I feel like this was better suited to the situation, and not trying to shoehorn my LDAP authentication requirement into the Laravel's built-in authentication.

I had built a custom 'framework' of sorts with components from everywhere! Great you think, small decoupled libraries are fantastic so we can build our own small application from the ground up.

Truth be told looking back at the experience, even these simple things that I needed, (Caching, Templates, Authentication) are already solved for me in Laravel, out of the box, with almost zero configuration, 'it just works'.

I'm not dismissing micro frameworks, I just feel for this use, and many other uses, the time saving and developer productivity gains with using a full stack framework outweighs lots of manual configuration, picking what library to use and putting it all together.

While working on this, another thing crossed my mind. What if this wasn't a side project? What if it was some huge production application, the cognitive overhead of trying to keep all these libraries and components in your head would be quite overwhelming, and imagine how long it would take to get another developer up to speed? Full stack frameworks solve the already solved problems for us, so why reinvent the universe?

You can find the code for the project on my Github, [http://www.github.com/samgreenwood/nodemap](http://www.github.com/samgreenwood/nodemap<)

And if you're interested in long-range wireless networks and happen to live in Adelaide, Australia, come check our [http://www.air-stream.org](Air-Stream Wireless<)
