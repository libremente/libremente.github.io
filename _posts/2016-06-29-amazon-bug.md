---
layout:     post
title:      "Amazon: why is this happening?"
date:       2016-06-29 00:00:00
author:     "surF"
header-img: "img/post-bg-02.jpg"
---

Today while browsing for a new pair of hearphones I encountered this
interesting behaviour of the Amazon marketplace search.<br> 
By inserting the keyword **auricolari** the result is the following:
<br><br>
![The amazon bug](/img/amazon_bug.png )

Why is this happening?

The URL created for this query is the following:

```
https://www.amazon.it/s/ref=nb_sb_noss_2?__mk_it_IT=%C3%85M%C3%85%C5%BD%C3%95%C3%91&url=search-alias%3Daps&field-keywords=auricolari&rh=i%3Aaps%2Ck%3Aauricolari&ajr=2
```

and the _broken_ page appears just when the user is logged in...
