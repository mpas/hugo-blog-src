---
title: "Using DavMail Gateway as a mail proxy for Microsoft Exchange (OWA)"
tags:
  - davmail
  - exchange
  - owa
  - postbox
date: "2015-11-17"
created: "2020-03-09T16:00:00Z"
modified: "2026-04-29T08:03:00Z"
---

If you find yourself into a situation where you have a need for non Microsoft mail client that needs support for Microsoft Exchange then you are often out of luck. In my case I needed Exchange support for the terrific [PostBox](http://www.postbox-inc.com) mail client.

As for now PostBox does not support Microsoft Exhange natively so the hunt starts on how to get Exchange working. As it stands most companies also enable Exchange Web Access (or Outlook Web Access [OWA]) so maybe we can use that to feed our native mail client.

Enter the use of [DavMail](http://davmail.sourceforge.net/)!

#### Davmail Gateway

Davmail is a local mail proxy that can work together with Microsoft Exchange [OWA] in a way that DavMail is actually connecting to a Exchange OWA and your mail client connects to DavMail as a proxy.

<img src="./images/davmail-sequence.png" width="800" class="postimage">

#### Configure Davmail

In order to get DavMail working correctly you need to provide the correct settings so it can use the OWA endpoint.

<img src="./images/davmail-settings-main.png" width="800" class="postimage">

#### Configure PostBox

In order to get PostBox working with DavMail you need to create an outgoing mail server and an account that will use that outgoing mailserver.

#### Configure PostBox - Outgoing mailserver

<img src="./images/postbox-outgoing-mailserver.png" width="400" class="postimage">

#### Configure PostBox - Account setup

<img src="./images/postbox-account-setup.png" width="800" class="postimage">

#### Configure PostBox - Identity setup

<img src="./images/postbox-identity-setup.png" width="800" class="postimage">

Now you are ready to send mail using your PostBox Client using DavMail and OWA.
