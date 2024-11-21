---
title: "Cluster heat output"
author: Christopher Milan
---

<script id="MathJax-script" async src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>

I was curious to get a better sense of how much heat our servers will output.

$$ \Delta T = \frac{Q}{mc} $$

Google tells me that the specific heat for air given a constant volume is
0.718kJ/kg K. Our room in Boelter Hall is about 50 cubic meters, so using
the density of air at STP,[^1] that's about 60 kg of air. Let's just consider
all 14 GPUs running at full-tilt: NVIDIA says that a 3090 has a TDP of 350W,
giving us a total of 4.9kW, that is, about 5kJ per second, or 300kJ per minute.

Putting it all together:

$$\Delta T = \frac{300\rm{kJ}}{(60\rm{kg})(0.718\rm{kJ}/\rm{kg\ K})}\approx 7\rm{K}$$

Which is 12.5 degrees Fahrenheit per minute!

* * *

[^1]: Obviously this isn't quite accurate, but it's close enough.

