**NOTE: The code for this project is now hosted at**<a href='http://github.com/ripienaar/flashpolicyd'>github</a> 

A multi threaded daemon for serving up XML needed by Adobe Flash 9 and later when making direct socket connections.

For information about why you might need this please see [Policy file changes in Flash Player 9 and Flash Player 10](http://www.adobe.com/devnet/flashplayer/articles/fplayer9_security_04.html).

Adobe supplies a Perl based daemon but I have not found it to be very stable or well written and wrote a Ruby based solution.

This code has been used in production for over a year, I've seen uptimes of 300 days without memory leaks or thread leaks and have served 10s of millions of requests using it.

See the [Introduction](Introduction.md) page for full details.