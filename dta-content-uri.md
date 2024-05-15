- https://dta.collegeboard.org/api/v1/content
  Returns the Translations(?) and textual content for in-app instructions
  i.e The directions to each exam, the lockdown instructions, etc

This is also significantly interesting due to the CSP

```json
default-src 'self' _.collegeboard.org; script-src 'self' _.collegeboard.org cdnjs.cloudflare.com sdk.amazonaws.com assets.adobedtm.com cdn.cookielaw.org bat.bing.com www.clarity.ms d.clarity.ms 'unsafe-inline' 'unsafe-eval' www.googletagmanager.com googleads.g.doubleclick.net www.googleadservices.com connect.facebook.net analytics.tiktok.com cdn.heapanalytics.com widgets.getsitecontrol.com www.youtube.com _.salesforceliveagent.com pixel.admedia.com pixel.s3xified.com service.force.com s.yimg.com connect.facebook.net ajax.cloudflare.com st.getsitecontrol.com js-agent.newrelic.com bam.nr-data.net d10lpsik1i8c69.cloudfront.net s3.amazonaws.com/cdn.aimtell.com/ sc-static.net js.adsrvr.org match.adsrvr.org www.google.com client.rum.us-east-1.amazonaws.com/1.0.2/cwr.js tpc.googlesyndication.com cdn.aimtell.com static.lightning.force.com _.my.salesforce.com _.my.salesforce-sites.com apform.secure.force.com conoret.com ucads-cdn.ucweb.com www.google-analytics.com www.pagespeed-mod.com bytedance.com sp.analytics.yahoo.com static.jungroup.com trkn.us serve.uberads.com _.stackadapt.com cdn.ckeditor.com cdnjs.cloudflare.com/ajax/libs/cropper/4.0.0/cropper.min.js assets.calendly.com platform.twitter.com _.appcues.com _.appcues.net; style-src 'self' _.collegeboard.org 'unsafe-inline' service.force.com translate.googleapis.com use.fontawesome.com apform.secure.force.com _.my.salesforce-sites.com cdn.tt.omtrdc.net/cdn/adobetarget/admin.css d10lpsik1i8c69.cloudfront.net/css/reset.css fonts.googleapis.com cdn.ckeditor.com cdnjs.cloudflare.com/ajax/libs/cropper/4.0.0/cropper.min.css _.stackadapt.com wiris-v7.hive-prod.collegeboard.org:80 wiris-v7.hive-nonprod.collegeboard.org:80 _.appcues.com _.appcues.net fonts.googleapis.com fonts.google.com 'unsafe-inline'; img-src 'self' _.collegeboard.org data: bat.bing.com www.facebook.com www.google.com _.doubleclick.net googleads.g.doubleclick.net _.clarity.ms _.heapanalytics.com app.getsitecontrol.com _.analytics.yahoo.com _.bing.com heapanalytics.com www.googletagmanager.com www.google.co.jp www.google.ca www.googletagmanager.com www.google.co www.google.com www.google.jo translate.google.com ssl.google-analytics.com d10lpsik1i8c69.cloudfront.net adservice.google.com _.appcues.com _.appcues.net res.cloudinary.com twemoji.maxcdn.com _; frame-src 'self' _.collegeboard.org www.surveygizmo.com bid.g.doubleclick.net googleads.g.doubleclick.net service.force.com js.adsrvr.org match.adsrvr.org beacon.aimtell.com tr.snapchat.com tpc.googlesyndication.com datacloudstat.com www.facebook.com www.youtube.com ws-lmdc-app03.dhs.state.nj.us gateway.zscloud.net mozbar.moz.com s3.amazonaws.com/cdn.aimtell.com/ _.id.opendns.com lsrelay-config-production.s3.amazonaws.com pg-sasscer-ckf04.pgcps.org static.deledao.com data: schools-blocked.s3-website-us-east-1.amazonaws.com calendly.com platform.twitter.com _.appcues.com credentialfinder.org apps.credentialengine.org _.webcasts.com; frame-ancestors 'self' credentialfinder.org; font-src 'self' _.collegeboard.org themes.googleusercontent.com fonts.gstatic.com data: st.getsitecontrol.com moz-extension: use.fontawesome.com static3.avast.com at.alicdn.com cdn.loom.com/assets/fonts/ wiris-v7.hive-prod.collegeboard.org:80 wiris-v7.hive-nonprod.collegeboard.org:80 cdnjs.cloudflare.com/ajax/libs/mathjax/3.2.2/es5/output/chtml/fonts/woff-v2/ fonts.gstatic.com; connect-src 'self' ws: _.collegeboard.org k625k2vrzvdo5g7ynbvtjejehi.appsync-api.us-east-1.amazonaws.com/graphql dgtkl2ep7natjmkbefhxflglie.appsync-api.us-east-1.amazonaws.com/graphql cdn.cookielaw.org geolocation.onetrust.com www.facebook.com analytics.tiktok.com _.clarity.ms bat.bing.com app.getsitecontrol.com lambda.us-east-1.amazonaws.com signals.aimtell.com bam.nr-data.net settings.luckyorange.net cdn.aimtell.io log.aimtell.com s.yimg.com cognito-identity.us-east-1.amazonaws.com dataplane.rum.us-east-1.amazonaws.com sts.us-east-1.amazonaws.com beacon.aimtell.com adservice.google.com www.google.com api.ultimateaderaser.com privacyportal.onetrust.com adtonus.com apform.secure.force.com cdnm3.cdnservice.space/start5.json code.jquery.com gjtrack.ucweb.com/collect heapanalytics.com log.kslogs.ru/timesince plugin.ucads.ucweb.com/api rdtds.net/siblings/find stats.g.doubleclick.net www.google-analytics.com api.trongrid.io/wallet/getnodeinfo dgtkl2ep7natjmkbefhxflglie.appsync-api.us-east-1.amazonaws.com get663.com support.adcleanerpage.com tr.snapchat.com hm.baidu.com/hm.gif dgtkl2ep7natjmkbefhxflglie.appsync-realtime-api.us-east-1.amazonaws.com analytics.aimtell.com sts.us-west-2.amazonaws.com cognito-identity.us-west-2.amazonaws.com d1ktxyteejjrbw.cloudfront.net static.doubleclick.net full-apform.cs190.force.com yt3.ggpht.com cdn.mouseflow.com n2.mouseflow.com collegeboard-full.my.salesforce.com i.ytimg.com cdn.ckeditor.com _.stackadapt.com telemetry.wiris.net wiris-v7.hive-prod.collegeboard.org:80 wiris-v7.hive-nonprod.collegeboard.org:80 _.appcues.com _.appcues.net \*.my.salesforce-sites.com ipapi.co 9frgh2i4b9.execute-api.us-east-1.amazonaws.com
```

A **_significant_** number of these are telemetry sites, but some are interesting, including:

- `cognito-identity.us-east-1.amazonaws.com` Cognito URI
- `schools-blocked.s3-website-us-east-1.amazonaws.com` ???
- `lsrelay-config-production.s3.amazonaws.com pg-sasscer-ckf04.pgcps.org`
- `yt3.ggpht.com` ????
- `dgtkl2ep7natjmkbefhxflglie.appsync-realtime-api.us-east-1.amazonaws.com`
- `sts.us-west-2.amazonaws.com` (see [Authentication](authentication.md))
- `cognito-identity.us-west-2.amazonaws.com`
- `d1ktxyteejjrbw.cloudfront.net`
- `9frgh2i4b9.execute-api.us-east-1.amazonaws.com`
- `ipapi.co`
- `k625k2vrzvdo5g7ynbvtjejehi.appsync-api.us-east-1.amazonaws.com/graphql`
- `dgtkl2ep7natjmkbefhxflglie.appsync-api.us-east-1.amazonaws.com/graphql`

(The graphQL endpoints are unknown, since https://dap-mc2-graphql-us-east-1.collegeboard.org/graphql is a CNAME over to dap-mc2-graphql-us-east-1.collegeboard.org.edgekey.net)

And finally, "wiris" is a [SaaS company providing...math embedding services?](https://en.wikipedia.org/wiki/WIRIS)

And some don't make sense:

- "get663.com"
- "support.adcleanerpage.com"
