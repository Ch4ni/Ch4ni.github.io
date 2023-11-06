---
layout: post
title: Expanding-the-T-Mobile-Activity-Log
date: 2023-03-07 00:00 +0000
last_modified_at: 2023-11-05 00:00 +0000
category: blog
tags:
  - accessibility
  - frontend
  - javascript
  - T-Mobile
---

{% include notification.html
message="To skip right to the code and instructions, start at the bottom of the page. There should be no ads or life story that make scrolling terribly tedious."
status="is-info"
dismissable="true"
%}

I'm a professional ... and that means: I do expense reports! Yay!

One of my recurring expenses, a cell phone, is reimbursable for on-call rotation. I became a T-Mobile customer almost 5 years ago, starting with a pre-pay plan that gets unlimited calling (who _calls_ people on the phone anymore?), and enough bandwidth to satisfy my needs. At a paltry $45 USD/month, there's been _no need_ for me to switch to another plan, so I haven't.

As a pre-pay T-Mobile customer, my account is a second-class account. There is _no way_ for me to request an itemized bill for the lines on my account. To get that, I'd need to be a post-pay customer. I'm not going to do that, for a few reasons. The first reason is: I _like_ my plan, why should I change it just to get access to something I should already have? The second is that there are few to _no_ plans at my current rate, and those offer significantly less than what I currently get. Again, why should I change?

My workaround? Gather the data I need from the T-Mobile website, combined with transactions from my credit card statement. It's good enough to get approval on my expense reports.

The problem? T-Mobile's website auto-collapses line items whenever one expands a second item.

## Solution

{% include notification.html message="## disclaimer
I am not a front-end developer. This is a dirty, dirty script to get the information I need with as little effort as possible."
status="is-danger"
icon="fas fa-exclamation-triangle" %}
<br/>

If we take a look at the source through page inspection, we get to see _a lot_ of generated HTML. I'm using the firefox built-in Web Developer Tools. The snippet below is some of the generated content from the "Activity Log" page:

```html
<p-accordiontab
  _ngcontent-nln-c19=""
  class="m-t-sm ng-tns-c18-58 ng-star-inserted"
  id="p-accordionTab-default-10_0"
>
  <div class="ui-accordion-header ui-state-default ui-corner-all">
    <span class="fa fa-fw fa-caret-down"> </span>
    <!---->
    <!---->
    <a
      href="#"
      role="tab"
      class="ng-tns-c18-58 ng-star-inserted"
      aria-expanded="true"
      aria-selected="true"
    >
      <p-header _ngcontent-nln-c19="">
        <h6 _ngcontent-nln-c19=""></h6>
        <p _ngcontent-nln-c19="" class="blade-desc">
          <!-- snipped by Ch4ni -->
        </p>
      </p-header>
    </a>
  </div>
  <div
    class="ui-accordion-content-wrapper ng-trigger ng-trigger-tabContent"
    role="tabpanel"
    aria-hidden="true"
  >
    <!---->
    <div
      class="ui-accordion-content ui-widget-content ng-tns-c18-58 ng-star-inserted"
      style=""
    >
      <div _ngcontent-nln-c19="" class="p-content">
        <div _ngcontent-nln-c19="" class="row">
          <p _ngcontent-nln-c19="" class="col-12 m-t-sm">
            <!-- snipped by Ch4ni -->
          </p>
        </div>
        <!---->
      </div>
    </div>
  </div></p-accordiontab
>
```

The `div` class names, and `p-accordiontab` tag were enough context for me to figure out that this view makes use of [PrimeNG accordion](https://primeng.org/accordion) widgets. I tried to figure out how to trigger the multi-select by manipulating only the top-level accordion widget, but am limited here by only looking at the code _after_ angular has had its way with the dom.

So, to make sure I could look at all elements, I iterated through each child node of the `accordionTab` that I was interested in, and manually modified the attributes to render the nodes while bypassing DOM events (`onClick` and friends).

It would be _so easy_ to add multi-select as an option for this display, but alas, my feature request with T-Mobile has gone unnoticed for well over a year, so ... here's the code to get around that particular limitation for anyone else who needs it.

## The Code

<script src="https://gist.github.com/Ch4ni/7ddd008c7dae1b2da11e894ce443d008.js"></script>

## Instructions

Copy the contents of the code block above. When visiting the T-Mobile website for pre-pay customers, perform the appropriate "activity" filtering using the web UI (I do for the month I'm looking at and choose "Billing and Payment" activity). Once satisfied, paste the code into the Developer Console, and you should see every item in the activity log expand.

... you're welcome.
