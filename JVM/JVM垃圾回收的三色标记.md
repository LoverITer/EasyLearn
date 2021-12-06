#  JVM垃圾回收的三色标记



三色标记法是一种垃圾回收法，它可以让JVM不发生或仅短时间发生STW(Stop The World)，从而达到清除JVM内存垃圾的目的。JVM中的**CMS、G1垃圾回收器**所使用垃圾回收算法即为三色标记法。





三色标记法，顾名思义，JVM将对象的颜色分为了黑、灰、白，三种颜色。

* **白色**：该对象没有被标记过。（对象垃圾）
* **灰色**：该对象已经被标记过了，但该对象下的属性没有全被标记完。（GC需要从此对象中去寻找垃圾）
* **黑色**：该对象已经被标记过了，且该对象下的属性也全部都被标记过了。（程序所需要的对象

### 算法流程

<img src="https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fpic3.zhimg.com%2Fv2-0d80e5be64090127472383b3b7a5ccfe_r.jpg&refer=http%3A%2F%2Fpic3.zhimg.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1639817662&t=ea01944e2b88ee93b932c81b870832f6" style="zoom:67%;" />

