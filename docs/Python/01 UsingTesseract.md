# Tesseract

[下载地址](https://github.com/UB-Mannheim/tesseract/wiki)



1. 安装

2. 设置环境变量

	+ 系统变量Path`Tesseract-OCR路径`
	+ 新建`TESSDATA_PREFIX`系统变量，变量值为`...\Tesseract-OCR\tessdata`
	+ 测试

	![](https://i.loli.net/2019/07/31/5d406b1f299be97008.jpg)

3. python安装pytesseract`pip install pytesseract`

4. 修改Python37\site-packages内的`pytesseract\pytesseract.py`内的文件，指定安装路径`tesseract_cmd = '.../Tesseract-OCR/tesseract.exe')`

```python
import pytesseract
from PIL import Image

// pytesseract.pytesseract.tesseract_cmd = 'C://Program Files (x86)/Tesseract-OCR/tesseract.exe'
text = pytesseract.image_to_string(Image.open('./demo.jpg'))

print(text)
```





## REF:

[安装](https://blog.csdn.net/qq_38900441/article/details/82823312)

[测试](https://blog.csdn.net/u013421629/article/details/76854778)