# 数据可视化

## Matplolib

```python
from matplotlib import pyplot as plt
import numpy as np
import math
x = np.arange(0, math.pi*2, 0.05)
y = np.sin(x)
plt.plot(x,y)
plt.xlabel("angle")
plt.ylabel("sine")
plt.title('sine wave')
plt.show()
```

事实上，Pandas库中的Series和DataFrame是可以直接调用plot函数的，只不过它们的plot函数也都是基于matplotlib的。

## Seaborn

根据图形的适应场景，Seaborn 的绘图方法大致分类 6 类，分别是：关联图、类别图、分布图、回归图、矩阵图和组合图。

``` python
import seaborn as sns
import numpy as np

sns.heatmap(np.random.rand(10, 10))
```

![output heatmap](https://cdn.huhuhang.com/images/2019/seaborn-basic/output_114_1.png)

Seaborn 中的 API 分为 Figure-level 和 Axes-level 两种。

Figure-level 和 Axes-level API 的区别在于，Axes-level 的函数可以实现与 Matplotlib 更灵活和紧密的结合，而 Figure-level 则更像是「懒人函数」，适合于快速应用。

## Bokeh

Bokeh是一款交互式可视化库，可以将Python使用处理得到数据结果放在在浏览器上进行展示。
