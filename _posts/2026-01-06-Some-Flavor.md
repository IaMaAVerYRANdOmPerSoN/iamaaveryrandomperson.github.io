---
layout: post
title: "Some Flavor"
categories: "Site Updates"
tags: "Site Updates"
author: "Apostla"
---

I began adding my own HMTL/CSS today. You're viewing my crude work through the DevTools `inspector-stylesheet` right now, though I should probably explain the (minor) changes in layman's terms:

>1. I changed the background color from #fffff to #28282f.  
>2. First-order headings are now #fffff.
>3. Paragraphs are now light grey.  
>4. Link behavior has been updated; Inline code has been restyled.  
>5. The sidebar and body have been compacted (With mobile support)

... And (Not Much) More! Here's the updated `style.scss` for reference:

```scss
---
---
@import "jekyll-theme-minimal";

body {
    background-color: #28282f;
}

h1 {
    color: white;
}

p {
    color: lightgrey;
}

a {
    padding-left: 4px;
    padding-right: 4px;
    padding-top: 2px;
    padding-bottom: 2px;
    font-weight: bold;
    color: #267CB9;
    transition: all 0.2s ease;
}

a:hover {
    text-decoration: underline;
    border-radius: 8px;
    background-color: #383838;
    
}

header a:hover {
    background-color: transparent;
}

header a {
    color: whitesmoke;
}

code {
    color: whitesmoke;
    border-radius: 8px;
    background-color: #000000;
    padding-left: 4px;
    padding-right: 4px;
    padding-top: 2px;
    padding-bottom: 2px;
}

section {
    width: 100%;
    max-width: 1200px;
    margin: 0 auto;
    
    @media (min-width: 960px) {
        width: 65%;
    }
}
```