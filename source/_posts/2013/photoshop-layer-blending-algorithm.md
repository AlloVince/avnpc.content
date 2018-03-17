---
slug: photoshop-layer-blending-algorithm
date: '2013-04-18 20:57:20'
title: Photoshop图层混合(Layer Blending)模式的算法实现
id: 181
tags:
  - php
  - Photoshop
  - 图形
---

Photoshop的图层混合(Layer Blending)是实现各种特效的基础之一，在Photoshop新版中已经提供了接近30种图层混合模式，而运用这些图层混合模式则可以将两个图层叠加并且通过一些算法使叠加后的图层呈现新的效果，比如可以通过“变暗”、“正片叠底”使底层图像变暗，通过“叠加”、“柔光”增强底层图片对比度等。

我之前以为这些特效一定经过了复杂的算法，但稍微了解之后才知道图层混合采用的算法其实都简单到难以置信，几乎全是加减乘除就可以搞定。来复习一下计算机图形的基础知识，一张位图由若干像素点组成，而每个像素点，都有自己的颜色与透明度，因此每一个像素都可以分拆为RGB与Alpha四个通道，一般可以采用0-255的值来表示单一通道的颜色值。比如在CSS中，就可以分别指定4个通道的值来定义一个颜色。

    color:rgba(153, 134, 117, 0.2);
    
而两个图层混合，本质上是将两个图层的同一位置的像素点取出，对其RGB通道的值分别进行某种运算，最终生成一个新的RGB值。


来看一个最简单的例子，如果我们想将上层图片`top.png`与下层图片`bottom.png`采用PhotoShop中“正片叠底（Multiply）”模式混合，使用php+GD实现：


    $top = imagecreatefrompng('top.png');
    $bottom = imagecreatefrompng('bottom.png');

    $width = imagesx($top);
    $height = imagesy($top);
    $layer = imagecreatetruecolor($width, $height);
    for ($x = 0; $x < $width; $x++) {
        for ($y = 0; $y < $height; $y++) {
            $color = imagecolorat($top, $x, $y);
            $tR = ($color >> 16) & 0xFF;
            $tG = ($color >> 8) & 0xFF;
            $tB = $color & 0xFF;
            $color = imagecolorat($bottom, $x, $y);
            $bR = ($color >> 16) & 0xFF;
            $bG = ($color >> 8) & 0xFF;
            $bB = $color & 0xFF;
            imagesetpixel($layer, $x, $y, imagecolorallocate($layer, $tR * $bR / 255, $tG * $bG / 255, $tB * $bB / 255));
        }
    }
    header('Content-Type: image/png');
    imagepng($layer);

程序做的事情其实非常简单，遍历图片的所有像素，取得上下图层的RGB值，分别进行`上*下/255`这样一个简单的运算，将新的颜色填充到原来的位置，就完成了一次“正片叠底”的混合。看看效果：


![原图](http://evathumber.avnpc.com/thumb/watermark/demo.jpg) + ![图层](http://evathumber.avnpc.com/thumb/watermark/blend.png) = ![Multiply](http://evathumber.avnpc.com/thumb/watermark/demo,l_Multiply.jpg)

图层混合(Layer Blending)模式算法实现
------------------------------------

首先因为所有的图层混合的处理流程都是差不多的，唯一不同的是颜色通道的算法，因此我们将上面的图片处理抽象一下，将颜色通达算法做成一个可以替换的部分。

``` php
class Blending
{
    public static function layerMultiply($A, $B)
    {
        return $A * $B / 255;
    }
}

function layerBlending($mode, $top = 'top.png', $bottom = 'bottom.png')
{
    $handler = 'Blending::layer' . $mode;
    $top = imagecreatefrompng($top);
    $bottom = imagecreatefrompng($bottom);

    $width = imagesx($top);
    $height = imagesy($top);
    $layer = imagecreatetruecolor($width, $height);
    for ($x = 0; $x < $width; $x++) {
        for ($y = 0; $y < $height; $y++) {
            $color = imagecolorat($top, $x, $y);
            $tR = ($color >> 16) & 0xFF;
            $tG = ($color >> 8) & 0xFF;
            $tB = $color & 0xFF;
            $color = imagecolorat($bottom, $x, $y);
            $bR = ($color >> 16) & 0xFF;
            $bG = ($color >> 8) & 0xFF;
            $bB = $color & 0xFF;
            imagesetpixel($layer, $x, $y, imagecolorallocate($layer, 
                call_user_func($handler, $tR, $bR),
                call_user_func($handler, $tG, $bG),
                call_user_func($handler, $tB, $bB)
            ));
        }
    }
    header('Content-Type: image/png');
    imagepng($layer);
}
```

我们定义了一个混合模式的算法类Blending，Blending中会有一系列静态方法，方法中入口参数A为上图层，B为下图层，只记载最核心的颜色通道算法，就可以通过替换算法的名称来切换不同的混合模式，比如此时我们应用“正片叠底”只需要：

    layerBlending('Multiply');


下面就实际[用PHP+GD来尝试实现Photoshop中所有的图层混合(Layer Blending)模式算法](http://avnpc.com/pages/photoshop-layer-blending-algorithm)吧。在下面所有示例中，A均代表上图层（混合层），B代表下图层（基层）。



###1. 变暗 Darken

> (B > A) ? A : B

取A与B中当前通道颜色值较小的一个，整体会变暗。


    public static function layerDarken($A, $B)
    {
        return $B > $A ? $A : $B;
    }


- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">B</span> = <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">B</span> = <span class="label" style="background:rgb(10, 10, 10)" title="RGB(10, 10, 10)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(10, 10, 10)" title="RGB(10, 10, 10)">C</span>
- <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(10, 10, 10)" title="RGB(10, 10, 10)">C</span>
- <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(10, 10, 10)" title="RGB(10, 10, 10)">C</span>
- <span class="label" style="background:rgb(127, 127, 127)" title="RGB(127, 127, 127)">A</span> + <span class="label" style="background:rgb(30, 0, 30)" title="RGB(30, 0, 30)">B</span> = <span class="label" style="background:rgb(30, 0, 30)" title="RGB(30, 0, 30)">C</span>
- <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">A</span> + <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">B</span> = <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">C</span>
- <span class="label" style="background:rgb(0, 127, 127)" title="RGB(0, 127, 127)">A</span> + <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>


![原图](http://evathumber.avnpc.com/thumb/watermark/demo.jpg) → ![变暗](http://evathumber.avnpc.com/thumb/watermark/demo,l_Darken.png)



###2. 正片叠底 Multiply

> (A * B) / 255

这种方式混合会得到一个比两个图层都暗的颜色。

    public static function layerMultiply($A, $B)
    {
        return $A * $B / 255;
    }


- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">B</span> = <span class="label" style="background:rgb(300, 110, 15)" title="RGB(300, 110, 15)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">B</span> = <span class="label" style="background:rgb(205, 110, 15)" title="RGB(205, 110, 15)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(205, 15, 110)" title="RGB(205, 15, 110)">C</span>
- <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(15, 205, 110)" title="RGB(15, 205, 110)">C</span>
- <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(205, 205, 110)" title="RGB(205, 205, 110)">C</span>
- <span class="label" style="background:rgb(127, 127, 127)" title="RGB(127, 127, 127)">A</span> + <span class="label" style="background:rgb(30, 0, 30)" title="RGB(30, 0, 30)">B</span> = <span class="label" style="background:rgb(142, 127, 142)" title="RGB(142, 127, 142)">C</span>
- <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">A</span> + <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">B</span> = <span class="label" style="background:rgb(254, 127, 254)" title="RGB(254, 127, 254)">C</span>
- <span class="label" style="background:rgb(0, 127, 127)" title="RGB(0, 127, 127)">A</span> + <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">B</span> = <span class="label" style="background:rgb(0, 127, 127)" title="RGB(0, 127, 127)">C</span>



![原图](http://evathumber.avnpc.com/thumb/watermark/demo.jpg) → ![正片叠底](http://evathumber.avnpc.com/thumb/watermark/demo,l_Multiply.png)


###3. 颜色加深 ColorBurn (NG)

> B == 0 ? B : max(0, (255 - ((255 - A) << 8 ) / B))

    public static function layerColorBurn($A, $B)
    {
        return $B == 0 ? $B : max(0, (255 - ((255 - $A) << 8 ) / $B));
    }


- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(127, 127, 127)" title="RGB(127, 127, 127)">A</span> + <span class="label" style="background:rgb(30, 0, 30)" title="RGB(30, 0, 30)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">A</span> + <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(0, 127, 127)" title="RGB(0, 127, 127)">A</span> + <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>



![原图](http://evathumber.avnpc.com/thumb/watermark/demo.jpg) → ![颜色加深](http://evathumber.avnpc.com/thumb/watermark/demo,l_ColorBurn.png)


###4. 线性加深 LinearBurn | 减去 Subtract

> (A + B < 255) ? 0 : (A + B - 255)

比变暗效果更加强烈，深色几乎被转成黑色，浅色也全部被加深。

    public static function layerSubtract($A, $B)
    {
        return $A + $B < 255 ? 0 : $A + $B - 255;
    }

- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">B</span> = <span class="label" style="background:rgb(89, 0, 30)" title="RGB(89, 0, 30)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">B</span> = <span class="label" style="background:rgb(220, 0, 30)" title="RGB(220, 0, 30)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(220, 30, 0)" title="RGB(220, 30, 0)">C</span>
- <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(30, 220, 0)" title="RGB(30, 220, 0)">C</span>
- <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(220, 220, 0)" title="RGB(220, 220, 0)">C</span>
- <span class="label" style="background:rgb(127, 127, 127)" title="RGB(127, 127, 127)">A</span> + <span class="label" style="background:rgb(30, 0, 30)" title="RGB(30, 0, 30)">B</span> = <span class="label" style="background:rgb(187, 127, 187)" title="RGB(187, 127, 187)">C</span>
- <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">A</span> + <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">B</span> = <span class="label" style="background:rgb(126, 0, 126)" title="RGB(126, 0, 126)">C</span>
- <span class="label" style="background:rgb(0, 127, 127)" title="RGB(0, 127, 127)">A</span> + <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">B</span> = <span class="label" style="background:rgb(0, 127, 127)" title="RGB(0, 127, 127)">C</span>



![原图](http://evathumber.avnpc.com/thumb/watermark/demo.jpg) → ![线性加深](http://evathumber.avnpc.com/thumb/watermark/demo,l_LinearBurn.png)

----

###5. 变亮 Lighten

> (B > A) ? B : A

取A与B中当前通道颜色值较大的一个，整体效果就会偏亮。


    public static function layerLighten($A, $B)
    {
        return $B > $A ? $B : $A;
    }




- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">B</span> = <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">B</span> = <span class="label" style="background:rgb(10, 10, 10)" title="RGB(10, 10, 10)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(10, 10, 10)" title="RGB(10, 10, 10)">C</span>
- <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(10, 10, 10)" title="RGB(10, 10, 10)">C</span>
- <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(10, 10, 10)" title="RGB(10, 10, 10)">C</span>
- <span class="label" style="background:rgb(127, 127, 127)" title="RGB(127, 127, 127)">A</span> + <span class="label" style="background:rgb(30, 0, 30)" title="RGB(30, 0, 30)">B</span> = <span class="label" style="background:rgb(30, 0, 30)" title="RGB(30, 0, 30)">C</span>
- <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">A</span> + <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">B</span> = <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">C</span>
- <span class="label" style="background:rgb(0, 127, 127)" title="RGB(0, 127, 127)">A</span> + <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>


![原图](http://evathumber.avnpc.com/thumb/watermark/demo.jpg) → ![变亮](http://evathumber.avnpc.com/thumb/watermark/demo,l_Lighten.png)


###6. 滤色 Screen

> 255 - (((255 - A) * (255 - B)) >> 8))

与正片叠底正好相反，滤色会由两个颜色得到一个比较亮的颜色。

    public static function layerScreen($A, $B)
    {
        return 255 - ( ((255 - $A) * (255 - $B)) >> 8);
    }
    

- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(127, 127, 127)" title="RGB(127, 127, 127)">A</span> + <span class="label" style="background:rgb(30, 0, 30)" title="RGB(30, 0, 30)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">A</span> + <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(0, 127, 127)" title="RGB(0, 127, 127)">A</span> + <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>


![原图](http://evathumber.avnpc.com/thumb/watermark/demo.jpg) → ![滤色](http://evathumber.avnpc.com/thumb/watermark/demo,l_Screen.png)

###7. 颜色减淡 ColorDodge

> (B == 255) ? B : min(255, ((A << 8 ) / (255 - B)))

    public static function layerColorDodge($A, $B)
    {
        return $B == 255 ? $B : min(255, (($A << 8 ) / (255 - $B)));
    }


- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">B</span> = <span class="label" style="background:rgb(184, 0, 0)" title="RGB(184, 0, 0)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(127, 127, 127)" title="RGB(127, 127, 127)">A</span> + <span class="label" style="background:rgb(30, 0, 30)" title="RGB(30, 0, 30)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">A</span> + <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">B</span> = <span class="label" style="background:rgb(126, 0, 126)" title="RGB(126, 0, 126)">C</span>
- <span class="label" style="background:rgb(0, 127, 127)" title="RGB(0, 127, 127)">A</span> + <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>



![原图](http://evathumber.avnpc.com/thumb/watermark/demo.jpg) → ![颜色减淡](http://evathumber.avnpc.com/thumb/watermark/demo,l_ColorDodge.png)



###8. 线性减淡 LinearDodge | 添加 Add

> min(255, (A + B))

    public static function layerAdd($A, $B)
    {
        return min(255, ($A + $B));
    }



- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">B</span> = <span class="label" style="background:rgb(145, 0, 0)" title="RGB(145, 0, 0)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(127, 127, 127)" title="RGB(127, 127, 127)">A</span> + <span class="label" style="background:rgb(30, 0, 30)" title="RGB(30, 0, 30)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">A</span> + <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">B</span> = <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">C</span>
- <span class="label" style="background:rgb(0, 127, 127)" title="RGB(0, 127, 127)">A</span> + <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>


![原图](http://evathumber.avnpc.com/thumb/watermark/demo.jpg) → ![线性减淡](http://evathumber.avnpc.com/thumb/watermark/demo,l_LinearDodge.png)

----

###9. 叠加 Overlay

> (B < 128) ? (2 * A * B / 255):(255 - 2 * (255 - A) * (255 - B) / 255)

    public static function layerOverlay($A, $B)
    {
        return ($B < 128) ? (2 * $A * $B / 255) : (255 - 2 * (255 - $A) * (255 - $B) / 255);
    }


- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">B</span> = <span class="label" style="background:rgb(215, 174, 5)" title="RGB(215, 174, 5)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">B</span> = <span class="label" style="background:rgb(12, 174, 5)" title="RGB(12, 174, 5)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(12, 5, 174)" title="RGB(12, 5, 174)">C</span>
- <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(5, 12, 174)" title="RGB(5, 12, 174)">C</span>
- <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(12, 12, 174)" title="RGB(12, 12, 174)">C</span>
- <span class="label" style="background:rgb(127, 127, 127)" title="RGB(127, 127, 127)">A</span> + <span class="label" style="background:rgb(30, 0, 30)" title="RGB(30, 0, 30)">B</span> = <span class="label" style="background:rgb(29, 0, 29)" title="RGB(29, 0, 29)">C</span>
- <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">A</span> + <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">B</span> = <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">C</span>
- <span class="label" style="background:rgb(0, 127, 127)" title="RGB(0, 127, 127)">A</span> + <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>


![原图](http://evathumber.avnpc.com/thumb/watermark/demo.jpg) → ![叠加](http://evathumber.avnpc.com/thumb/watermark/demo,l_Overlay.png)



###10. 柔光 SoftLight

> B < 128 ?  (2 * (( A >> 1) + 64)) * (B / 255) :  (255 - ( 2 * (255 - ( (A >> 1) + 64 ) )  *  ( 255 - B ) / 255 ));

    public static function layerSoftLight($A, $B)
    {
        return $B < 128 ? 
             (2 * (( $A >> 1) + 64)) * ($B / 255) : 
             (255 - ( 2 * (255 - ( ($A >> 1) + 64 ) )  *  ( 255 - $B ) / 255 ));
    }
    

- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">B</span> = <span class="label" style="background:rgb(231, 15, 0)" title="RGB(231, 15, 0)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">B</span> = <span class="label" style="background:rgb(149, 15, 0)" title="RGB(149, 15, 0)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(149, 0, 15)" title="RGB(149, 0, 15)">C</span>
- <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(0, 149, 15)" title="RGB(0, 149, 15)">C</span>
- <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(149, 149, 15)" title="RGB(149, 149, 15)">C</span>
- <span class="label" style="background:rgb(127, 127, 127)" title="RGB(127, 127, 127)">A</span> + <span class="label" style="background:rgb(30, 0, 30)" title="RGB(30, 0, 30)">B</span> = <span class="label" style="background:rgb(29, 0, 29)" title="RGB(29, 0, 29)">C</span>
- <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">A</span> + <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">B</span> = <span class="label" style="background:rgb(254, 0, 254)" title="RGB(254, 0, 254)">C</span>
- <span class="label" style="background:rgb(0, 127, 127)" title="RGB(0, 127, 127)">A</span> + <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>


![原图](http://evathumber.avnpc.com/thumb/watermark/demo.jpg) → ![柔光](http://evathumber.avnpc.com/thumb/watermark/demo,l_SoftLight.png)



###11. 强光 HardLight

> Overlay(B,A)
> (A < 128) ? (2 * A * B / 255) : (255 - 2 * (255 - A) * (255 - B) / 255)

    public static function layerHardLight($A, $B)
    {
        return ($A < 128) ? (2 * $A * $B / 255) : (255 - 2 * (255 - $A) * (255 - $B) / 255);
    }


- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">B</span> = <span class="label" style="background:rgb(255, 46, 10)" title="RGB(255, 46, 10)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">B</span> = <span class="label" style="background:rgb(208, 46, 10)" title="RGB(208, 46, 10)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(208, 10, 46)" title="RGB(208, 10, 46)">C</span>
- <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(10, 208, 46)" title="RGB(10, 208, 46)">C</span>
- <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(208, 208, 46)" title="RGB(208, 208, 46)">C</span>
- <span class="label" style="background:rgb(127, 127, 127)" title="RGB(127, 127, 127)">A</span> + <span class="label" style="background:rgb(30, 0, 30)" title="RGB(30, 0, 30)">B</span> = <span class="label" style="background:rgb(144, 127, 144)" title="RGB(144, 127, 144)">C</span>
- <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">A</span> + <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">B</span> = <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">C</span>
- <span class="label" style="background:rgb(0, 127, 127)" title="RGB(0, 127, 127)">A</span> + <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">B</span> = <span class="label" style="background:rgb(0, 127, 127)" title="RGB(0, 127, 127)">C</span>



![原图](http://evathumber.avnpc.com/thumb/watermark/demo.jpg) → ![柔光](http://evathumber.avnpc.com/thumb/watermark/demo,l_HardLight.png)



###12. 亮光 VividLight

> B < 128 ? ColorBurn(A,(2 * B)) : ColorDodge(A,(2 * (B - 128)))

    public static function layerVividLight($A, $B)
    {
        return $B < 128 ? 
            (
                $B == 0 ? 2 * $B : max(0, (255 - ((255 - $A) << 8 ) / (2 * $B)))
            ) :
            (
                (2 * ($B - 128)) == 255 ? (2 * ($B - 128)) : min(255, (($A << 8 ) / (255 - (2 * ($B - 128)) )))
            ) ;
    }



- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">B</span> = <span class="label" style="background:rgb(200, 144, 10)" title="RGB(200, 144, 10)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">B</span> = <span class="label" style="background:rgb(20, 144, 10)" title="RGB(20, 144, 10)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(20, 10, 144)" title="RGB(20, 10, 144)">C</span>
- <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(10, 20, 144)" title="RGB(10, 20, 144)">C</span>
- <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(20, 20, 144)" title="RGB(20, 20, 144)">C</span>
- <span class="label" style="background:rgb(127, 127, 127)" title="RGB(127, 127, 127)">A</span> + <span class="label" style="background:rgb(30, 0, 30)" title="RGB(30, 0, 30)">B</span> = <span class="label" style="background:rgb(60, 0, 60)" title="RGB(60, 0, 60)">C</span>
- <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">A</span> + <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">B</span> = <span class="label" style="background:rgb(254, 254, 254)" title="RGB(254, 254, 254)">C</span>
- <span class="label" style="background:rgb(0, 127, 127)" title="RGB(0, 127, 127)">A</span> + <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>


![原图](http://evathumber.avnpc.com/thumb/watermark/demo.jpg) → ![亮光](http://evathumber.avnpc.com/thumb/watermark/demo,l_VividLight.png)




###13. 线性光 LinearLight

> min(255, max(0, ($B + 2 * $A) - 1))


    public static function layerLinearLight($A, $B)
    {
        return min(255, max(
            0, (($B + 2 * $A) - 255)
        ));
    }


- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">B</span> = <span class="label" style="background:rgb(255, 23, 0)" title="RGB(255, 23, 0)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">B</span> = <span class="label" style="background:rgb(0, 23, 0)" title="RGB(0, 23, 0)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(0, 0, 23)" title="RGB(0, 0, 23)">C</span>
- <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(0, 0, 23)" title="RGB(0, 0, 23)">C</span>
- <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(0, 0, 23)" title="RGB(0, 0, 23)">C</span>
- <span class="label" style="background:rgb(127, 127, 127)" title="RGB(127, 127, 127)">A</span> + <span class="label" style="background:rgb(30, 0, 30)" title="RGB(30, 0, 30)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>
- <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">A</span> + <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">B</span> = <span class="label" style="background:rgb(255, 0, 255)" title="RGB(255, 0, 255)">C</span>
- <span class="label" style="background:rgb(0, 127, 127)" title="RGB(0, 127, 127)">A</span> + <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>




![原图](http://evathumber.avnpc.com/thumb/watermark/demo.jpg) → ![线性光](http://evathumber.avnpc.com/thumb/watermark/demo,l_LinearLight.png)


###14. 点光 PinLight

> max(0, max(2 * B - 255, min(B, 2*A))) 


    public static function layerPinLight($A, $B)
    {
        return max(0, max(2 * $A - 255, min($B, 2 * $A)));
    }


- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">B</span> = <span class="label" style="background:rgb(200, 20, 10)" title="RGB(200, 20, 10)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">B</span> = <span class="label" style="background:rgb(145, 20, 10)" title="RGB(145, 20, 10)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(145, 10, 20)" title="RGB(145, 10, 20)">C</span>
- <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(10, 145, 20)" title="RGB(10, 145, 20)">C</span>
- <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(145, 145, 20)" title="RGB(145, 145, 20)">C</span>
- <span class="label" style="background:rgb(127, 127, 127)" title="RGB(127, 127, 127)">A</span> + <span class="label" style="background:rgb(30, 0, 30)" title="RGB(30, 0, 30)">B</span> = <span class="label" style="background:rgb(30, 0, 30)" title="RGB(30, 0, 30)">C</span>
- <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">A</span> + <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">B</span> = <span class="label" style="background:rgb(254, 0, 254)" title="RGB(254, 0, 254)">C</span>
- <span class="label" style="background:rgb(0, 127, 127)" title="RGB(0, 127, 127)">A</span> + <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>

    
![原图](http://evathumber.avnpc.com/thumb/watermark/demo.jpg) → ![点光](http://evathumber.avnpc.com/thumb/watermark/demo,l_PinLight.png)

###15. 实色混合 HardMix

> (VividLight(A,B) < 128) ? 0 : 255

    public static function layerHardMix($A, $B)
    {
        return ($B < 128 ? 
            (
                $B == 0 ? 2 * $B : max(0, (255 - ((255 - $A) << 8 ) / (2 * $B)))
            ) :
            (
                (2 * ($B - 128)) == 255 ? (2 * ($B - 128)) : min(255, (($A << 8 ) / (255 - (2 * ($B - 128)) )))
            ))
            < 128 ? 0 : 255 ;
    }



- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">B</span> = <span class="label" style="background:rgb(255, 1, 0)" title="RGB(255, 1, 0)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">B</span> = <span class="label" style="background:rgb(163, 1, 0)" title="RGB(163, 1, 0)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(163, 0, 1)" title="RGB(163, 0, 1)">C</span>
- <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(0, 163, 1)" title="RGB(0, 163, 1)">C</span>
- <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(163, 163, 1)" title="RGB(163, 163, 1)">C</span>
- <span class="label" style="background:rgb(127, 127, 127)" title="RGB(127, 127, 127)">A</span> + <span class="label" style="background:rgb(30, 0, 30)" title="RGB(30, 0, 30)">B</span> = <span class="label" style="background:rgb(71, 63, 71)" title="RGB(71, 63, 71)">C</span>
- <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">A</span> + <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">B</span> = <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">C</span>
- <span class="label" style="background:rgb(0, 127, 127)" title="RGB(0, 127, 127)">A</span> + <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">B</span> = <span class="label" style="background:rgb(0, 63, 63)" title="RGB(0, 63, 63)">C</span>



![原图](http://evathumber.avnpc.com/thumb/watermark/demo.jpg) → ![实色混合](http://evathumber.avnpc.com/thumb/watermark/demo,l_HardMix.png)

----

###16. 差值 Difference

> abs(A - B)

取A与B差值的绝对值，会得到一个与AB有色彩反差的颜色。

    public static function layerDifference($A, $B)
    {
        return abs($A - $B);
    } 


- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">B</span> = <span class="label" style="background:rgb(145, 45, 235)" title="RGB(145, 45, 235)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">B</span> = <span class="label" style="background:rgb(45, 45, 235)" title="RGB(45, 45, 235)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(45, 235, 45)" title="RGB(45, 235, 45)">C</span>
- <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(235, 45, 45)" title="RGB(235, 45, 45)">C</span>
- <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(45, 45, 45)" title="RGB(45, 45, 45)">C</span>
- <span class="label" style="background:rgb(127, 127, 127)" title="RGB(127, 127, 127)">A</span> + <span class="label" style="background:rgb(30, 0, 30)" title="RGB(30, 0, 30)">B</span> = <span class="label" style="background:rgb(98, 128, 98)" title="RGB(98, 128, 98)">C</span>
- <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">A</span> + <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">B</span> = <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">C</span>
- <span class="label" style="background:rgb(0, 127, 127)" title="RGB(0, 127, 127)">A</span> + <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">B</span> = <span class="label" style="background:rgb(255, 128, 128)" title="RGB(255, 128, 128)">C</span>


![原图](http://evathumber.avnpc.com/thumb/watermark/demo.jpg) → ![差值](http://evathumber.avnpc.com/thumb/watermark/demo,l_Difference.png)



###17. 排除 Exclusion

> A + B - 2 * A * B / 255

    public static function layerExclusion($A, $B)
    {
        return $A + $B - 2 * $A * $B / 255;
    }



- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">B</span> = <span class="label" style="background:rgb(231, 149, 0)" title="RGB(231, 149, 0)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">B</span> = <span class="label" style="background:rgb(15, 149, 0)" title="RGB(15, 149, 0)">C</span>
- <span class="label" style="background:rgb(200, 10, 10)" title="RGB(200, 10, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(15, 0, 149)" title="RGB(15, 0, 149)">C</span>
- <span class="label" style="background:rgb(10, 200, 10)" title="RGB(10, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(0, 15, 149)" title="RGB(0, 15, 149)">C</span>
- <span class="label" style="background:rgb(200, 200, 10)" title="RGB(200, 200, 10)">A</span> + <span class="label" style="background:rgb(10, 10, 200)" title="RGB(10, 10, 200)">B</span> = <span class="label" style="background:rgb(15, 15, 149)" title="RGB(15, 15, 149)">C</span>
- <span class="label" style="background:rgb(127, 127, 127)" title="RGB(127, 127, 127)">A</span> + <span class="label" style="background:rgb(30, 0, 30)" title="RGB(30, 0, 30)">B</span> = <span class="label" style="background:rgb(29, 0, 29)" title="RGB(29, 0, 29)">C</span>
- <span class="label" style="background:rgb(127, 0, 127)" title="RGB(127, 0, 127)">A</span> + <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">B</span> = <span class="label" style="background:rgb(255, 255, 255)" title="RGB(255, 255, 255)">C</span>
- <span class="label" style="background:rgb(0, 127, 127)" title="RGB(0, 127, 127)">A</span> + <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">B</span> = <span class="label" style="background:rgb(0, 0, 0)" title="RGB(0, 0, 0)">C</span>


![原图](http://evathumber.avnpc.com/thumb/watermark/demo.jpg) → ![滤色](http://evathumber.avnpc.com/thumb/watermark/demo,l_Exclusion.png)




参考
-----

- [http://jswidget.com/blog/category/photoshop/](http://jswidget.com/blog/category/photoshop/)
- [http://illusions.hu/effectwiki/doku.php?id=list_of_blendings](http://illusions.hu/effectwiki/doku.php?id=list_of_blendings)


