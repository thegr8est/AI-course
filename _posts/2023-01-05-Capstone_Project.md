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
而DQN想要達成的目標就是訓練一種policy，而這個policy可以讓discounted cumulative reward最大化<br>
![](https://github.com/thegr8est/AI-course/blob/gh-pages/images/cumulative.PNG?raw=true)<br>

其中γ是一個介於0到1的常數，因為越早的action對越晚的reward的影響也會最小<br>
如果γ的值設得太小，這會使agent想在更短的時間內獲得更高的reward，而忽略長期的計畫


* **Exploration**<br>
agent所採取的action必須是有隨機性的，隨機性大一點則蒐集到的資料也會比較豐富<br>


* **Value Function**<br>
![](https://github.com/thegr8est/AI-course/blob/gh-pages/images/value.PNG?raw=true)<br>

Value Function的工作就是要去估測某一個agent在某一個時間點他接下來的discounted cumulative reward會是多少<br>
而不同的agent在同樣的時間點所得的Value Function不一定會相同<br>
有兩種可以訓練Value Function的方法，分別為Monte Carlo和Temporal Difference
![](https://github.com/thegr8est/AI-course/blob/gh-pages/images/MCTD.PNG?raw=true)<br>


* Monte Carlo<br>
![](https://github.com/thegr8est/AI-course/blob/gh-pages/images/monte.PNG?raw=true)<br>

agent在Sa的環境下所產生的cumlative reward為G'a，而Value Function要和G'a越接近越好<br>


* Temporal Difference<br>
![](https://github.com/thegr8est/AI-course/blob/gh-pages/images/temporal.PNG?raw=true)<br>

在St環境下產生的Value Function減掉在S(t+1)的環境下產生的Value Funciton乘上γ要和rt越接近越好<br>


* **Action-Value function**<br>
在得出了Value Function後，接下來我們再把action給考慮進來，計算每個action對應reward所獲得的價值，就可以定義出Action-Value function<br>
![](https://github.com/thegr8est/AI-course/blob/gh-pages/images/actionvalue.PNG?raw=true)<br>

然而我們在上面的公式上有看到一個π，上面有提到，Policy指的是從state跟reward中找到一個最好的action，所以上面的公式所代表的就是在策略π下的動作價值函數<br>


* **Optimal value function**<br>
強化學習最重要的點在於找到一個最好的Policy，所以最好的動作價值函數就是在所有策略下的動作價值函數的最大值
![](https://github.com/thegr8est/AI-course/blob/gh-pages/images/optimal.PNG?raw=true)<br>

這個公式代表最優的動作價值函數


### DQN的特色Experience replay

對於網路輸入，DQN 算法是把整個遊戲的像素作為神經網路的輸入<br>
![](https://github.com/thegr8est/AI-course/blob/gh-pages/images/experience.PNG?raw=true)<br>

用一塊內存空間D，用來儲存每次探索獲得數據![](https://github.com/thegr8est/AI-course/blob/gh-pages/images/text.PNG?raw=true)<br>
利用經驗回放，可以充分發揮off-policy 的優勢，behavior policy 用來蒐集經驗數據，而target policy 只專注於價值最大化<br>


---
### 程式碼
https://www.kaggle.com/code/thegr8est/stock-dqn <br>


* **創建Agent**
```
class DQNAgent:
    def __init__(self, state_size, is_eval=False, model_name=""):
        self.state_size = state_size # normalized previous days
        self.action_size = 3 # sit, buy, sell
        self.memory = deque(maxlen=1000)
        self.inventory = []
        self.model_name = model_name
        self.is_eval = is_eval
        self.gamma = 0.95
        self.epsilon = 1.0
        self.epsilon_min = 0.01
        self.epsilon_decay = 0.995
        self.model = load_model(model_name) if is_eval else self._model()
 ```
 初始化一些基本的agent常數，確保整個股票買賣過程並保持參數穩定<br>
 
 
 ---
 * **定義基本函式**
 ```
 def formatPrice(n):
    return("-Rs." if n<0 else "Rs.")+"{0:.2f}".format(abs(n))

def getStockDataVec(symbol):
    vec = []
    dat = []
    lines = open('/kaggle/input/stocks/'+symbol+".csv","r").read().splitlines()
    for line in lines[1:]:
        vec.append(float(line.split(",")[4]))
        dat.append(line.split(",")[0])
    return vec, dat

def sigmoid(x):
    return 1/(1+math.exp(-x))

def getState(data, t, n):
    d = t - n + 1
    block = data[d:t + 1] if d >= 0 else -d * [data[0]] + data[0:t + 1] # pad with t0
    res = []
    for i in range(n - 1):
        res.append(sigmoid(block[i + 1] - block[i]))
    return np.array([res])
```
formatPrice()定義貨幣的格式<br>
getStockDataVec()將股票的資料轉乘python<br>
sigmoid()用於數學計算<br>
getState()用來表示資料現在的狀態


---
* **訓練Agent**
```
for e in range(num_episode):
    print("Episode " + str(e+1) + "/" + str(num_episode))
    state = getState(data, 0, window_size + 1)
    total_profit = 0
    agent.inventory = []
    for t in range(l):        
        action = agent.act(state)
        # sit
        next_state = getState(data, t + 1, window_size + 1)
        reward = 0
        if action == 1: # buy
            agent.inventory.append(data[t])
            print(date[t]+" Buy: " + formatPrice(data[t]))
        elif action == 2 and len(agent.inventory) > 0: # sell
            bought_price = window_size_price = agent.inventory.pop(0)
            reward = max(data[t] - bought_price, 0)
            total_profit += data[t] - bought_price
            print(date[t]+" Sell: " + formatPrice(data[t]) + " | Profit: " + formatPrice(data[t] - bought_price))
        done = True if t == l - 1 else False
        agent.memory.append((state, action, reward, next_state, done))
        state = next_state
        if done:
            print("--------------------------------")
            print("Total Profit: " + formatPrice(total_profit))
            print("--------------------------------")
        if len(agent.memory) > batch_size:
            agent.expReplay(batch_size)
    if e % 15 == 0:        
        save_model(agent.model, model_name)
    save_model(agent.model, model_name)
```
根據模型預測的操作，買入/賣出調用會增加或減少資金。通過多個episode。隨後保存模型。


---
### 測試結果

![](https://github.com/thegr8est/AI-course/blob/gh-pages/images/result.PNG?raw=true)<br>

<br>
<br>

*This site was last updated {{ site.time | date: "%B %d, %Y" }}.*

