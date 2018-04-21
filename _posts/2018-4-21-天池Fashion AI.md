---
layout: post
title: 天池Fashion AI 比赛失败分享。
category: Machine Learning
description: I'm that dumb.
---
昨天是天池Fashion AI初赛Deadline， 成绩出来复赛都没能进，虽然结果很遗憾，但在比赛的过程中也接触到了不少的新东西，希望能在这里把我尝试过的方法都分享出来。作为对自己的总结，如果能让看到的人以后少踩点坑，那也算是一丢丢的贡献吧。

由于各种原因，真的着手在这个比赛的时间大概就一个星期，失败也是对自己拖延症的一种惩罚吧。不说废话，下面进入正题。

# 比赛内容
我参加的是Fashion AI 服装关键点定位的比赛，目标就是通过算法找到图片中衣服的各个关键点。具体关键点如下图所示：

[keypoints_demo]({{ site.url }}/assets/Tianchi-Fashion-AI/keypoints-demo.jpg)

详细的赛制和数据可以在这里找到：[FashionAI全球挑战赛——服饰关键点定位](https://tianchi.aliyun.com/competition/introduction.htm?spm=5176.100066.0.0.6acdd780km5qMe&raceId=231648)。

# 算法
