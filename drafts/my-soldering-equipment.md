---
layout: post
author: chani
title:  "Soldering Equipment"
categories: blog
tags:
  - soldering
  - equipment
  - my setup
---

A question that I've either been asked personally, or seen asked in public forums several times now is: what should I get to begin soldering?

The succinct, accurate answer is: it really depends on what you want to do!

That answer, however, isn't really helpful for anyone, so I thought it might be nice to provide a list of the equipment I have available, why I chose it, and what might work instead. What works for me may not work for someone else, but at least this list and rationale should give anyone else a starting point to curating their own home lab, or whatever type of kit they want!

Before diving in, it's _very_ important to note: I am _not_ a professional! I don't do any hardware work for money! Tinkering is a hobby for me, and I've been extraordinarily lucky to have access to information, tools, and funds. My needs will differ from someone who does professional hardware design/prototyping, drone operators who need a field kit, folks who perform board-level repair for a living, or even from other hobby tinkerers!

## Soldering Iron

Like many things, this is a fairly subjective item ... a soldering iron that feels good and works well for me may not work well for anyone else, and vice versa.

Equipment Owned:

|Iron|Description|features|Link|
|----|----|----|
|Pinecil|Small, portable soldering iron with an LCD Screen. Powered by a Risc-V processor|Temperature Control, USB-C Power Delivery, Customizable Firmware, [breakout board](https://pine64.com/product/pinecil-break-out-board/), Tip Compatibility*| [Pine64](https://pine64.com/product/pinecil-smart-mini-portable-soldering-iron/), [Amazon](https://www.amazon.com/PINECIL-Smart-Mini-Portable-Soldering/dp/B096X6SG13)|
|T12/FX-951 Clone|Inexpensive, relatively powerful soldering iron. Unexceptional, but not bad.|Temperature Control, Tip Compatibility*|[Amazon](https://www.amazon.com/gp/product/B08DCDMVJ5)(No longer available)|
|[FX-600](https://hakkousa.com/fx-600-soldering-iron.html) Clone|Cheap|Temperature Control, Tip Compatibility*|[Amazon](https://www.amazon.com/gp/product/B083LXQMGK)(No longer available)|

### FX-600 Clone
This soldering iron was _nothing_ but a pain to use. The air gap between the ceramic heating element in the handle and the tip prevented reliable temperature control. Being an all-in-one device (meaning that the power supply, heating element, and controls were all in the handle of the iron) made it difficult to do any work without making accidental adjustments, and the power supply cord was constantly in the way. -2/10 stars, would not recommend.

### T12/FX-951 Clone
This was the first "serious" iron/station I had picked up. It served me well for a time. Even after I had picked up my Pinecil, I continued to use this iron as my "daily driver" (for hobby/tinkering work) for months.

Having the power supply and interface on a box separate from the iron itself made a _huge_ difference in terms of ergonomics. While better than the 900M clone, there are still some folks who are not a fan of these cloned stations, [and for good reason](https://www.eevblog.com/forum/dodgy-technology/dangerous-solder-station-ksger-t12/).

While the kit itself is a no-frills clone of the venerable [Hakko FX-951](https://hakkousa.com/products/soldering/fx-951-soldering-station.html) station, I don't think that I would have chosen this particular model if I had to go down the same road again. The station is _perfectly_ serviceable, albeit a bit risky if you don't know what you're doing and/or take shortcuts, but there are smaller cheaper units all over the place now that implement the same type of stylus (with compatibility for T15/T12 tips). A quick search on amazon pops up units [like this one](https://www.amazon.com/QUICKO-Soldering-Adjustable-temperature-Autosleep/dp/B09Y39FZB2/ref=sr_1_6?crid=OYLHBVH94E0F&keywords=t12+soldering+station&qid=1696227829&sprefix=T12+Sol%2Caps%2C180&sr=8-6).

### Pinecil
I was honestly surprised by the Pinecil ... _of course_ I picked it up as soon as I discovered the thing. A $25 iron powered over USB-C that I could shove into a bag and take with me? Count me In!

Some of the other features that attracted me were the compatibility with TS-100 tips, temperature control with a screen, and the capability to pick up a breakout board should the desire to use it as a Risc-V dev-board ever struck my fancy!

There are some _minor_ annoyances, like how the iron _may_ not fit properly in most standard iron stands, but it's not really a dealbreaker for me. Being a smaller iron, it also lacks thermal mass required to melt giant gobs of solder, but if you're working on smaller PCBs or discrete components, this shouldn't really be an issue (unless you've got a huge ground plane, and choose to use a low temp for whatever reason). Other irons that are also well loved, such as the TS-100 and TS-80 suffer from the same nits, and there are many ways to work around it. I used this iron exclusively for my last keyboard build, and it was a pure joy to work with.

## Soldering Iron Tips

There are as many soldering Iron tips to choose from as there are specialty chips to solder ... the type of tip one chooses to use is very personal.

Many people will tell you that there is one right tip to use for any job, and that the standard conical tip is useless for any job. I will concede that there's _always_ a better tool for any job, the question is: does the tool you selected _do the job_?

I've tried several different tips: conical, long conical, bevel, chisel, knife tip. I _always_ tend to go back to long conical ... and when long conical doesn't work for me, long chisel does the job.

The best way to figure out which tips work best for you is to try a variety of them. When you find yourself gravitating back towards one or two types of tip, that's a pretty good indication that they're working well for you.

## Materials

You're going to _need_ a few materials to get started, and you're going to _want_ a few others. This list is made according to my preferences, and it may not ring true for everyone. This may be a good starting point to consider what one needs for their own setup.

|Needs|Nice to Haves|
|----|----|
|Fume Extractor|Syringes with Blunt-tip dispensing needles|
|Flux|Silicone Mat|
|solder|Tip Tinner|
|Tip Cleaner (brass, not sponge)|Nitrile Gloves|
|99% Isopropyl Rubbing Alcohol (IPA)|Tweezers|
|Cotton Swabs||
|Desoldering Pump or Wick/Braid||


### Needs
#### Fume Extractor
*TL;DR* - I have [this fume extractor](https://www.amazon.com/Extractor-TOAUTO-Noise-free-Adjustable-Adsorption/dp/B09TPXKWVZ). You don't need to buy anything fancy though, if you have a Box Fan lying around you can add activated charcoal filter [like this](https://www.amazon.com/ALL-PARTS-ETC-Anti-Odor-Microwaves/dp/B092RF9DVS/ref=sr_1_18?crid=7HBUGGF4K1DS&keywords=activated%2Bcharcoal%2Bfilter&qid=1696230457&sprefix=activated%2Bcharcoal%2Bfilter%2Caps%2C162&sr=8-18&th=1) on the intake side of your fan, and blow away from the work area. Alternatively, you could do the same with a spare PC case fan, or you could build your own functional fume extractor [like this one](https://oscarliang.com/diy-solder-smoke-extractor/) or any of the ones [tagged on hackaday](https://hackaday.com/tag/solder-fume-extractor/).

I put this at the top of the list for a reason. Solder contains lead ... while there's not _usually_ any reason to be concerned about the lead vaporizing because that doesn't happen below 400 degrees celcius - most hobby soldering should be done around 300-350 degrees celcius (yes, _especially_ those tiny little LEDs ... more on this later); working with molten metal of any kind carries an inherent risk of flicking tiny little balls of molten metal anywhere and everywhere. This, though, isn't even the main concern. _Flux_ is one of the main reason that we want a fume extractor - relative to solder, the flux has a rather low smoking point, and releases fumes that irritate the lungs and mucous lined membranes, [here](https://www.protoexpress.com/blog/soldering-flux-acids-solids-and-solvents/) is a pretty decent article that covers some of the chemistry behind flux, and how it works.

Fortunately, fume extractors don't _need_ to be expensive, or pretty, they just need to keep the air gently flowing away from your workspace to keep smoke and fumes out of your breathing holes!

#### Flux

I've been using [Kester 186](https://www.amazon.com/gp/aw/d/B01MR49JY1) liquid flux for a while now. I've tried several different types of flux to date, from the [ChipQuik flux marker](https://www.amazon.com/Liquid-Flux-No-Clean-10ml-0-34oz/dp/B07B53LNGX), to no-name rosin flux paste that came with a different accessory. There's many that I haven't yet tried, but for me the liquid flux with an appropriate application tool has provided many projects worth of easy-cleanup soldering, with consistent application and reliable results.

I would recommend trying many different formulas and application methods yourself, but if you want a low overhead flux, a good liquid rosin flux has been what I've found reliable and simple.

#### Solder

This is why you're here ... without this wonder of the metallurgy world, we couldn't be sending bits across vast distances as we do today!

Don't bother trying to cut costs on solder ... Invest in a good spool of 60/40 (Tin/Lead) solder. I prefer [kester 24-6040-0027 (0.031")](https://www.amazon.com/dp/B00068IJPO), but that's because I like to have rosin core for tinning my iron, and the diameter is comfortable for me. Most Solders in the Kester 24-0640-* family should be sufficient. The number appears to decode to `<product type>-<alloy ration>-<product index>` - so here: `24` -> any kind of flux-core solder wire ... `14` is the solid wire. `6040` indicates 60% Tin, 40% Lead alloy, and the `0027` is the product index/ID `27` ... a model that has been around, and loved, for decades.

#### Tip Cleaner

Use a copper-wool or brass-wool cleaner. Throw out that sponge.

Seriously though ... a brass or copper wool tip-cleaner can be acquired for about $6USD on amazon, and will prolong the life of your tip. Any one will do. Sponges are bad for your tips because the water in the sponge rapidly changes the temperature, and promotes oxidation (which is _exactly_ what we're trying to avoid by cleaning the tip). The soft metals will remove any oxidized solder or residue from your tip without drastically affecting the temperature, allowing you to protect the tip better by tinning it immediately after cleaning.

#### 99% Isopropyl Rubbing Alcohol

Any sufficiently reputable IPA will do. The only requirements are that it needs to be 99% Isopropyl Alcohol, and it needs to be industrial grade or better.

I started with a small bottle of [belle chemical IPA](https://www.amazon.com/gp/product/B0868R7JF6/ref=ppx_yo_dt_b_asin_title_o09_s00?ie=UTF8&psc=1), and have since used alcohol from [Eyeglass Supply](https://eyeglasssupplystore.com/collections/all-products/products/isopropyl-alcohol?variant=42764561449167), both to good effect. [MG Chemicals](https://mgchemicals.com/products/electronics-maintenance/isopropyl-alcohol/99-9-isopropyl-alcohol/) is another reputable chemical supply company.

#### Cotton Swabs

I like the [combination pointed and flat cosmetic cotton swabs](https://www.amazon.com/CGR-Precision-Flattened-Cosmetic-Applicator/dp/B09VKYBNV6) for board cleanup. The flat tip is good for wiping larger parts of a PCB, and the pointed tips allow you to get into most of the smaller areas, which is great when working on keyboards or other PCBs containing a mix of surface mount and through-hole components.

#### Desoldering Wick/Braid and/or Solder Sucker

For about $8 USD I got [a combo pack containing both](https://www.amazon.com/gp/product/B07CB9V427/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1). These are cheap, no-frills copper wire braids, and an okay solder sucker/desoldering pump.

I honestly much prefer using the braid/wick over the desoldering pump, but it took some time to warm up to it - this was mostly because I started with poor technique. The most effective way to use a soldering braid is to ensure you apply fresh flux and solder to a joint, as well as applying flux to the braid. Ensure your iron is hot (300-350 degrees celcius is sufficient for most cases) and the tip is well-tinned. Depending on the joint, you _may_ need to apply additional solder to the braid to kickstart the capillary action, and ensure good contact with the joint's solder for thermal transfer. Move the braid (and the iron) around to keep the joint in contact with a non-saturated part of the wick that's still got liquid solder on it.

### Nice to Haves

#### Syringes for dispensing

* [1ml luer slip syringe](https://www.amazon.com/1ml-Syringe-Sterile-Luer-Slip/dp/B07BQDRDC2)
* [18ga blunt tip dispensing needle](https://www.amazon.com/20-Pack-Blunt-Dispensing-Needle/dp/B07VX6JB97).

You can use syringe/needle with 99% Isopropyl rubbing alcohol as well. 

#### Silicone Mat

* [I have this one](https://www.amazon.com/gp/product/B07TWFK6PL)

#### Nitrile Gloves


=== Original from discord ===

Flux: I use [this](https://www.amazon.com/gp/aw/d/B01MR49JY1) with a [1ml luer slip syringe](https://www.amazon.com/1ml-Syringe-Sterile-Luer-Slip/dp/B07BQDRDC2) and an [18ga blunt tip dispensing needle](https://www.amazon.com/20-Pack-Blunt-Dispensing-Needle/dp/B07VX6JB97). You can use syringe/needle with 99% Isopropyl rubbing alcohol as well. 

[Combination pointed and flat cosmetic cotton swabs](https://www.amazon.com/CGR-Precision-Flattened-Cosmetic-Applicator/dp/B09VKYBNV6) for board cleanup.

You'll also likely want a desoldering wick/braid. Don't bother with pre-fluxed ones ... with the liquid flux and a tinted tip you can get good results.
  
Silicone soldering mats are _nice_ but not mandatory.

Some kind of fume extractor is recommended (do _not_ breathe in solder fumes!) ...I have one of [these](https://www.amazon.com/Extractor-TOAUTO-Noise-free-Adjustable-Adsorption/dp/B09TPXKWVZ) because I was too lazy to build my own, but on the cheap and easy, a box fan or PC fan with some type of activated carbon filter (close to, but pointed _away_ from the work area) should suffice ... air purifier filters are usually inexpensive, and you can cut them to size.

If you don't like things on your hands, nitrile gloves are nice.

Solder ... I've been using a roll of [this](https://www.amazon.com/dp/B00068IJPO) for about 3 years. It's worth the relatively high up front cost compared to other solder available on Amazon.
