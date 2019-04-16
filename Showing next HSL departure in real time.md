# Showing next HSL departure in real time

I've long wanted a screen next to the front door showing the next departure of my preferred public transportation, in real time. Eventually, I made one, using the lovely [HSL GraphQL API](https://digitransit.fi/en/developers/apis/1-routing-api/0-graphql/).

## HTML

Nothing too magical here, we just need an element to update with the next departure time:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>HSL Stop Next Departure</title>
  </head>
  <body>
    <script src="hsl-stop-next-departure.js" async></script>
    <pre>Next departure in <strong id="next-departure">n/a</strong> minutes</pre>
  </body>
</html>
```

So something like this:

![image](https://user-images.githubusercontent.com/560055/56206681-99c7b700-6055-11e9-8a9b-b5d043088e87.png)

## JS

Put this into `hsl-stop-next-departure.js`:

```js
(() => {
  const HSL_STOP_ID = 'HSL:1220430'; // to look up this ID for the stop you want, query https://api.digitransit.fi/routing/v1/routers/hsl/index/graphql with e.g. { "query": "{stops(name: \"Kotkankatu\") {name gtfsId code}}" }
  const HSL_ROUTE_NAME = '9'; // use this to filter the stop departures to only the route you want (e.g. tram "9")
  const HSL_TIME_RANGE = 3600; // how much into the future to query; in this case, if there's no departures in 1 hour, show nothing
  const HSL_DEPARTURES = 3; // how many results to receive, at most; you may need to increase this for busy stops

  const UI_DATA_REFRESH = 30 * 1000; // how often to query the HSL API for new data; please don't abuse it
  const UI_REFRESH = 1000; // how often to update the UI; remaining time is extrapolated based on the latest data refresh

  const $el = document.querySelector('#next-departure'); // this is the element that will be used for output

  let data = null;

  start();

  return; // only function declarations below here

  function start() {
    refreshData();
    setInterval(refreshData, UI_DATA_REFRESH);
    setInterval(refreshUi, UI_REFRESH);
  }

  function log(...args) {
    console.log.apply(null, [{ HSL_STOP_ID, HSL_ROUTE_NAME }].concat(args));
  }

  function refreshData() {
    return getNextDeparturesFromStop(HSL_STOP_ID)
      .then(res => {
        log('Got API response', res);
        const nextDep = res.data.stop.departures.filter(dep => dep.trip.pattern.route.shortName === HSL_ROUTE_NAME)[0];
        if (nextDep) {
          const delta = nextDep.serviceDay + nextDep.realtimeDeparture - Math.round(Date.now() / 1000);
          data = {
            hslTimeTilNext: delta,
            hslIsRealTime: nextDep.realtime ? '' : '~',
            hslLastDataRefresh: Date.now()
          };
        } else {
          log('No departures found within time limit');
          data = null;
        }
      })
      .catch(err => {
        log('Got API error', err);
        data = null;
      });
  }

  function refreshUi() {
    if (data) {
      const dataAge = Date.now() - data.hslLastDataRefresh;
      const compensatedTimeTilNext = Math.max(0, data.hslTimeTilNext - Math.round(dataAge / 1000));
      $el.textContent = data.hslIsRealTime + (compensatedTimeTilNext / 60).toFixed(1);
    } else {
      $el.textContent = 'n/a';
    }
  }

  function getNextDeparturesFromStop() {
    const url = 'https://api.digitransit.fi/routing/v1/routers/hsl/index/graphql';
    const query = `
      query StopDepartures($stopId: String!, $startTime: Long!, $timeRange: Int!, $numberOfDepartures: Int!) {
        stop(id: $stopId) {
          code
          name
          departures: stoptimesWithoutPatterns(startTime: $startTime, timeRange: $timeRange, numberOfDepartures: $numberOfDepartures) {
            scheduledDeparture
            serviceDay
            realtime
            realtimeDeparture
            headsign
            trip {
              pattern {
                route {
                  shortName
                }
              }
            }
          }
        }
      }
    `;
    const variables = {
      stopId: HSL_STOP_ID,
      startTime: Math.round(Date.now() / 1000),
      timeRange: HSL_TIME_RANGE,
      numberOfDepartures: HSL_DEPARTURES
    };
    return fetch(url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        query,
        variables
      })
    }).then(res => res.json());
  }
})();
```

## Styling for Grafana

Personally, I run this as a panel on Grafana, with the [ajax/iframe panel](https://grafana.com/plugins/ryantxu-ajax-panel). Adding the following styles will make the result easier on the eyes:

```html
<style>
  body {
    font-family: Roboto, Helvetica, Arial, sans-serif;
    color: rgb(216, 217, 218);
    font-size: 58.5px;
    font-weight: 500;
    text-align: center;
    padding-top: 10px;
  }
</style>
<link
  href="https://fonts.googleapis.com/css?family=Roboto"
  rel="stylesheet"
/>
```

![image](https://user-images.githubusercontent.com/560055/56206485-30e03f00-6055-11e9-94c9-cdb9d07eb5c7.png)

## Training wheels for ancient browsers

If you're running this on a web kiosk with less-than-recent JS engine, the following can help:

```html
<script crossorigin="anonymous" src="https://polyfill.io/v3/polyfill.min.js?features=fetch"></script>
```

Also, you'll probably need to pass the JS through e.g. [Babel's REPL](https://babeljs.io/en/repl) to compile it down to `es2015`.
