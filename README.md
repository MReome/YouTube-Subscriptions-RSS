# YouTube Subscriptions to RSS

**Use this script to get the RSS feeds of all your YouTube subscriptions. Downloads as an OPML file, which can be imported into your favorite RSS reader.**

## Usage:

1. Navigate to https://www.youtube.com/feed/channels
2. Scroll to the bottom of the page to make sure all channels have been loaded
3. Run the script or activate the bookmarklet. An OPML file will download, and a list of RSS feeds will be logged to the console if you need it.

---

## Script:

Paste this into the console at [https://www.youtube.com/feed/channels](https://www.youtube.com/feed/channels) to download the OPML file:

```javascript
(async () => {
	const dialog = document.createElement("dialog");
	const label = document.createElement("label");
	const progress = document.createElement("progress");
	dialog.style.cssText = "display: flex; flex-direction: column; gap: 15px; padding: 20px;";
	dialog.appendChild(label);
	dialog.appendChild(progress);
	document.querySelector("ytd-app").appendChild(dialog);
	dialog.showModal();
	label.innerText = "Loading subscriptions...";
  const content = document.getElementById("content");
  let contentH;
  do {
    contentH = content.offsetHeight;
    window.scrollBy(0, 100000);
    await new Promise((r) => setTimeout(r, 500));
  } while (
    content.querySelector("#spinnerContainer.active") != null ||
    content.offsetHeight > contentH
  );
	try {
		const channelElements = [
      ...content.querySelectorAll(
        "ytd-browse:not([hidden]) #main-link.channel-link"
      ),
    ];
		progress.max = channelElements.length;
		progress.value = 0;
		const channels = [];
		for (e of channelElements) {
			label.innerText = `Fetching URLS... (${progress.value}/${progress.max})`;
			try {
				const channelName = e.querySelector("yt-formatted-string.ytd-channel-name").innerText;
				const channelReq = await fetch(e.href);
				if (!channelReq.ok) { console.error(`Couldn't fetch channel page for ${channelName}`); continue; }
				const channelPageDoc = new DOMParser().parseFromString(await channelReq.text(), "text/html");
				const links = channelPageDoc.querySelectorAll("body > link[rel=alternate], body > link[rel=canonical]");
				const channelIdMatch = [...links].map(e => e.href.match("/channel/([a-zA-Z0-9_\-]+?)$")).find(e => e != null);
				if (channelIdMatch == null) { console.error(`Couldn't find channel id for ${channelName}`); continue; }
				channels.push([`https://www.youtube.com/feeds/videos.xml?channel_id=${channelIdMatch[1]}`, channelName, e.href]);
			} finally {
				progress.value++;
				progress.replaceWith(progress);
			}
		};
		if (channelElements.length == 0) alert("Couldn't find any subscriptions");
		const missedChannels = channelElements.length - channels.length;
		if (missedChannels > 0) alert(`${missedChannels} channel${missedChannels > 1 ? "s" : ""} couldn't be fetched. Check the console for more info.`);
		const escapeXML = (str)=> str.replace(/[<>&'"]/g, c=>({"<":"&lt;",">":"&gt;","&":"&amp;","'":"&apos;",'"':"&quot;"}[c]))
		if (channels.length > 0) {
			console.log(channels.map(([feed]) => feed).join("\n"));
			let opmlText = `<?xml version="1.0" encoding="UTF-8"?>\n<opml version="1.0">\n\t<head>\n\t\t<title>YouTube Subscriptions as RSS</title>\n\t</head>\n\t<body>\n\t\t<outline text="YouTube Subscriptions">${channels
        .map(
          ([feed, channelName, channelUrl]) =>
            `\n\t\t\t<outline type="rss" text="${escapeXML(channelName)}" title="${escapeXML(channelName)}" xmlUrl="${feed}" htmlUrl="${channelUrl}"/>`
        )
        .join("")}\n\t\t</outline>\n\t</body>\n</opml>`;
			const url = window.URL.createObjectURL(new Blob([opmlText], { type: "text/plain" }));
			const anchorTag = document.createElement("a");
			anchorTag.setAttribute("download", "youtube_subs.opml");
			anchorTag.setAttribute("href", url);
			anchorTag.dataset.downloadurl = `text/plain:youtube_subs.opml:${url}`;
			anchorTag.click();
		}
	} catch (e) {
		console.error(e);
		alert("Something went wrong. Check the console for more info.");
	} finally {
		dialog.close();
		dialog.remove();
	}
})();
```

## Bookmarklet:

You can save this as a bookmarklet to run it in one click. Just create a new bookmark and paste the following into the URL field:

```
javascript:(function()%7B(async%20()%20%3D%3E%20%7B%0A%09const%20dialog%20%3D%20document.createElement(%22dialog%22)%3B%0A%09const%20label%20%3D%20document.createElement(%22label%22)%3B%0A%09const%20progress%20%3D%20document.createElement(%22progress%22)%3B%0A%09dialog.style.cssText%20%3D%20%22display%3A%20flex%3B%20flex-direction%3A%20column%3B%20gap%3A%2015px%3B%20padding%3A%2020px%3B%22%3B%0A%09dialog.appendChild(label)%3B%0A%09dialog.appendChild(progress)%3B%0A%09document.querySelector(%22ytd-app%22).appendChild(dialog)%3B%0A%09dialog.showModal()%3B%0A%09label.innerText%20%3D%20%22Loading%20subscriptions...%22%3B%0A%20%20const%20content%20%3D%20document.getElementById(%22content%22)%3B%0A%20%20let%20contentH%3B%0A%20%20do%20%7B%0A%20%20%20%20contentH%20%3D%20content.offsetHeight%3B%0A%20%20%20%20window.scrollBy(0%2C%20100000)%3B%0A%20%20%20%20await%20new%20Promise((r)%20%3D%3E%20setTimeout(r%2C%20500))%3B%0A%20%20%7D%20while%20(%0A%20%20%20%20content.querySelector(%22%23spinnerContainer.active%22)%20!%3D%20null%20%7C%7C%0A%20%20%20%20content.offsetHeight%20%3E%20contentH%0A%20%20)%3B%0A%09try%20%7B%0A%09%09const%20channelElements%20%3D%20%5B%0A%20%20%20%20%20%20...content.querySelectorAll(%0A%20%20%20%20%20%20%20%20%22ytd-browse%3Anot(%5Bhidden%5D)%20%23main-link.channel-link%22%0A%20%20%20%20%20%20)%2C%0A%20%20%20%20%5D%3B%0A%09%09progress.max%20%3D%20channelElements.length%3B%0A%09%09progress.value%20%3D%200%3B%0A%09%09const%20channels%20%3D%20%5B%5D%3B%0A%09%09for%20(e%20of%20channelElements)%20%7B%0A%09%09%09label.innerText%20%3D%20%60Fetching%20URLS...%20(%24%7Bprogress.value%7D%2F%24%7Bprogress.max%7D)%60%3B%0A%09%09%09try%20%7B%0A%09%09%09%09const%20channelName%20%3D%20e.querySelector(%22yt-formatted-string.ytd-channel-name%22).innerText%3B%0A%09%09%09%09const%20channelReq%20%3D%20await%20fetch(e.href)%3B%0A%09%09%09%09if%20(!channelReq.ok)%20%7B%20console.error(%60Couldn't%20fetch%20channel%20page%20for%20%24%7BchannelName%7D%60)%3B%20continue%3B%20%7D%0A%09%09%09%09const%20channelPageDoc%20%3D%20new%20DOMParser().parseFromString(await%20channelReq.text()%2C%20%22text%2Fhtml%22)%3B%0A%09%09%09%09const%20links%20%3D%20channelPageDoc.querySelectorAll(%22body%20%3E%20link%5Brel%3Dalternate%5D%2C%20body%20%3E%20link%5Brel%3Dcanonical%5D%22)%3B%0A%09%09%09%09const%20channelIdMatch%20%3D%20%5B...links%5D.map(e%20%3D%3E%20e.href.match(%22%2Fchannel%2F(%5Ba-zA-Z0-9_%5C-%5D%2B%3F)%24%22)).find(e%20%3D%3E%20e%20!%3D%20null)%3B%0A%09%09%09%09if%20(channelIdMatch%20%3D%3D%20null)%20%7B%20console.error(%60Couldn't%20find%20channel%20id%20for%20%24%7BchannelName%7D%60)%3B%20continue%3B%20%7D%0A%09%09%09%09channels.push(%5B%60https%3A%2F%2Fwww.youtube.com%2Ffeeds%2Fvideos.xml%3Fchannel_id%3D%24%7BchannelIdMatch%5B1%5D%7D%60%2C%20channelName%2C%20e.href%5D)%3B%0A%09%09%09%7D%20finally%20%7B%0A%09%09%09%09progress.value%2B%2B%3B%0A%09%09%09%09progress.replaceWith(progress)%3B%0A%09%09%09%7D%0A%09%09%7D%3B%0A%09%09if%20(channelElements.length%20%3D%3D%200)%20alert(%22Couldn't%20find%20any%20subscriptions%22)%3B%0A%09%09const%20missedChannels%20%3D%20channelElements.length%20-%20channels.length%3B%0A%09%09if%20(missedChannels%20%3E%200)%20alert(%60%24%7BmissedChannels%7D%20channel%24%7BmissedChannels%20%3E%201%20%3F%20%22s%22%20%3A%20%22%22%7D%20couldn't%20be%20fetched.%20Check%20the%20console%20for%20more%20info.%60)%3B%0A%09%09const%20escapeXML%20%3D%20(str)%3D%3E%20str.replace(%2F%5B%3C%3E%26'%22%5D%2Fg%2C%20c%3D%3E(%7B%22%3C%22%3A%22%26lt%3B%22%2C%22%3E%22%3A%22%26gt%3B%22%2C%22%26%22%3A%22%26amp%3B%22%2C%22'%22%3A%22%26apos%3B%22%2C'%22'%3A%22%26quot%3B%22%7D%5Bc%5D))%0A%09%09if%20(channels.length%20%3E%200)%20%7B%0A%09%09%09console.log(channels.map((%5Bfeed%5D)%20%3D%3E%20feed).join(%22%5Cn%22))%3B%0A%09%09%09let%20opmlText%20%3D%20%60%3C%3Fxml%20version%3D%221.0%22%20encoding%3D%22UTF-8%22%3F%3E%5Cn%3Copml%20version%3D%221.0%22%3E%5Cn%5Ct%3Chead%3E%5Cn%5Ct%5Ct%3Ctitle%3EYouTube%20Subscriptions%20as%20RSS%3C%2Ftitle%3E%5Cn%5Ct%3C%2Fhead%3E%5Cn%5Ct%3Cbody%3E%5Cn%5Ct%5Ct%3Coutline%20text%3D%22YouTube%20Subscriptions%22%3E%24%7Bchannels%0A%20%20%20%20%20%20%20%20.map(%0A%20%20%20%20%20%20%20%20%20%20(%5Bfeed%2C%20channelName%2C%20channelUrl%5D)%20%3D%3E%0A%20%20%20%20%20%20%20%20%20%20%20%20%60%5Cn%5Ct%5Ct%5Ct%3Coutline%20type%3D%22rss%22%20text%3D%22%24%7BescapeXML(channelName)%7D%22%20title%3D%22%24%7BescapeXML(channelName)%7D%22%20xmlUrl%3D%22%24%7Bfeed%7D%22%20htmlUrl%3D%22%24%7BchannelUrl%7D%22%2F%3E%60%0A%20%20%20%20%20%20%20%20)%0A%20%20%20%20%20%20%20%20.join(%22%22)%7D%5Cn%5Ct%5Ct%3C%2Foutline%3E%5Cn%5Ct%3C%2Fbody%3E%5Cn%3C%2Fopml%3E%60%3B%0A%09%09%09const%20url%20%3D%20window.URL.createObjectURL(new%20Blob(%5BopmlText%5D%2C%20%7B%20type%3A%20%22text%2Fplain%22%20%7D))%3B%0A%09%09%09const%20anchorTag%20%3D%20document.createElement(%22a%22)%3B%0A%09%09%09anchorTag.setAttribute(%22download%22%2C%20%22youtube_subs.opml%22)%3B%0A%09%09%09anchorTag.setAttribute(%22href%22%2C%20url)%3B%0A%09%09%09anchorTag.dataset.downloadurl%20%3D%20%60text%2Fplain%3Ayoutube_subs.opml%3A%24%7Burl%7D%60%3B%0A%09%09%09anchorTag.click()%3B%0A%09%09%7D%0A%09%7D%20catch%20(e)%20%7B%0A%09%09console.error(e)%3B%0A%09%09alert(%22Something%20went%20wrong.%20Check%20the%20console%20for%20more%20info.%22)%3B%0A%09%7D%20finally%20%7B%0A%09%09dialog.close()%3B%0A%09%09dialog.remove()%3B%0A%09%7D%0A%7D)()%3B%7D)()%3B
```

---

<br>

![](https://img.shields.io/badge/Safari-FF1B2D?style=for-the-badge&logo=Safari&logoColor=white)
![](https://img.shields.io/badge/Google_chrome-4285F4?style=for-the-badge&logo=Google-chrome&logoColor=white)
![](https://img.shields.io/badge/Firefox_Browser-FF7139?style=for-the-badge&logo=Firefox-Browser&logoColor=white)

_Working in Chrome, Firefox and Safari at last commit date_

_This script relies on YouTube maintaining RSS feeds for each channel, information from `<link>` tags on each channel page and class names on YouTube, so it may well break in the future._
