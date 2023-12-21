Flutter中如何使用Material Design 3的Color system

概念
* Color role： 颜色主题中定义的颜色名称，M3中有26+的color roles
* Dynamic color： 从手机壁纸中获取的主题色（只有Android 12才支持）或者从图片内容中获取的主题色，注意主题色是指一个单一的颜色
* Static color： 用户自己定义的颜色常量
* Baseline color： 就是系统默认的颜色主题

动态主题  
1. 从壁纸或者图片内容中获取一个单一的颜色
2. 通过算法根据生成的颜色再次获取5个颜色：Primary,Secondary,Tertiary,Neutral,Neutral variant
3. 生成上面5个颜色对应的调色板（tonal palette）
4. 然后选择调色板中不同色调的颜色生成完整的Light ColorScheme和Dark ColorScheme,这里面的原理是通过调色板中不同对比度来选择颜色配对color role, 可以有Standard, Medium, High三个档位的对比度

官方链接
https://m3.material.io/styles/color/system/overview
![light](https://github.com/yeshu-cn/yeshu.github.io/assets/3785459/86f4ec22-7601-4750-8b44-0d067cb13730)

demo:https://github.com/yeshu-cn/flutterdemo



