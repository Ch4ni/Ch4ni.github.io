---
layout: post
title: Expanding-the-T-Mobile-Activity-Log
date: 2023-03-07 00:00 +0000
category: blog
tags:
  - accessibility
  - frontend
  - javascript
  - T-Mobile
---

## Disclaimer

I am not a web developer. This is a dirty, dirty script to get the information I need for expense reports

## Preamble

I had figured this out once before, but I never backed up that script, and was stuck trying to figure this out _again_ because my old laptop died. That's why we're putting this on github ... so I can get to it the next time I need to use this code on a different machine.

We all know that the accordion tables in use are [PrimeNG accordion](https://primeng.org/accordion) widgets. My question is: _Why_ would they disable multiselect? At worst, shouldn't that be an _optional_ toggle? You know, given that Pre-pay customers are second-tier, so we don't even have the option to request a paper or pdf bill. I'm frugal sometimes, this plan gives me everything I need for the lowest price of a non-MVNO that I've found.

## The Code

```javascript
// The accordion table on the website used for activity log is the '10' table ... why are there so many tables?
document.querySelectorAll("p-accordionTab[id^=p-accordionTab-default-10]").forEach((value, idx, listObj) => {
  
    // Use the selector's node id to get the Element from the DOM so that modifications show live
    // This is needed because the selector results are a static view that doesn't change when the
    // DOM does. I assumed that this meant changes weren't reflected back to the DOM either.
  	var accordionElement = document.getElementById(value.id)
  
    // get the descendant accordion headers to modify
  	Array.from(accordionElement.getElementsByClassName("ui-accordion-header")).forEach((accH) => {

        // We need to set each header as selected and expanded
    		Array.from(accH.getElementsByTagName("a")).forEach((linkNode) => {
          linkNode.setAttribute("aria-expanded", "true")
          linkNode.setAttribute("aria-selected","true")
        })
    
        // And then make every caret look like it's been clicked (is if this was a multi-select accordion table)
    		Array.from(accH.getElementsByClassName("fa-caret-right")).forEach((spanElement) => {
          spanElement.classList.add("fa-caret-down")
          spanElement.classList.remove("fa-caret-right")
        })
    })

    // Get the descendant content wrappers and remove the class and styling that hide the elements
  	Array.from(accordionElement.getElementsByClassName("ui-accordion-content-wrapper")).forEach((wrapper) => {
      	wrapper.removeAttribute("style")
      	wrapper.classList.remove("ui-accordion-content-wrapper-overflown")
    	})
 })
```

## Instructions

Copy the contents of the code block above. When visiting the T-Mobile website for pre-pay customers, perform the appropriate "activity" filtering using the web UI (I do for the month I'm looking at and choose "Billing and Payment" activity). Once satisfied, paste the code into the Developer Console, and you should see every item in the activity log expand.

... you're welcome.


