---
layout:     post
title:      "Soundex: fonetic search"
date:       2010-05-15 18:59:00
author:     "Daniel Vela"
og_image: "/img/post-bg-01.jpg"
---

[http://en.wikipedia.org/wiki/Soundex](http://en.wikipedia.org/wiki/Soundex)

Soundex is a phonetic algorithm for indexing names by sound, as pronounced in English.

{% highlight ruby %}
class String  
    SoundexChars = 'BPFVCSKGJQXZDTLMNR'  
    SoundexNums &nbsp;= '111122222222334556'  
    SoundexCharsEx = '^' + SoundexChars  
    SoundexCharsDel = '^A-Z'  
    # desc: http://en.wikipedia.org/wiki/Soundex  
    def soundex(census = true)  
        str = upcase.delete(SoundexCharsDel).squeeze  
        str[0 .. 0] + str[1 .. -1].  
        delete(SoundexCharsEx).  
        tr(SoundexChars, SoundexNums)[0 .. (census ? 2 : -1)].  
        ljust(3, '0') rescue ''  
    end  

    def sounds_like(other)  
        soundex == other.soundex  
    end  
end  
# puts "Quadratically".soundex  
# 'Q363'  
# puts "Quadratically".soundex(false)  
# 'Q36324'
{% endhighlight %}