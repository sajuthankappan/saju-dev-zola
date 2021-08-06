+++
title = "Convert Saju.bike to zola"
description = "Converting saju.bike website from eleventy to zola"
date = 2021-08-03
[taxonomies]
tags =["rust"]
+++

1. Delete eleventy/npm related files at root
```bash
rm package.json package-lock.json .eleventy.js 
```

2. Update .gitignore to reflect zola
```
public
```

3. Create a temporary folder temp and init zola, and follow prompts
```
mkdir temp
cd temp
zola init
```

4. Move all files & folders in temp to root folder, and remove temp folder
```
mv  ./* ../
cd ..
rm -rf ./temp
```

5. Create templates/base.html, based on src/layout.njk

6. Create templates/index.html as below..
```
{% extends "base.html" %}

{% block content %}

<h2 class="title">
  Hello World!
</h2>
{% endblock content %}

```

7. Create folder content/brm and create _index.md
```
+++
title = "BRM"
sort_by = "date"
template = "brm.html"
page_template = "brm-topic.html"
+++
```

8. Copy all files from src/brm/*.njk to content/brm. 

9. Remove the front matter content from all these files in content/brm, and add zola specific front matter content. For example..
```
+++
title = "BRM Checklist"
date = 2021-08-01
+++
```

10. Remove src folder
```
rm -rf src
```

11. Videos page uses amp-youtube component. We can replace this with zola youtube shortcode

Old Code:
```
<amp-youtube
    data-videoid="9EpofVekJx8"
    layout="responsive"
    width="480"
    height="270">
</amp-youtube>
```

New Code: (To be used within braces {{..}})
```
youtube(id="9EpofVekJx8", class="youtube")
```

12. Run zola serve to preview the site
```
zola serve
```

13. All done!