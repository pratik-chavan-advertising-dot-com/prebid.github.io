---
layout: page_v2
title: GAM Step by Step - Native Creatives
head_title: GAM Step by Step - Native Creatives
description: Set up native creatives for Prebid in Google Ad Manager.
sidebarType: 3
---

# GAM Step by Step - Native Creatives
{: .no_toc}

- TOC
{:toc}

## Overview

This page walks you through the steps required to set up native ads in GAM for these scenarios:

- Create a native ad template within the Google Ad Manager (GAM) native design tool.
- Prebid Mobile In-App native

You don't necessarily need to use GAM's native design tool - if you're using Prebid.js and have chosen to use
in-adunit or external native templates, then you can just follow the
instructions for [banner creatives with the PUC](/adops/gam-creative-banner-sbs.html).

{: .alert.alert-success :}
For complete instructions on setting up Prebid line items in Google Ad Manager, see [Google Ad Manager with Prebid Step by Step](/adops/step-by-step.html).

For more information about Google Ad Manager native ad setup, see the [Google Ad Manager native ads](https://support.google.com/admanager/answer/6366845) documentation.

## Create a Native Ad Template

1. In GAM, select **Delivery** > **Native**.
2. Click **New native ad** and select **Single ad**.

{: .alert.alert-info :}
For information on the Multiplex ad option, see the [Traffic Multiplex ads](https://support.google.com/admanager/answer/9428537) GAM documentation.

{:start="3"}
3. Under **HTML & CSS editor**, click **Select**. This will slide out the **New native style** window.

### Define Settings for Native Format

1. Under **Ad size** you can select a specific size for the ad unit or specify the "fluid" size.  In this case we'll go with **Fluid**.
2. Under **Custom format**, select **New format**. (If you’ve already created an ad unit with the format you want, you can select **Existing format** and select the format to apply to this ad unit.)
3. Enter a **Format name**, such as `pb-native-fluid`.
4. Click **Add variable**. This will slide out the **New variable** window.

Every format needs at least one variable. Don't worry, you can add more later. GAM requires at least one variable in order to move on to the next step.

{:start="5"}
5. Type in a **Variable name**. In this example, we've used the name `title`.
6. Click **OK**.

![Native adunit settings](/assets/images/ad-ops/gam-sbs/gam-native-format.png){: .pb-md-img :}

{:start="7"}
7.Click **Continue**.

### Style Your Native Ad

1. The first step in styling your native ad is to add the HTML and CSS to define your native ad template. To allow for trackers, titles, images, and other assets within a Prebid native creative template, you’ll need to include a CDN-hosted script in the HTML.

![native ad styling](/assets/images/ad-ops/gam-sbs/gam-native-template.png){: .pb-xlg-img :}

{: .alert.alert-warning :}
Any link that needs to fire a click tracker must include `class='pb-click'`.

If this creative is served, it will fire impression trackers on load. Clicking the link will fire the click tracker and the link will work as normal, in this case going to the `hb_native_linkurl` destination.

The creative template HTML will depend on which of the three scenarios you're implementing. You can choose to manage the native template in one of these ways:

- in GAM ([Managing the Native Template in GAM](#managing-the-native-template-in-gam) below)
- in the Prebid.js AdUnit - this is handled as a 3rd party HTML creative using the Prebid Universal Creative.
- in a separate JavaScript file - this is handled as a 3rd party HTML creative using the Prebid Universal Creative.

{: .alert.alert-info :}
For engineering instructions, see [Native Implementation Guide](/prebid/native-implementation.html).

{:start="2"}
2. After entering your HTML and CSS per the approriate instructions below, click **Continue**.
3. Add any targeting you want to apply and click **Save and activate** (or **Save** if you're not yet ready to activate your template.)
4. Provide a **Name** for your native style and click **Save**.

#### Managing the Native Template in GAM

There are three key aspects of the native template:

1. Build the creative with special Prebid.js macros, e.g. `##hb_native_assetname##`. Note that macros can be placed in the body (HTML) and/or head (CSS) of the native creative.
2. Load the Prebid.js native rendering code. You can utilize the jsdelivr version of native.js or host your own copy. If you use the version hosted on jsdelivr, make sure to declare jsdelivr as an ad technology provider in GAM. (Go to **Privacy & messaging** and click the Settings icon under **GDPR**. Under **Review your ad partners** click into **Commonly used ad partners**.) See Step 6 under [Create a New Native Creative](#create-a-new-native-creative) below.
3. Invoke the Prebid.js native rendering function with an object containing the following attributes:
    1. adid - Used to identify which Prebid.js creative holds the appropriate native assets.
    2. pubUrl - The URL of the page, which is needed for the HTML postmessage call.
    3. requestAllAssets - Tells the renderer to get all the native assets from Prebid.js.

Example creative HTML:

```html
<div class="sponsored-post">
  <div class="thumbnail" style="background-image: url(##hb_native_image##);"></div>
  <div class="content">
    <h1><a href="%%CLICK_URL_UNESC%%##hb_native_linkurl##" target="_blank" class="pb-click">##hb_native_title##</a></h1>
    <p>##hb_native_body##</p>
    <div class="attribution">##hb_native_brand##</div>
  </div>
</div>
<script src="https://cdn.jsdelivr.net/npm/prebid-universal-creative@latest/dist/%%PATTERN:hb_format%%.js"></script>
<script>
    var pbNativeTagData = {};
    pbNativeTagData.pubUrl = "%%PATTERN:url%%";
    pbNativeTagData.adId = "%%PATTERN:hb_adid%%";
    // if you're using 'Send All Bids' mode, you should use %%PATTERN:hb_adid_BIDDERCODE%%;
    pbNativeTagData.requestAllAssets = true;
    window.ucTag.renderAd(document, pbNativeTagData);
</script>
```

{: .alert.alert-warning :}
When using Send All Bids, use `pbNativeTagData.adId = "%%PATTERN:hb_adid_BIDDERCODE%%";` rather than `pbNativeTagData.adId = "%%PATTERN:hb_adid%%";` for each bidder’s creative, replacing `BIDDERCODE` with the actual bidder code, such as `%%PATTERN:hb_adid_BidderA%%`.

Example CSS:

```css
.sponsored-post {
    background-color: #fffdeb;
    font-family: sans-serif;
    padding: 10px 20px 10px 20px;
}

.content {
    overflow: hidden;
}

.thumbnail {
    width: 120px;
    height: 100px;
    float: left;
    margin: 0 20px 10px 0;
    background-image: url(##native_image##);
    background-size: cover;
}

h1 {
    font-size: 18px;
    margin: 0;
}

a {
    color: #0086b3;
    text-decoration: none;
}

p {
    font-size: 16px;
    color: #444;
    margin: 10px 0 10px 0;
}

.attribution {
    color: #fff;
    font-size: 9px;
    font-weight: bold;
    display: inline-block;
    letter-spacing: 2px;
    background-color: #ffd724;
    border-radius: 2px;
    padding: 4px;
}
```

## Create a New Native Creative

Now that you've defined your native template you can create your native creatives.

1. Select **Display** > **Creatives** and click **New creative**.
2. Select your advertiser.
3. Under **Native Format** select the native template you just created and click **Continue**.

![Native Format](/assets/images/ad-ops/gam-sbs/native-format.png){: .pb-md-img :}

{:start="4"}
4. Under **Settings**, enter a **Name** for your creative.
5. Enter any value into the **Click-through URL** field; this value will be overwritten by the native asset values. Also, if you operate in Europe and are using the jsdelivr-hosted native.js, make sure you set jsdelivr as your ad technology provider. (See Step 6 below.)

![Native Creative](/assets/images/ad-ops/gam-sbs/gam-new-creative-part2.png){: .pb-md-img :}

{:start="6"}
6. If you're using jsdelivr, set your **Associated ad technology provider**:

{% include /adops/adops-creative-declaration.html %}

{:start="7"}
7. Click **Save and preview**.

## Create Mobile In-App Template

Use these instructions if you integrate In-App native ads on [iOS](/prebid-mobile/pbm-api/ios/ios-sdk-integration-gam-original-api.html#in-app-native) or [Android](/prebid-mobile/pbm-api/android/android-sdk-integration-gam-original-api.html#in-app-native). The difference is in choosing the GAM option for supporting Android & iOS app code.

1. Sign in to Google Ad Manager.
2. Create an ad unit with fluid ad size.
3. Click `Delivery` and then `Native`
4. Click `New native style`.
5. Click `Android & iOS app code`.
6. Name your new format.
7. Choose `Add variable` and add the following variable names and placeholders as type `text`.

{: .table .table-bordered .table-striped }
| Variable Name       | Placeholder             | Type |
|---------------------+-------------------------+------|
| isPrebid            | [%isPrebid%]            | Text |
| hb_cache_id_local   | [%hb_cache_id_local%]   | Text |

Make sure to indicate that the variables are required.

{:start="8"}
8. Hit "Save".
9. Return to the home screen, click `Delivery > Creatives`, and create a creative with `Native Format`, choosing the format you created.
10. Choose a creative name and other desired settings. In the user-defined variables you just created, set the following values:

{: .table .table-bordered .table-striped }
| Variable Name       | Value                            |
|---------------------+----------------------------------|
| isPrebid            | 1                                |
| hb_cache_id_local   | %%PATTERN:hb_cache_id_local%%    |

{:start="11"}
11. Create Prebid line items with price priority and a display ad type that is targeting `hb_pb key-values`. Associate the creative you added in steps 4 thru 8 (making sure to choose your native format as expected creatives on the line item) to the ad unit you created in the second step.

## Attach the Creative to Your Line Item

Follow the instructions in [Google Ad Manager with Prebid Step by Step](/adops/step-by-step.html#duplicate-creative) to duplicate your creative and attach it to your line item.

## Further Reading

- [Google Ad Manager with Prebid Step by Step](/adops/step-by-step.html)
- [Prebid Native Implementation Guide](/prebid/native-implementation.html)
- [Send All Bids vs Top Price](/adops/send-all-vs-top-price.html)
- [Prebid Universal Creative](/overview/prebid-universal-creative.html)
- [Creative Considerations](/adops/creative-considerations.html)
- [Ad Ops Planning Guide](/adops/adops-planning-guide.html)
