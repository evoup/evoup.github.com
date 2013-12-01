---
layout: post
title: "JS跳出框架"
date: 2008-05-19 22:39
comments: true
categories: javascript
---
```javascript
<script language= "JavaScript">
  <!--Break out of frames
    if (top.frames.length > 0)
    top.location=self.document.location;
  //-->
</script>
```