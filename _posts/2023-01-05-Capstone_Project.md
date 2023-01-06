---
layout: post
title: Capstone Project
author: [葉軒宇]
category: [期末專題]
---

## 期末專題 -- Stock DQN


### Reinforcement Learning(強化學習)
![](https://github.com/thegr8est/AI-course/blob/gh-pages/images/RL.PNG?raw=true)<br>
這是強化學習的概念圖，而其中腦袋表示agent，地球表示environment，reward表示agent做出一個action後environment所提供的回饋<br>
agent會根據現在的做出一個相對應的action，則這一連串的action稱為policy<br>
reward的值是由操作者定義的，且reward在強化學習中扮演很重要的一環<br>

強化學習的目標就是將最後reward的值越高越好<br>
---

### Deep Q-Learning(DQN)
DQN 是由兩個龐大體系所組成的，第一個是強化式學習，另一個就是深度學習<br>
而DQN想要達成的目標就是訓練一種policy，而這個policy可以讓cumulative reward最大化<br>
![](https://github.com/thegr8est/AI-course/blob/gh-pages/images/cumulative.PNG?raw=true)<br>
其中γ是一個介於0到1的常數，因為越早的action對越晚的reward的影響也會最小


<br>
<br>

*This site was last updated {{ site.time | date: "%B %d, %Y" }}.*

