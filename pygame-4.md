title: Pygame第四章 创建视觉
date: 2015-11-07 21:28:44
tags: [Python, Pygame]
---

电脑游戏倾向于视觉上很自然，游戏开发者花费很多精力在处理图像和提升视觉效果来创建最令人愉快的体验上。

# 像素的威力

显示器里的一个独立的点称为一个图像单元或像素。如果一个显示器分辨率为1280x1024，则像素总数为1310720。一个屏幕上像素越多，则图片的质量越好。典型的家用电脑可以显示的颜色数量最高为16.7百万个。如果你想显示每一种颜色，你需要一个分辨率4096x4096的显示器。

    import pygame
    pygame.init()

    screen = pygame.display.set_mode((640, 480))

    all_colors = pygame.Surface((4096, 4096), depth=24)

    for r in xrange(256):
        print r + 1, 'out of 256'
        x = (r & 15) * 256
        y = (r >> 4) * 256
        for g in xrange(256):
            for b in xrange(256):
                all_colors.set_at((x + g, y + b), (r, g, b))

    pygame.image.save(all_colors, 'allcolors.bmp')

上面程序用到的类或方法：

    class Surface(__builtin__.object)
     |  pygame.Surface((width, height), flags=0, depth=0, masks=None): return Surface
     |  pygame.Surface((width, height), flags=0, Surface): return Surface
     |  pygame object for representing images

    set_at(...)
        Surface.set_at((x, y), Color): return None
        set the color value for a single pixel

    save(...)
        pygame.image.save(Surface, filename): return None
        save an image to disk

# 和颜色打交道

### Pygame中表示颜色

当Pygame需要一种颜色，你需要传进一个包含三个数字的元组，分别对应红绿蓝三种颜色，每个数值范围为[0, 255]。

|颜色     |红色    |绿色    |蓝色    |元组    |
|--------|:-------|:------|:------|:------|
|黑(Black)|0|0|0|(0, 0, 0)|
|蓝(Blue)|0|0|255|(0, 0, 255)|
|绿(Green)|0|255|0|(0, 255, 0)|
|蓝绿(Cyan)|0|255|255|(0, 255, 255)|
|红(Red)|255|0|0|(255, 0, 0)|
|洋红(Magenta)|255|0|255|(255, 0, 255)|
|黄(Yellow)|255|255|0|(255, 255, 0)|
|白(White)|255|255|255|(255, 255, 255)|

(96, 130, 51)僵尸绿，(221, 99, 20)火球黄

    import pygame
    from pygame.locals import *
    from sys import exit

    pygame.init()

    screen = pygame.display.set_mode((640, 480), 0, 32)

    # Creates images with smooth gradients
    def create_scales(height):
        red_scale_surface = pygame.surface.Surface((640, height))
        green_scale_surface = pygame.surface.Surface((640, height))
        blue_scale_surface = pygame.surface.Surface((640, height))
        for x in range(640):
            c = int((x / 639.) * 255.)
            red = (c, 0, 0)
            green = (0, c, 0)
            blue = (0, 0, c)
            line_rect = Rect(x, 0, 1, height)
            pygame.draw.rect(red_scale_surface, red, line_rect)
            pygame.draw.rect(green_scale_surface, green, line_rect)
            pygame.draw.rect(blue_scale_surface, blue, line_rect)
        return red_scale_surface, green_scale_surface, blue_scale_surface

    red_scale, green_scale, blue_scale = create_scales(80)

    color = [127, 127, 127]

    while True:
        for event in pygame.event.get():
            if event.type == QUIT:
                exit()

        screen.fill((0, 0, 0))

        # Draw the scales to the screen
        screen.blit(red_scale, (0, 0))
        screen.blit(green_scale, (0, 80))
        screen.blit(blue_scale, (0, 160))

        x, y = pygame.mouse.get_pos()

        # If the mouse was pressed on one of the sliders, adjust the color component
        if pygame.mouse.get_pressed()[0]:
            for component in range(3):
                if y > component * 80 and y < (component + 1) * 80:
                    color[component] = int((x / 639.) * 255.)
            pygame.display.set_caption('Pygame Color Test - ' + str(tuple(color)))

        # Draw a circle for each slider to represent the current setting
        for component in range(3):
            pos = (int((color[component] / 255.) * 639), component * 80 + 40)
            pygame.draw.circle(screen, (255, 255, 255), pos, 20)

        pygame.draw.rect(screen, tuple(color), (0, 240, 640, 240))

        pygame.display.update()

上面程序用到的类或方法：

    class Rect(__builtin__.object)
     |  pygame.Rect(left, top, width, height): return Rect
     |  pygame.Rect((left, top), (width, height)): return Rect
     |  pygame.Rect(object): return Rect
     |  pygame object for storing rectangular coordinates

    rect(...)
        pygame.draw.rect(Surface, color, Rect, width=0): return Rect
        draw a rectangle shape

    get_pressed(...)
        pygame.moouse.get_pressed(): return (button1, button2, button3)
        get the state of the mouse buttons

    circle(...)
        pygame.draw.circle(Surface, color, pos, radius, width=0): return Rect
        draw a circle around a point

### 调整颜色

要使颜色变暗，只需RGB每一个值都乘于一个0和1之间的系数。

    def scale_color(color, scale):
        red, green, blue = color
        red = int(red * scale)
        green = int(green * scale)
        blue = int(blue * scale)
        return red, green, blue

    fireball_orange = (221, 99, 20)
    print fireball_orange
    print scale_color(fireball_orange, .5)

如果乘于一个大于1的数，颜色会变得更亮。但是如果颜色值大于255时，Pygame会抛出TypeError异常。

    def saturate_color(color):
        red, green, blue = color
        red = min(red, 255)
        green = min(green, 255)
        blue = min(blue, 255)
        return red, green, blue

当颜色饱和至255时，可能就失去原来的色调了，所以需要注意调整的系数。

### 混合颜色

有时候我们想要将一种颜色逐渐混入另一种颜色，比如一个僵尸在路过一个火山熔岩坑的时候，它会由绿色变成橙红色，再变为正常的绿色。我们怎样才能计算中间的颜色使得颜色转换平滑自然呢？
我们用一种叫做**线性插值(linear interpolation)**的方法来做这件事情。公式如下：

    def lerp(value1, value2, factor):
        return value1 + (value2 - value1) * factor

如果逐渐改变系数因子，就会产生颜色的平滑转换。

    import pygame
    from pygame.locals import *
    from sys import exit

    pygame.init()

    screen = pygame.display.set_mode((640, 480), 0, 32)

    color1 = (221, 99, 20)
    color2 = (96, 130, 51)

    factor = 0

    def lerp(value1, value2, factor):
        return value1 + (value2 - value1) * factor

    def blend_color(color1, color2, blend_factor):
        red1, green1, blue1 = color1
        red2, green2, blue2 = color2
        red = lerp(red1, red2, blend_factor)
        green = lerp(green1, green2, blend_factor)
        blue = lerp(blue1, blue2, blend_factor)
        return int(red), int(green), int(blue)

    while True:
        for event in pygame.event.get():
            if event.type == QUIT:
                exit()

        screen.fill((255, 255, 255))

        tri = [(0, 120), (639, 100), (639, 140)]
        pygame.draw.polygon(screen, (0, 255, 0), tri)
        pygame.draw.circle(screen, (0, 0, 0), (int(factor * 639.), 120), 10)

        x, y = pygame.mouse.get_pos()
        if pygame.mouse.get_pressed()[0]:
            factor = x / 639.
            pygame.display.set_caption('Pygame Color Blend Test - %.3f' % factor)

        color = blend_color(color1, color2, factor)
        pygame.draw.rect(screen, color, (0, 240, 640, 240))

        pygame.display.update()

用到的方法：

    polygon(...)
        pygame.draw.polygon(Surface, color, pointlist, width=0): return Rect
        draw a shape with any number of sides

# 使用图片

图片对大多数游戏都是一个必须的部分。电脑存储图片为颜色的网阵。除了常见的红绿蓝，有些图片还存储额外的信息(alpha通道)。颜色的alpha值通常用来表示当图片画在另一张图片上的透明度。

### 存储图片

有很多方式存储图片到硬盘。多年来，很多种图片格式被开发出来。最有用的两种格式是JPEG和PNG。

* JPEG(Joint Photographic Expert Group)-JPEG图片文件扩展名为.jpg或.jpeg。数码相机常用这种格式，因为它是特别设计来存储图片的。它使用有损压缩，会降低图片质量，但是能很好地减小文件大小。JPEG不支持透明，对拥有硬边缘的东西比如字体和图表支持不好。
* PNG(Portable Network Graphics)-PNG文件可能是最全面的图片格式，因为它能保存很多种图片类型并且压缩也很好。它也支持alpha通道，对游戏开发者来说是福利。PNG使用的压缩是无损压缩，缺点就是文件比JPEG文件大。

除了JPEG和PNG，Pygame支持读取下面的格式：

* GIF(非动态)-网上使用的很多，支持透明和动画，只是只能有256种颜色，软件和游戏中使用很少
* BMP-Windows上的标准图像格式，无压缩，质量很高但尺寸很大，一般不使用
* PCX
* TGA(只限非压缩的)
* TIF
* LBM(和PBM)
* PBM(和PGM，PPM)
* XPM

根据经验，对于拥有很多颜色变化的大图片文件使用JPEG，否则使用PNG。

### 使用Surface对象

加载图片到Pygame只需一行，**pygame.image.load**接收一个图片文件名返回一个Surface对象，它是图片的容器。Surface对象可以表示许多种图片类型，但是Pygame隐藏了大多数细节，因此我们能够以相同的方式对待它们。你可以在Surface对象上画图，变形，或拷贝到另一个Surface。甚至屏幕也是一个Surface对象。

### 创建Surface对象

调用**pygame.image.load**是一种创建Surface的方式，它会创建和图片文件相同尺寸和颜色的Surface。也可以创建任意尺寸的空白Surface。使用**pygame.Surface**构造函数可以创建一个空白Surface。

    blank_surface = pygame.Surface((256, 256))

还有一些选项会影响图片的创建。第一个参数是flags：

* HWSURFACE-创建硬件外观，比非硬件外观更快。通常最好不要设置这个标志，而是留给Pygame决定是否使用硬件外观。
* SRCALPHA-创建拥有alpha信息的外观。这个选项需要depth参数设为32。

第二个参数为depth，这个参数和**pygame.display.set_mode**的depth参数一样。通常最好不要设置这个参数(或设置为0)，但是如果想要保留alpha信息，则必须设置为32。

    blank_alpha_surface = pygame.Surface((256, 256), flags=SRCALPHA, depth=32)

### 转换Surface

当你使用Surface对象时，你不需要担心图像信息在内存是怎么存储的，因为Pygame隐藏了这些细节，因此不管什么图片类型，你的代码都能工作。自动转换的唯一缺点是如果你使用不同格式的图片，Pygame需要做更多工作，这可能会降低游戏性能。解决的方法是使用**convert**方法转换所有的图片为相同的格式。
如果不带参数调用**convert**，Surface会被转换为和显示设备的Surface一样的格式。这很有用因为通常起点和终点是相同类型的话拷贝Surface非常快，而大多数图片最终会被拷贝到显示设备。调用**pygame.image.load**时附加**.convert()**是个好习惯。Pygame还提供**convert_alpha**方法，转换Surface并保留alpha信息。
**convert**和**convert_alpha**可以接收另外一个Surface参数，如果提供了这个参数，则Surface会被转换为和参数一样的Surface。

### 矩形对象

当一个函数调用会影响到屏幕的某一部分时，Pygame通常需要你提供一个矩形区域。你可以使用一个4元组定义矩形区域，(left, top, width, height)。或者使用2个2元组，((left, top), (width, height))。

    my_rect1 = (100, 100, 200, 150)
    my_rect2 = ((100, 100), (200, 150))

Pygame有一个Rect类存储和处理矩形对象。因为Rect对象很常使用，所以它包含在**pygame.locals**里面。创建Rect对象使用一样的元组参数。

    my_rect3 = Rect(100, 100, 200, 150)
    my_rect4 = Rect((100, 100), (200, 150))

一旦创建了Rect对象，就可以使用它来调整位置和大小，检查一个点是否在其中，判断矩形是否交叉等。详细请参考[www.pygame.org/docs/ref/rect.html](http://www.pygame.org/docs/ref/rect.html)

### 剪裁

通常创建游戏画面的时候，你想要只绘制一部分。为了解决这个问题，Surface有一个剪裁区域，定义了屏幕的哪一部分可以被绘制。使用**set_clip**方法设置剪裁区域，**get_clip**获得当前剪裁区域。

    screen.set_clip(0, 0, 640, 300)
    draw_map()
    screen.set_clip(0, 300, 640, 180)
    draw_panel()

### 子表面(Subsurface)

一个子表面在其它Surface里面的Surface。当你在子表面上画图时，同时也画在父表面上。子表面的一个用法是画图形字体。**pygame.font**模块可以产生漂亮的一种颜色的文字，但是游戏可能需要更丰富的表现。你可以为每一个字母保存一个文本，但是更简单的方法是保存所有字母到一张图片，然后创建26个子表面。
使用Surface对象的**subsurface**方法创建子表面，它接收一个Rect参数，表示覆盖父表面哪一部分，返回一个新的和父表面一样格式的Surface对象。

    my_font_image = Pygame.load("font.png")
    letters = {}
    letters["a"] = my_font_image.subsurface((0,0), (80,80))
    letters["b"] = my_font_image.subsurface((80,0), (80,80))

    subsurface(...)
        Surface.subsurface(Rect): return Surface
        create a new surface that references its parent

当你使用子表面时，需要记住它们拥有自己的坐标系统。

### 填充表面

**fill**方法使用指定颜色填充Surface对象。

    screen.fill((0, 0, 0))

**fill**方法也可以带一个可选的Rect参数，指定填充区域。

> **注意** 如果你使用其它方法画整个屏幕，就不必调用**fill**来清屏了。

### 设置表面像素

你能对表面做的一个最基本的事情就是设置单一像素，效果就是画一个非常小的点。很少需要一次画一个像素，因为有更多有效的画图方式，但是你需要处理脱机图片将会很有用。
使用**set_at**方法，参数为坐标和颜色，可以在表面画一个像素。

        import pygame
        from pygame.locals import *
        from sys import exit
        from random import randint

        pygame.init()
        screen = pygame.display.set_mode((640, 480), 0, 32)

        while True:
            for event in pygame.event.get():
                if event.type == QUIT:
                    exit()

            rand_col = (randint(0, 255), randint(0, 255), randint(0, 255))
            for _ in xrange(100):
                rand_pos = (randint(0, 639), randint(0, 479))
                screen.set_at(rand_pos, rand_col)

            pygame.display.update()

### 获得表面的像素

与**set_at**互补的是**get_at**，它返回给定坐标像素的颜色。获取像素有时候对于碰撞检测很必要。

    my_color = screen.get_at((100, 100))

> **注意** **get_at**在读取硬件表面时很慢。显示可能是硬件表面，尤其是全屏显示的时候，所以这时应该避免使用**get_at**。

### 锁定Surface

当Pygame在surface上画图时，surface必须先锁定。当surface被锁定时，Pygame完全控制surface，计算机其它进程只能等到surface解锁时才能使用。当你在surface上画图时，加锁和解锁自动发生。但是如果Pygame需要做太多加锁和解锁，程序会变得没有效率。我们可以通过手动加锁和解锁来减少加锁和解锁的次数，从而提高运行速度。

> **注意** **lock**和**unlock**的次数必须一样，如果忘记了**unlock**一个surface，Pygame失去响应。

    import pygame
    from pygame.locals import *
    from sys import exit
    from random import randint

    pygame.init()
    screen = pygame.display.set_mode((640, 480), 0, 32)

    while True:
        for event in pygame.event.get():
            if event.type == QUIT:
                exit()

        rand_col = (randint(0, 255), randint(0, 255), randint(0, 255))
        screen.lock()
        for _ in xrange(100):
            rand_pos = (randint(0, 639), randint(0, 479))
            screen.set_at(rand_pos, rand_col)
        screen.unlock()
        pygame.display.update()

不是所有的surface需要加锁。硬件surface需要(屏幕通常是硬件surface)，普通的surface不需要。Pygame提供一个**mustlock**方法返回True如果surface需要加锁。加锁不需要加锁的surface没有问题。

### Blitting

最常用的surface对象的方法可能是**blit**，是位块传输(**bit block transfer**)的缩写。Blitting意思是从一个surface拷贝图片数据到另一个surface。你可以使用它来画背景，字体，角色和游戏中几乎所有的事。
为了blit一个surface，用目标surface对象调用**blit**，传递源surface和目的坐标。也可以只blit一部分surface。

    screen.blit(background, (0,0))
    screen.blit(ogre, (300, 200), (100*frame_no, 0, 100, 100))

# 用Pygame画图

