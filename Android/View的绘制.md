# View的绘制

## View树的结构

![](https://images0.cnblogs.com/blog2015/429692/201508/141845331148151.png)

View的绘制是按照View树的结构DecorView-->ViewGroup（--->ViewGroup）-->View从上往下遍历，依次进行绘制的

## View的绘制流程

![](https://images0.cnblogs.com/blog2015/429692/201508/141842167707468.png)

View的绘制流程是从ViewRootImpl类的performTraversals()方法中展开的
从上图可知，View的绘制分为三步：**measure -> layout -> draw**

### measure

### layout

### draw


### 参考链接
https://www.jianshu.com/p/5a71014e7b1b