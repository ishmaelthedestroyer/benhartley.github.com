---
layout: post
title: "Search the web from Vim"
date: 2012-07-21 11:12
comments: true
categories: [vim]
---

Seamless web search - lose less focus<!-- more -->

## Laziness, Impatience...
90% of the time I switch context away from Vim, it's to load Chrome to search for something. In order to make this as seamless as possible, I've added a function to my .vimrc which brings up a prompt where I can enter my search terms, hit enter and it launches Chrome and searches the search engine of my choice with the entered text.

I should note - this is for Chrome on OS X, however, it could be adapted to work on other platforms and with other browsers.

``` vim Key mapping for search prompt
function! Terms()
	call inputsave()
	let searchterm = input('Search: ')
	call inputrestore()
	return searchterm
endfunction
map © <ESC>:! /usr/bin/open -a "/Applications/Google Chrome.app" 'https://google.com/search?q=<C-R>=Terms()<CR>'<CR><CR>
```

The above code maps `alt + g` to bring up the prompt, hitting enter then launches a Google search in Chrome. You needn't restrict yourself to just searching Google either - for [DuckDuckGo](https://duckduckgo.com/) you can replace `https://google.com/search?q=` with `https://duckduckgo.com/?q=` or for [Blekko](https://blekko.com/) you can use `https://blekko.com/ws/`.

You can check out [my full .vimrc](https://github.com/benhartley/dotfiles/blob/master/vimrc) on GitHub.
