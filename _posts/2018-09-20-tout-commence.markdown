---
layout: post
title:  "Tout commence par ... spring-boot-maven-plugin"
date:   2018-09-20 14:53:00 +0200
categories: 
---
Eh oui, il faut bien commencer quelque part, mais par où commencer et comment ?
Peut-être d'abord se présenter ? Ou alors présenter des objectifs et un plan à cinq ans sur ce que sera ce blog, un plan que l'on ne suivra jamais, of course.
Ou alors entrer tout droit dans le vif du sujet ? Foin de blah blah, on veut du code...

Savez-vous que le plugin maven spring-boot est un peu capricieux ?
Pour lui faire prendre en compte un profile spring boot, il ne faut pas passer par l'habituelle variable `spring.profiles.active`, mais plutôt par `run.profiles` ou `spring-boot.run.profiles` (selon la doc officielle).

{% highlight shell %}
mvn -Drun.profiles=default,monsuperprofile spring-boot:run
{% endhighlight %}

Voilà ... je dis cela, je dis rien ...
![Joy](/assets/g001.gif)


