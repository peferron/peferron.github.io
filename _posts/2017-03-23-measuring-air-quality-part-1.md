---
layout: post
title: "Measuring air quality in my home (part 1)"
description: "How to measure air quality using a Dylos particle counter, a Raspberry Pi, Rust, Redis, InfluxDB and Grafana."
---

I lived for about three years in Beijing and Shanghai. This was around 2012, when air pollution was fast becoming a matter of concern. [Multiple studies](https://ehp.niehs.nih.gov/1408092/) were singling out particulate matter below 10&nbsp;µm in diameter (PM<sub>10</sub>) and below 2.5&nbsp;µm in diameter (PM<sub>2.5</sub>) as key contributing factors to lung cancer.

{% include figure.html srcset="/images/pm25/pm25_768x536.jpg 768w, /images/pm25/pm25_1536x1072.jpg 1536w, /images/pm25/pm25_1950x1361.jpg 1950w" src="/images/pm25/pm25_768x536.jpg" caption="Particulate matter size comparison <a href='https://www.epa.gov/pm-pollution/particulate-matter-pm-basics'>from the EPA</a> before they got defunded." %}

Trying to protect both my then-girlfriend (now wife 💕) and myself, I started looking at air purifiers. But most purifiers, at the time, didn't come with embedded air quality sensors. You were just supposed to turn them on and hope for the best. That's unsatisfactory for several reasons:

- How do you know if the air quality reported by third parties accurately reflects the air quality around your home?

- In Shanghai, we were lucky to have the [U.S. Consulate air quality monitoring station](https://twitter.com/cgshanghaiair) a few miles away, but it still didn't tell us the air quality inside our home. Air purifiers get noisy at higher settings—how do you know if you can safely move to a lower setting or even turn it off for the night?

- How do you know whether your windows and doors are leaking and letting in external pollution? Back-of-the-envelope math show that even small exchange flows with the outside can ruin your day. Buying adhesive foam or rubber liner at the hardware store can be a much better investment than buying additional purifiers.

- How do you know when to replace [HEPA](https://en.wikipedia.org/wiki/HEPA) filters? My testing showed that if you eyeball it and replace filters when they look dirty, you are likely overspending. (Funnily enough, some filters capture a higher % of particles dirty than new. But as airflow decreases due to higher resistance, you still eventually end up capturing fewer particles per unit of time.)

{% include figure.html src="/images/filter/filter_768x222.png" caption="The middle filter has a solid 50 days of life left, according to the thresholds from the great folks at <a href='http://smartairfilters.com/en/blog/hepa-longevity-test-day-200/'>Smart Air Filters</a>. I bought a few of their cheap purifiers while in China." %}

I decided to buy an air quality sensor. At the time, the only widely available ones were the [Dylos](http://www.dylosproducts.com/) line of products. How these devices work is quite simple. A small fan creates an airflow through a duct; a laser shines a beam through the airflow; and a photodetector registers when the beam is scattered by particles in the airflow. If you want to know more, there's a [good teardown](http://woodgears.ca/dust/dylos.html).

This gives us an estimate of the particle count per unit of volume. In our case, Dylos displays the particle count per 0.01 cubic foot of air. Yes, that's barbaric, but it's America and everything is possible, so I'm just grateful they didn't do it in fl.&nbsp;oz.

More expensive air quality sensors, like the ones used by governments, do not use the laser approach, and measure PM<sub>2.5</sub> in a more civilized µg/m<sup>3</sup>.

The issue is that there is no exact conversion from particle count to weight. Fortunately, correlation is high and approximate conversion formulas exist. This allows us to compare interior air quality measured by a Dylos, with exterior air quality measured by a government station.

{% include figure.html srcset="/images/empreinte/empreinte_768x617.jpg 768w, /images/empreinte/empreinte_1536x1235.jpg 1536w, /images/empreinte/empreinte_2284x1836.jpg 2284w" src="/images/empreinte/empreinte_768x617.jpg" caption="Empreinte. Huile sur toile, 2008, <a href='http://www.ferron-verron.com'>Odile Ferron-Verron</a>." href="http://www.ferron-verron.com" %}

The Dylos was instrumental ([/r/dadjokes](https://www.reddit.com/r/dadjokes/)) in helping me fight pollution in my home in China. At the end of the day, though, it was a losing fight. Staying stuck in your home because _going out may give you cancer_ is exactly as awful as it sounds. There's probably a dystopian YA movie somewhere with a similar plot. (It also reminds me of Hugh Howey's excellent [Wool](https://smile.amazon.com/Wool-Omnibus-Kindle-Motion-Silo-ebook/dp/B0071XO8RA) series.)

China sacrificed the environment to let breakneck industrialization raise hundreds of millions of people out of poverty. I don't completely blame them, but I do feel hugely for the people and families who live there. And now that I'm back under the blue skies of Northern California, I know exactly why they are worth preserving.

[Part&nbsp;2](/2017/04/07/measuring-air-quality-part-2/) will be about how to pipe together a Serial-to-USB cable, a Raspberry Pi, Redis, InfluxDB, Grafana, and Rust to stream Dylos measurements straight to my homepage.
