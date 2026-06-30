---
layout:     post
title:      "RDoc Notation Reference"
date:       2010-06-01 04:48:00
author:     "Daniel Vela"
og_image: "/img/post-bg-06.jpg"
locale:       en
lang-ref:   rdoc-notation
---

# Simple RDoc Reference (written in RDoc!)  
 #  
 #--  
 # RDoc will not process what's between the -- and ++, so you can  
 # put notes or messages here (like FIXMEs or TODOs, that will not  
 # end up in your documenation.  
 #++  
 #  
 # = Heading 1  
 # == Heading 2  
 # === Heading 3  
 # ==== Heading 4  
 #  
 # == Lists  
 #  
 # * First List  
 # * first list first item  
 # * second list second item  
 # * Second List  
 # * second list first item  
 # * second list second item  
 #  
 # == Numbered Lists  
 #  
 # 1. Item number one  
 # 2. Item number two  
 # 3. Item number three  
 #  
 # == Links  
 #  
 # Visit {My Personal Blog}[http://www.ozmox.com]  
 #  
 # == Text Formatting  
 #  
 # Text can be shown in *bold* and _italic_. You can even  
 # denote +code_word+ to appear as typewritter with pluses.  
 #  
 # A lot of people write their options out like this:  
 #  
 # :option -- description of what to pass  
 #  
 # class Indented  
 # def text(appears)  
 # as code  
 # end  
 # end  
 #  
 # label:: With double colons  
 #  
 # :title: RDoc Simple Reference  