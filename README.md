# Private Click Measurement & Attribution Reporting module for Ingestly

As of 22nd July, 2021, this module only supports Safari's Private Click Measurement.
Chrome's Attribution Reporting will be supported soon.


## Prerequisite

You must have your own Ingestly Endpoint.
See the guide for [Ingestly Endpoint](https://github.com/ingestly/ingestly-endpoint).

## Installation (Endpoint)

### 1. Create a table for this module on BigQuery

- Use a table schema in `./BigQuery/table_schema`

### 2. Create a new logging cndition on Fastly

On Fastly console, open "Conditions" menu then add a request condition by copy-and-paste below code.

```vcl
(req.url ~ "^/\.well-known/(attribution-reporting|private-click-measurement)/.*")
```

### 3. Create a new logging endpoint on Fastly

- Go to "Logging" then create a new BigQuery endpoint.
- Usually you can duplicate settings from the existing logging configuration for Ingestly Endpoint.
- Specify a table name for this module (must be matched with a table name you created as #1)
- Copy & paste a log format from `./BigQuery/log_format` to the Fastly console.

## Installation (Client)

In this scenario,
- "Referrer" is a news media website running Ads business. (Publisher)
- "Destination" is an E-commerce website who is a customer of "Publisher". (Advertiser)

### 1. Add attributes to the link tag on the Referrer side website.

On the web page of Referrer side, add `attributionSourceId`, `attributeon` and `attributionDestination` to a link to the Destination.
If your (eferrer side) domain is `report.destination`, and your client is `link.destination`:
    - `attributeon` should be `https://link.destination` (same with link destination)
    - `attributionDestination` should be `https://report.destination` (same with your website)

```html
<a href="https://link.destination/"

attributionSourceId="123"
attributeon="https://link.destination"
attributionDestination="https://report.destination"

attributionsourceeventid="123"
attributionreportto="https://report.destination"

>A link to the destination</a>
```
### 2. Place a pixel tag to a "conversion" page on the destination website 

On the web page of Destination side, add the following pixel tag to trigger the PCM reporting scheduling event.

The PCM requires a "pixel" request for triggerring a PCM reporting.
The pixel request must be redirected to a "well-known" structured URL with HTTP 302.

So, use the following pixel tag.

```html
<script>
    (function (w, e, pd, pp) {
        var d = window[w].document,
            i = d.getElementsByTagName('script')[0],
            t = d.createElement('img');
            t.style = 'visibility:hidden;';
        t.async = true;
        t.src = 'https://' + e + '/ingestly-pixel/pcm/?trigger_data=' + pd + '&trigger_priority=' + pp;
        i.parentNode.insertBefore(t, i);
    })('self', 'example.com', '00', '00');
</script>
```
You need to modify at least three parameters in the last line.

|Parameter|Example|Description|
|:----|:----|:----|
|Target Window|`self`||
|Hostname|`example.com`||
|Trigger Data|`00`|A trigger event type. You need to define the type ID between 00 to 15, and the value must be 2 digit string.|
|Optional Priority|`00`|A trigger priority. PCM stores a single attribution for each trigger data, so if you want to breakdown a conversion into multiple steps, assign the priority and PCM may keep an attribution with highest priority. A value must be between 00 to 63, and must be 2 digit string.|


