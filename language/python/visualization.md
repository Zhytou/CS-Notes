# 数据可视化

- [数据可视化](#数据可视化)
  - [Matplolib](#matplolib)
  - [Seaborn](#seaborn)
  - [Bokeh](#bokeh)

## Matplolib

**Simpe Example**:

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

**Matplotlib Backend**:

Matplotlib的绘图结果可以有各种输出形式，例如常见用的将绘图结果嵌入到wxpython、pygtk、Qt等GUI窗体中，或者将绘图结果输出为图片文件，或在Web应用程序中输出绘图结果。

为了便于用户实现这些不同的输出，Matplotlib在设计上对用户编写的绘图代码和对不同输出形式的处理方法进行了隔离，因此出现了前端（frontend）和后端的概念（backend）。后端可以认为就是不同输出形式的处理功能，前端可以认为就是用户所要绘制的图像。

``` python
# 使用tkagg作为后端
plt.switch_backend('tkagg')
```

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

Bokeh是一款交互式可视化库，可以将Python使用处理得到数据结果放在在浏览器上进行展示。事实上，Bokeh是通过Bokeh-JS库来在浏览器中执行交互式地图、直观的用户界面元素等。此外，还有Pandas-Bokeh，它额外提供了一些功能来使Pandas DataFrame更容易地可视化到Bokeh中，即：提供了另一种matplotlib后端。
