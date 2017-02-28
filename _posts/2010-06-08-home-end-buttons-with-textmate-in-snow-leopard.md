---
layout: post
title: Shortcuts (&#8984;&#8594; &amp; &#8984;&#8592;) with Textmate and Snow Leopard
comments: true
---
Don’t know why these two shortcuts (aswell as home and end buttons) stoped working in textmate after I upgraded to SL.

The solution (based on [this post](http://blog.macromates.com/2005/key-bindings-for-switchers/) of the textmate blog) is simple: create and edit the DefaultKeyBinding.dic file:

{% highlight bash %}
    mkdir ~/Library/KeyBindings
    cd ~/Library/KeyBindings
    touch DefaultKeyBinding.dict
    open -a TextMate DefaultKeyBinding.dict
{% endhighlight bash %}

Paste these bindings:

    {
        /* home */
        "\UF729"  = "moveToBeginningOfLine:"; /* home */
        "@\UF702"  = "moveToBeginningOfLine:"; /* command + left arrow*/
        "$\UF729" = "moveToBeginningOfLineAndModifySelection:";
        "$@\UF702" = "moveToBeginningOfLineAndModifySelection:";
    
        /* end */
        "\UF72B"  = "moveToEndOfLine:";
        "@\UF703"  = "moveToEndOfLine:";
        "$\UF72B" = "moveToEndOfLineAndModifySelection:";
        "$@\UF703" = "moveToEndOfLineAndModifySelection:";
    
        /* page up/down */
        "\UF72C"  = "pageUp:";
        "\UF72D"  = "pageDown:";
    }

Restart Textmate & enjoy : )