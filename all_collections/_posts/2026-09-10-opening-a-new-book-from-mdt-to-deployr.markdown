---
id: 145
title: 'Opening a new book: from MDT to DeployR'
date: '2026-09-10T20:15:00+01:00'
author: Thomas
excerpt: 'It has been quiet here for a while. A change of employer, a new set of challenges, and a device image process that needed rebuilding from the ground up. This is the short version of where that landed me: DeployR.'
layout: post
guid: 'https://codeartisanjourney.com/?p=145'
permalink: /deployr/opening-a-new-book-from-mdt-to-deployr/
categories:
  - DeployR
  - OSD
tags:
  - DeployR
  - 2PintSoftware
  - OSD
  - TaskSequence
  - WinPE
---

### It has been a while

Almost two years, to be honest. Not for a lack of ideas, more for a lack of a moment where an idea felt finished enough to share. I kept telling myself I would write that one nice article and post it. Then work happened.

In the meantime I changed employers and took on a new set of challenges, and one of them turned out to be exactly the kind of thing worth writing about: designing a new device image process for the organization I now work at.

That process led me back to a product I had bumped into before but never got around to acting on: **DeployR** by [2Pint Software](https://2pintsoftware.com/products/deployr).

So before anything else, a shoutout to the 2Pint Software team, and to Master Inventor [Michael Niehaus](https://2pintsoftware.com/news/details/you-say-mic-drop-we-say-mike-drop---welcome-michael-niehaus) - the person a lot of us have been quietly relying on since the MDT and Autopilot days. Seeing that name attached to a next-generation deployment product is a decent hint that it is worth a look.

### Why move away from MDT?

MDT has been the workhorse for a very long time, mine included. But the reasons to move on stack up quickly:

- **It is done.** MDT is no longer being developed or supported. Every new Windows release is a "will it still work?" moment.
- **The image is a maintenance job.** A captured `.wim` starts aging the day you build it, and the servicing treadmill never really stops.
- **The tooling shows its age.** A local console, share permissions, and a deployment share that only one person really understands.
- **No content story.** Getting bits to a remote site is somebody else's problem to solve, usually with more hardware.

DeployR keeps the part that made MDT good - a real task sequence engine with steps, groups, variables and conditions - and drops the parts that made it painful. It is web-based, peer-to-peer content distribution is native rather than bolted on, and, the part that made me sit up, it can build the OS straight from the cloud instead of from an image you maintain yourself.

### Booting with 2PXE

The starting point is still a PXE boot, handled here by 2PXE, which presents the available WinPE images:

![The 2PXE boot menu offering the available WinPE images](https://raw.githubusercontent.com/ThomasKlijnman/thomasklijnman.github.io/main/_images/2026/09/DeployR-PXE-Boot-Menu.png?resize=640%2C480)

Nothing exotic - pick the boot image, and WinPE comes up with the DeployR client waiting for a task sequence.

### A task sequence from a template

You do not start from a blank page. In the Task sequences node you use **+ Add** and pick **Add from template**, choose a template and hit **Clone**. DeployR creates the task sequence and a first version as a copy of that template, and from there you flip on **Edit mode** and adjust it.

The template I went for is **Windows bare metal from cloud**: the same as the regular bare metal template, except that it pulls the operating system from Windows Update and the driver packs directly from the OEM. My clone of it is the `Baremetal-Win11-25h2` sequence below, sitting in the step that does the interesting work - **Apply OS from Cloud**:

![The DeployR task sequence list running Baremetal-Win11-25h2 on the Apply OS from Cloud step](https://raw.githubusercontent.com/ThomasKlijnman/thomasklijnman.github.io/main/_images/2026/09/DeployR-TaskSequence-ApplyOSFromCloud.png?resize=640%2C480)

Note what it says it is downloading: *Windows 11 Enterprise 25H2*. There is no `.wim` on a share anywhere in this picture.

### What actually happens under that step

This is the part I found genuinely satisfying. Open the log viewer while the step runs and you can follow the whole chain.

First the step sets its variables and hands off to a script called `ApplyWU.ps1`:

    Setting OS = 'Windows 11' (Private), was ''
    Setting VERSION = '25H2' (Private), was ''
    Setting EDITION = 'Enterprise' (Private), was ''
    Setting LANGUAGECODE = 'nl-nl' (Private), was ''

Then those variables become a query against the 2Pint Software API, which answers with a download URI - and that URI points straight at Microsoft:

    URL: https://api.service.2pintsoftware.com/cloudos/search?os=Windows 11&osReleaseId=25H2&osArchitecture=...
    Downloading from: http://dl.delivery.mp.microsoft.com/filestreamingservice/files/4841b06c-9a95-411a-9e3e...
    Starting download to file S:\_2P\content\OSWIM\26200.9168.260809-0632.25h2_ge_release_svc_refresh_CLIENT...

![The DeployR log viewer showing the 2Pint cloudos API query and the resulting Microsoft download URI](https://raw.githubusercontent.com/ThomasKlijnman/thomasklijnman.github.io/main/_images/2026/09/DeployR-TaskSequence-LogViewer-CloudOS-API.png?resize=640%2C480)

So the 2Pint API is the catalogue, not the content. It resolves "Windows 11, 25H2, Enterprise, nl-nl, x64" to a specific build, and the bits themselves come down from `dl.delivery.mp.microsoft.com` - Microsoft's own content delivery endpoint, the same one Windows Update and Delivery Optimization use. That answers the question I had going in: it really is Microsoft's image, not a repackaged one. The filename it lands under gives it away too - `26200.9168.260809-0632.25h2_ge_release_svc_refresh_CLIENT...` is a Microsoft build and branch string, straight from the servicing pipeline.

One more line worth pointing at:

    BCMon: Availability check returned no data

That is the peer-to-peer layer looking for another machine on the network that already has this content. This was the first device, so it found nothing and went to Microsoft. The next one does not have to.

The end result behaves exactly like the classic **Apply Operating System** step - a Windows image applied to the volume formatted as `S:`, with an optional `unattend.xml`. What changed is everything before that: no capture, no golden image, no monthly servicing ritual.

### Do not skip the branding

One thing I would not leave out, however tempting it is to call it cosmetic: brand your boot media. Look at the bottom right of every screenshot in this post - that is our own artwork, blurred here, sitting on the 2PXE menu and on the WinPE background behind the task sequence window. It is the difference between "is this thing legitimate?" and a service desk that trusts what it is looking at, and it costs about ten minutes.

Straight from the [2Pint documentation](https://documentation.2pintsoftware.com/deployr/getting-started/generate-windows-pe-boot-images/brand-winpe-background), the short version for the WinPE background:

1. Create your background image and name it **`winpe.jpg`**.
2. Build a source folder that mirrors the target path: `Content Root\Windows\System32\winpe.jpg`. Anything you put in those folders gets copied into WinPE into the same folder structure.
3. Create a Content Item for it (tick *open after creation*, save, then **New version**) and upload the files from your source root.
4. Set that Content Item as the **Extra files** content item on your Boot Image.
5. Click **Generate Image** and wait for it to finish.

Credit where it is due - the 2Pint docs are genuinely good. Short pages, no fluff, and they tell you the exact folder structure instead of making you guess it. That is rarer than it should be.

### Opening a new book

This post is the shallow end. Rebuilding a device image process touches driver strategy, application layering, Autopilot, content distribution and the boring-but-critical question of what happens when a device boots in a branch office on a bad link.

So consider this a new book opened rather than a single article. I intend to go quite a bit deeper into DeployR and into the wider device image process over the coming posts - and this time with a shorter gap between them.
