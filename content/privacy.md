# Privacy Details

This site uses cookies and local storage to collect basic information about the visitors here.  The data is sent to a
private [Countly](https://count.ly) server deployed on Digital Ocean.

The information that is collected includes:

 - **Sessions**: Metrics on when, how often and how long users use the website
 - **Page Views**: Metrics on which views/pages users access
 - **Clicks**: Reports user clicks to form a heat map, including clicks following links
 - **Scrolls**: Reports user scrolling to form a scroll heat map
 - **Crashes**: Reports javascript errors thrown by the UI code
 - **GeoIP**: Coarse location information based on IP data (no GPS or other granular data is collected)
 - **UserAgent**: Data set by your browser with system information, including:
   - *Browser*: Web browser type and version
   - *System*: Operating system and version
   - *Resolution*: Screen size of the device

I do not sell these metrics or use them for any advertising purposes.  They are purely for my own reference when it
comes to looking at how people are engaging with the content here.

#### Trust, but Verify

The code which records metrics to the Countly server, as with the entirety of this site, is available on Github if you'd
like to review it.  The specific file which handles the metrics configuration, as well as opt-in and opt-out logic can
be found [here](https://github.com/twrobel3/hugo-theme-hello-friend-ng/blob/master/layouts/partials/countly.html).

And of course, since all that code is running client-side, you can pop open your browser's web inspector and verify that
nothing has been changed from what's shown in the repository.

<hr>

<div id="gdpr-is-opted-in">
<p>
You are currently opted <b>IN</b> to the collection of data as described on this page.  You may opt-out at any time.
</p>
<button type="button" class="button" onclick="removeConsent(900)">Opt-Out</button>
</div>

<div id="gdpr-is-opted-out">
<p>
You are currently opted <b>OUT</b> of the collection of data as described on this page.
<br>No data about your activity here is being recorded.
</p>
<p>
If you'd like, you may opt-in, or write a key to <code>localStorage</code> which will prevent the GDPR banner from
re-appearing at the top of this site after 15 minutes.
</p>
<p>
<button type="button" class="button" onclick="removeConsent(3078000000)">Hide GDPR Notice for this Browser</button>
<button type="button" class="button primary" onclick="giveConsent()">Opt-In</button>
</p>
</div>
