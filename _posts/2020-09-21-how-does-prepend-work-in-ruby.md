---
layout: post
title: Ruby如何实现Module#prepend
date: 2020-09-21 11:07 +0800
---
当在类中前置(prepend)模块的时候，Ruby会在超类链中把它置于类的正下方.


如下示例代码：

``` ruby
module Professor
  def name
    "Prof. #{super}"
  end
end

class Mathematician
  attr_accessor :name
  prepend Professor

end

poincare = Mathematician.new
poincare.name = 'Henri Poincare'
p poincare.name # => "Prof Henri Poincare"

```

当包含前置模块时，Ruby会创建目标类(Mathematician)的副本(在内部叫原生类)，并且把它设置为前置模块(Professor)的超类。Ruby使用了`rb_classext_struct`结构体中的`origin_`指针来记录该类(Mathematician)的原生副本。此外，Ruby会从原始类(original class Mathematician)中把所有的方法移动到原生类中，这意味着这些方法可能会被前置模块中的同名方法重载。

![how-does-prepend-work-in-ruby](/images/how-does-prepend-work-in-ruby.jpg)

源码：
``` c
void
rb_prepend_module(VALUE klass, VALUE module)
{
    int changed = 0;

    ensure_includable(klass, module);
    // 创建origin类
    rb_ensure_origin(klass);
    // 包含模块,这里做的事情其实和`Class#include`一样
    changed = include_modules_at(klass, klass, module, FALSE);
    if (changed < 0)
	rb_raise(rb_eArgError, "cyclic prepend detected");
    if (changed) {
	rb_vm_check_redefinition_by_prepend(klass);
    }
    if (RB_TYPE_P(klass, T_MODULE)) {
        rb_subclass_entry_t *iclass = RCLASS_EXT(klass)->subclasses;
        while (iclass) {
            include_modules_at(iclass->klass, iclass->klass, module, FALSE);
            iclass = iclass->next;
        }
    }
}
```

``` c
void
rb_ensure_origin(VALUE klass)
{
    VALUE origin = RCLASS_ORIGIN(klass);
    if (origin == klass) {
        // 为新的origin类分配内存空间
	origin = class_alloc(T_ICLASS, klass);
	OBJ_WB_UNPROTECT(origin); /* TODO: conservative shading. Need more survey. */
        // 设置新origin类的超类为被前置的类(Mathematician类)的超类
	RCLASS_SET_SUPER(origin, RCLASS_SUPER(klass));
        // 设置被前置的类(Mathematician类)的超类为origin类
	RCLASS_SET_SUPER(klass, origin);
        // 设置被前置的类的origin指针指向新创建的origin类
	RCLASS_SET_ORIGIN(klass, origin);
        // 复制被前置的类(Mathematician类)的方法，并删除被前置的类(Mathematician类)的方法
	RCLASS_M_TBL(origin) = RCLASS_M_TBL(klass);
	RCLASS_M_TBL_INIT(klass);
	rb_id_table_foreach(RCLASS_M_TBL(origin), move_refined_method, (void *)klass);
    }
}
```
