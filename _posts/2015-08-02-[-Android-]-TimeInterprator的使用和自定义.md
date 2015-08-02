---
layout: post
category : Android
tagline: "TimeInterpolator"
tags : [Android]
shortContent: "在使用Animation的时候，我们经常可以看到系统内提供的TimeInterpolator，TimeInterpolator的中文意思是差值器，可以看做是动画的运行过程中的点缀。"
---
{% include JB/setup %}

在使用Animation的时候，我们经常可以看到TimeInterpolator，TimeInterpolator的中文意思是差值器，可以看做是动画的运行过程中的点缀。对了，以前版本有一个接口Interpolator，这两个是一样的，Interpolator直接继承了TimeInterpolator。

##### 如何使用TimeInterpolator

TimeInterpolator一般都与动画进行使用，当然也有其他应用场景。使用方式如下，定义一个Animation，并设置它的setInterpolator(TimeInterpolator)即可。

	Animation anim = new SomeAnimation(some parameters;
	anim.setDuration(duration);
	anim.setInterpolator(interpolator);
	someView.startAnimation(anim);

##### TimeInterpolator的工作原理

Animation在设置了TimeInterpolator，开始startAnimation之后，Animation始终会根据当前运行时间和设置的运行时间相除，来返回input结果。如果我们并不设置TimeInterpolator，那么动画的走势即是线性的。通过TimeInterpolator可以改变这个input的值，并将我们所设置的函数走势返回，那么动画的运动轨迹就变得可控和有趣了。

##### 如何制作自己的TimeInterpolator

昨天偶然看了drakeet的一篇博客，有关于呼吸灯的，看到里面有一个他从Google上找的呼吸函数，我想将它制作成一个TimeInterpolator应该不错哟。

下面展示的是该呼吸函数

![呼吸函数](https://raw.githubusercontent.com/tracytheron/theron_blog/gh-pages/assets/images/s1.png)

![呼吸函数](https://raw.githubusercontent.com/tracytheron/theron_blog/gh-pages/assets/images/s2.png)


怎么样？看上去是不是很复杂？很厉害的样子。我跟着这个函数做了几下呼吸，感觉这个呼吸函数还是挺靠谱的。那我们就动手吧。

我们知道实现TimeInterpolator接口，必须实现这个方法 float getInterpolation(float input);input的在动画的时间内，返回平均地0-1的float值，我们所要做的就是将这个input值和我们的函数对应起来。

呼吸函数分为两部分，吸气和呼气，即y坐标0~1的递增和y坐标1~0的递减。这个函数和正弦函数sin很像，正如drakeet所说，如果用正弦函数来表述呼吸，那这个呼吸就有些急促，大家可以呼吸试一下。因此，吸气时间相对较短，大约占整个一个呼吸周期的1/3,而呼气则占2/3。所以，我们的input值中，0~1/3交给吸气函数，1/3~1交给呼气函数。

首先，我们解决吸气函数，也就是图中第一个函数。

它的x的取值范围是0~2，y的取值范围是0~1。图中的函数很复杂，是因为它包含N个周期，如果将它精简，函数为y=1/2sinA+1/2; 我将sin内部统一成A，那么我们的问题就简化了，也就是当x取值为0~1/3之间时，这个函数y的值为0~1。

由于y的值是从0到1，那么可以知道，A的值为-π/2~π/2。

也就可以列出下列方程组：

	0*a+b=-π/2
	1/3*a+b=π/2
	
可以解出a=3π,b=-π/2，即y=1/2sin(3πx-π/2)

所以，当input的值为0~1/3之间时，我们getInterpolation方法的返回函数应该是

	0.5f+0.5f*Math.sin(input*3.0f*Math.PI-Math.PI*0.5f)
	
接着，我们再来解决当input为1/3~1之间的呼气函数。

它的x的取值范围是2~6，y的取值范围是1~0。图中的函数很复杂，是因为它包含N个周期，如果将它精简，函数为y=(1/2sinA+1/2)^2；我将sin内部统一成A，那么我们的问题就再次简化了，也就是当x取值为1/3~1之间时，这个函数y的值为1~0。

由于y的值是从1到0，那么可以知道，A的值为π/2~3π/2。

也就可以列出下列方程组：

	1/3*a+b=π/2
	1*a+b=3π/2
	
可以解出a=3π/2,b=0，即y=(1/2sin(3/2*πx)+1/2)^2

所以，当input的值为1/3~1之间时，我们getInterpolation方法的返回函数应该是

	Math.pow((0.5 * Math.sin(3f * Math.PI * input * 0.5f) + 0.5f), 2)
	

综上，整个BreathingInterpolator如下：

	public class EaseBreathInterpolator implements TimeInterpolator {
        public EaseBreathInterpolator() {}
        public EaseBreathInterpolator(Context context, AttributeSet attrs) {}
        public float getInterpolation(float input) {
            if(input<0.333){
                return (float) (0.5f+0.5f*Math.sin(input*3.0f*Math.PI-Math.PI*0.5f));
            }else{
                return (float) Math.pow((0.5 * Math.sin(3f * Math.PI * input * 0.5f) + 0.5f), 2);
            }      
        }
    }


##### 小结

TimeInterpolator使我们的动画更加自然和动感，不过还是需要一些计算能力，当然自己写一个TimeInterpolator也是十分有趣的。

###### 好热的天！


