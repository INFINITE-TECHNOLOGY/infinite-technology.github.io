# Groovy Traits static and instance init blocks

**2018-11-17**

This is our first post!

Today we will show an example of a new useful Groovy Traits feature: 
- Multiple inheritance of static and instance init blocks

Note: it seems that this is first post about this feature as it was implemented just a few days back thanks to Groovy Team and [paulk_asert](https://groovy-community.slack.com/team/U2P6GPHHC).

Let's consider below use case:
* Several classes extend java.lang.Thread and are instantiated into a limited number of instances
* These classes are not within same inheritance hierarchy except for the common Thread ancestor
* However these classes share certain traits:
    * Every instantiation increments a static counter ("instance #")
    * Upon instantiation, it is needed to automatically set thread name to Simple Class Name + instance #, e.g.:
        - SenderThread1, SenderThread2, ReceiverThread1, ReceiverThread2, etc...

This is a classical example when multiple inheritance can be applied.

Moreover - this is the case when static and instance [init blocks](https://stackoverflow.com/a/3987586/7727700) can be used.

Java does not support multiple inheritance except for the interfaces.

That is where Groovy comes to our help - with the awesome "traits" concept.

And starting from Groovy version 2.5.5 - the "traits" can support init blocks - allowing multiple inheritance of initialization traits.

Let's try it out!

```groovy
trait InitializationTrait {
    static Integer instanceCounter
    static {
        instanceCounter = 0
    }
    {
        instanceCounter = instanceCounter + 1
        setName(getClass().getSimpleName() + instanceCounter)
    }
}
class SenderThread extends Thread implements InitializationTrait {
    @Override
    void run() {
        System.out.println("Starting thread " + getName())
        System.out.println("Sending")
    }
}
class ReceiverThread extends Thread implements InitializationTrait {
    @Override
    void run() {
        System.out.println("Starting thread " + getName())
        System.out.println("Receiving")
    }
}
(0..3).each {
    new SenderThread().start()
    Thread.sleep(200)
    new ReceiverThread().start()
    Thread.sleep(200)
}
```

And the result:

```
Starting thread SenderThread1
Sending
Starting thread ReceiverThread1
Receiving
Starting thread SenderThread2
Sending
Starting thread ReceiverThread2
Receiving
Starting thread SenderThread3
Sending
Starting thread ReceiverThread3
Receiving
Starting thread SenderThread4
Sending
Starting thread ReceiverThread4
Receiving
```

Happy hacking!

Live Groovy. Code Groovy.

<div id="disqus_thread"></div>
<script>

/**
*  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
*  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/
/*
var disqus_config = function () {
this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};
*/
(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');
s.src = 'https://i-t.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
                            