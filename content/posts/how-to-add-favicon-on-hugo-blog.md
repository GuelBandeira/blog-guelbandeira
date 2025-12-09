---
title: "How to add a favicon on a Hugo blog"
date: 2025-12-08
draft: false
series: "Definitive Guide to Hosting a Blog"
tags: ["Hugo", "Favicon"]
---
## Get your icon


After choosing the icon you want to appear on your website, use [CloudConvert](https://cloudconvert.com/png-to-ico) to convert your favicon to .ico. You can also use any other website that converts images to .ico format, there are plenty of options available.

Select the extension of your favicon, choose **ICO**, and click **Convert**:


![Converting favicon](/images/how-to-add-favicon-on-hugo-blog/convert-favicon.png)


## Add your favicon to Hugo

Open your _hugo.toml_ file and add the following lines:
```bash
[params]
    favicon = "/favicon.ico"
```

## Test to see if it works

That’s it! Now run Hugo on your local server and check if your favicon appears in the browser tab, just like mine.

![Favicon working](/images/how-to-add-favicon-on-hugo-blog/favicon-working.png)



