---
layout: post
title: "Talk: Modern JavaScript and intro to TypeScript"
---

I did a lunch talk at work about modern JS: ECMAScript 5, 2015 and 2016. Please forgive the playfully baity title. :)

{% include iframe.html src="https://peferron.com/modern-js-talk/" caption="Slides and source code on <a href='https://github.com/peferron/modern-js-talk/'>GitHub</a>." ratio="62.5%" %}

It was my first time using [Reveal.js](https://github.com/hakimel/reveal.js/) and it was awesome. It has several killer features compared to PowerPoint:

* Meaningful version control.
* Embed with a simple `iframe`—like on this blog.
* Share with a simple link.
* Open in any device that has a web browser.

That's the power of the web for you.

Going back to the talk itself, the *wow* moment for the audience was turning our JS into valid TS by simply renaming the file extension from `js` to `ts`, and then demonstrating how type inference already brought increased safety against type mismatches and typos!

 (Alright, nobody actually screamed *wow*, but don't ruin my rosy recollection.)

The TypeScript team has done a great job keeping up with ECMAScript 2015+. It's awesome to see the TS compiler infer the type of `maxHumidity` and `minTemperature` based on their default values:

```js
async function getLovelyDays({maxHumidity = 90, minTemperature = 80} = {}) {
  var response = await fetch('sf-forecast.json');
  var forecasts = await response.json();
  ...
}
```

The next step was showing the limits of type inference. The TS compiler cannot infer the type of `forecasts` since it's parsed from external JSON data. We solved it by declaring an interface and adding a type annotation:

```ts
interface Forecast {
  day: string;
  humidity: number;
  temperature: number;
}

async function getLovelyDays({maxHumidity = 90, minTemperature = 80} = {}) {
  var response = await fetch('sf-forecast.json');
  var forecasts: Forecast[] = await response.json();
  ...
}
```

It was a very cursory introduction to TS, but definitely worth the remaining 10 minutes after the end of the main talk.

Another thing that worked well was switching back and forth between slides and text editor to upgrade a sample app from "old" JS to modern JS, one feature at a time. We went from this:

```js
// Asynchronously calls callback with an array of lovely days.
function getLovelyDays(options, callback) {
  options = options || {};
  options.maxHumidity = options.maxHumidity || 90;
  options.minTemperature = options.minTemperature || 80;
  callback = callback || options;

  var request = new XMLHttpRequest();
  request.open('GET', 'sf-forecast.json', true);

  request.addEventListener('load', function() {
    var forecasts = JSON.parse(request.responseText);
    var lovelyDays = [];
    for (var i = 0; i < forecasts.count; i++) {
      var forecast = forecasts[i];
      if (forecast.humidity <= maxHumidity &&
        forecast.temperature >= minTemperature) {
        lovelyDays.push(forecast.day);
      }
    }
    callback(lovelyDays);
  });

  request.send();
}

// Called on button click.
function showLovelyDays() {
  getLovelyDays({maxHumidity: 95}, function(lovelyDays) {
    var html = '';
    for (var i = 0; i < lovelyDays.length; i++) {
      html += '<li>' + lovelyDays[i] + '</li>';
    }
    document.getElementById('lovely-days').innerHTML = html;
  });
}
```

To this:

```js
// Returns an array of lovely days.
async function getLovelyDays({maxHumidity = 90, minTemperature = 80} = {}) {
  var response = await fetch('sf-forecast.json');
  var forecasts = await response.json();
  return forecasts
    .filter(forecast => forecast.humidity <= maxHumidity &&
      forecast.temperature >= minTemperature)
    .map(forecast => forecast.day);
}

// Called on button click.
async function showLovelyDays() {
  var lovelyDays = await getLovelyDays({maxHumidity: 95});
  var html = lovelyDays.map(day => `<li>${day}</li>`).join('');
  document.getElementById('lovely-days').innerHTML = html;
}
```

It's not only much shorter, but also expresses intent more clearly, and is less bug-prone (no mutation or manual loops).

The full source code of the slides and sample app is on [GitHub](https://github.com/peferron/modern-js-talk).