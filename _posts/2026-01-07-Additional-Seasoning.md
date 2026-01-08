---
layout: post
title: "Additional Seasoning"
categories: "Site Updates"
tags: "Site Updates"
author: "Apostla"
---

I've completed the HTML/CSS for the `post` layout for the most part, though I'm still building off the `minimal` Jekyll theme. I've added several new features this update:

- Global smooth scrolling
- Corrected list coloring and multi-line code block artifacts
- Enhanced contrast for some text elements and multi-line code blocks
- Custom scrollbar
- A global, scriptless "To Top" button, which will be enhanced later

I'm going to stylize those bare index pages next and start showcasing some big projects! Hang tight!

Oh, and as always, here's the updated code:

```scss
---
---
@import "jekyll-theme-minimal";

html {
  scroll-behavior: smooth;
}

body {
    background-color: #28282f;
}

h1 {
    color: white;
}

p, li {
    color: lightgrey;
}

small {
    color: #afafaf
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

.highlight {
    background-color: transparent;
}

pre.highlight {
    background-color: #1f1f1f;
    border: none;
    border-radius: 12px;
    .nt { color: #569cd6 !important; }
    .nl { color: #9cdcfe !important; }
    .mh, .m, .mi { color: #b5cea8 !important; }
    .s2, .s { color: #ce9178 !important; }
    .o, .p { color: #d4d4d4 !important; }
    .nd { color: #a6e22e !important; }
    .no, .nb { color: #4ec9b0 !important; }
    .k { color: #ff90d3 !important; }
    .nc { color: goldenrod !important; }
    .n {color: #d800ff}
}

pre code {
    background-color: transparent;
    padding: 0;
}

::-webkit-scrollbar-track {
    background-color: #1f1f1f;
}

::-webkit-scrollbar-thumb {
    background-color: #303060;
    border-radius: 8px;
    border: 2px solid #1f1f1f;
    transition: all 0.2s ease;
}

::-webkit-scrollbar {
    width: 12px;
}

::-webkit-scrollbar-thumb:hover {
    background-color: #383868;
}

#Top-button {
    font-size: 12px;
    padding: 8px;
    position: fixed;
    bottom: 10px;
    left: 60%;
    background-color: #000000;
    border-radius: 32px;
    border: 1px solid #1f1f1f;
    transition: all 0.2s ease;
    z-index: 10;

    @media (max-width: 960px) {
        left: 45%;
    }
}

#Top-button:hover {
    background-color: #1a1a1a;
    border: 1px solid #2f2f2f;
    text-decoration: none;
}

#Top-button:active {
    background-color: #0f0f0f;
    border: 1px solid #3a3a3a;
}
```