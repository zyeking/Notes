# 大众点评评价爬取逻辑


## 分析
![](https://i.loli.net/2019/07/20/5d32de3e9119579229.jpg)

1. 如图，评价中的`孩`字在源码中时被`<svgmtsi class="cfd1i"></svgmtsi>`标签加密了。

2. 在css中可以看到该svg图片地址为`background-image: url(//s3plus.meituan.net/v1/mss_0a06a471f9514fc79c981b5466f56b91/svgtextcss/2ec02e25ea201ca1b6b415747003614e.svg);`

3. 进入该地址发现是字体文件![](https://i.loli.net/2019/07/20/5d32e0871aac789427.jpg)

4. 加密字体的css中有`background: -0.0px -1808.0px;`，通过搜索可知`x / font-size（svg文件中①） + 1`可以得到该字体在svg中的第几个；比较y坐标和②的比较，其中，取`y<N`中的N值

![](https://i.loli.net/2019/07/20/5d32de3e9119579229.jpg)

`孩`字`background: -0.0px -1808.0px;`

对照可知：x=1，y<1831的path id=47

![](https://i.loli.net/2019/07/20/5d32e1e33836915963.jpg)

![](https://i.loli.net/2019/07/20/5d32e2020d9f448822.jpg)







## REF：

[大众点评评论抓取-加密评论信息完整抓取](https://blog.csdn.net/sinat_32651363/article/details/85123876)

