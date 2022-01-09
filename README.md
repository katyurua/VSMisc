# VSMisc
## [sharpaamcmod.py](https://github.com/katyurua/VSMisc/blob/main/sharpaamcmod.py)
## TAAMBK
TAA说明：
使用方法：
taa(aatype, preaa, sharp, postaa, mtype, src, predown, aarepair, p1, p2, p3, p4, p5, p6)

例：
AVISource("video.avi")
taa(aatype=-2, preaa=-1, sharp=80, postaa=true, mtype=2, src=last, predown=true, aarepair=24, p1=48, p2=0.7, p3=0.15)


参数解释：

aatype [int: -3~5 默认=1]
----------------------
第二重（最主要的）AA模式:
= -3 : nnedi3 & Sangnom
= -2 : eedi3 & Sangnom
= -1 : eedi2 & Sangnom
= 0 : 无处理
= 1 : eedi2
= 2 : eedi3
= 3 : nnedi3
= 4 : nnedi3做upscale后Sangnom
= 5 : Spline36Resize做upscale后Sangnom
= 6 : 同5，速度比5快得多，不过默认参数下除了mtype=5以外强度略有降低


p1~p6 [float]
----------------------
第二重（最主要的）AA模式下的自定义参数。具体涵义与默认值根据不同aatype而不一样，具体看avsi内文档。


preaa [int: -1~2 默认=1]
----------------------
即原"drc"参数，preaa的处理方向，默认为横向（同daa）
= -1 : 两个方向
= 0 : 无preaa处理
= 1 : 横向
= 2 : 纵向


mtype [int: 0~6, 默认=0 (当aatype=0且preaa=0时) / 5 (其他情况下)]
----------------------
Masktools里的边缘处理方法
= 0 : 不处理
= 1 : Sobel算子
= 2 : Roberts算子
= 3 : Prewitt算子
= 4 : TEdgeMask方式
= 5 : tcanny方式（canny算子）
= 6 : MSharpen方式


mthr [int: 0~255, 默认=32]
----------------------
mask的阙值，越高则细小的线条越不会被当作edge mask


sharp [float, 默认=0.3 (preaa=1 or 2时) / -1 (preaa=-1时) / 0 (preaa=0且aatype=0时) / 0.2 (preaa=0且0<aatype<=3时) / 80 (其他情况)]
--------------
post-sharpening的方法和参数
< -1 : 指定lsfmod(defaults="slow")时的"strength"，取正值，较慢，例如sharp=-50时即为lsfmod(strength=50, defaults="slow", source=src)
= -1 : Contra-sharpening，较慢, 细节还原最好，但是源的锯齿很厉害的话这种sharpen会将锯齿重新引入（虽然肯定没原来强）
处于(-1,0)区间: 指定lsfmod(defaults="fast")时的"strength"，取正数并乘以100，例如sharp=-0.5时即为lsfmod(strength=50, defaults="fast", source=src)
= 0 : 无post-sharpening处理
处于(0,1)区间 : 直接用avs内置sharpen滤镜，最快，质量还可以，只取绝对值，取自Didée菊苣的HighQualitySharpen，不推荐用绝对值超过0.6的值
>= 1 : 指定lsfmod(defaults="old")时的"strength"，例如sharp=50时即为lsfmod(strength=50, defaults="old", source=src)，速度和效果同LimitedSharpenFaster


postaa [bool, 默认=true (当0.4<|sharp|<1或|sharp|>60时) / false (当sharp≠0时)]
----------------------
用Didée菊苣的Soothe做第三重aa，主要目的是对付post-sharpen里可能导致的aliasing


src [clip, 默认=input]
----------------------
用于后处理中的源视频，主要是像lsfmod的source参数、HQsharpen的chroma、以及mt_merge之类的用。一般来说不需要调整，如果想降低锐化程度，或者使用其他滤镜处理过的clip作为非edge区域的话可以酌情使用

mclip [clip, 默认=未设定]
----------------------
当设定mclip时，将mclip当作edge mask而不使用内部mask


predown [bool, 默认=false]
----------------------
先进行downscale到原始分辨率的3/4，处理后再拉回去。对付粪upconv非常有效，但是细节损失更大


aarepair [int, 默认=0 (当predown=false时) / 24 (其他情况下)]
----------------------
用repair进行细节修复，非0时等同于repair里的mode参数。
这是目前我自己实验出来对predown=true时先downscale再upscale导致的细节损失修复最快的方法，一般情况下效果也相当不错（默认的24或者略低的23时），过低的参数细节保留程度更高但是aliasing也会回复得更厉害，我个人觉得用23/24就可以了。
原理是将repair反过来用，将源作为filter过的，aa后的clip作为source，然后哟个repair来把线条中心不需要动的欢迎回源的样子，而线条邻近的pixel取aa后的来去除aliasing。
现在这个参数的默认值是在predown=true时为24，否则为0（不使用），实际上不管用不用predown，即使是关闭mtype和sharp就单独靠这个aarepair，也可以在快得多的速度下取得不错的细节修复效果，譬如可以直接这样处理
代码： 全选

taa(mtype=0, sharp=0.2, postaa=false, predown=false, aarepair=24)
edit（16 Feb 2012）：
貌似我加上src参数都快一年了，
始终没见有人想到去用taa的mask与src组合的隐藏用法，
于是来提一下：
taa(src=fft3dgpu.f3kdb)
这样就是对edge做aa，而非edge做降噪+deband……
