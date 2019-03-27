# ZWObjcRuntimeSource
iOS Objc runtime Soure （obj runtime源码）


# 官方地址：https://opensource.apple.com/tarballs/objc4/

## 1.category加载处理过程阅读顺序标记以及注释理解：
### objc-os.mm中
#### _objc_init
#### map_images
#### map_images_nolock

### objc-runtime-new.mm
#### _read.images
#### remethodizeClass
#### attachCategories
#### attachLists
#### realloc、memmvoe、memcpy
```
-------------------------Category的本质----------------------------------
runtime在运行时，动态将分类的方法合并到类对象方法和元类对象方法中

分类底层也是结构体，每个分类的结构是一样的，只是存储的数据不一样
{
   const char *name    
   struct _class_t *cls
   const struct _method_list_t *instance_methods 对象方法列表;
   const struct _method_list_t *class_methods    类方法列表;
   const struct _protocol_list_t *protocols      协议列表;
   const struct _prop_list_t *properties         属性列表;

}
-------------------------Category的加载处理过程----------------------------
1.通过runtime加载某个类的所有category的数据

2.把所有Category的对象方法，类方法，属性，协议数据，合并到一个大数组中（最后面参与编译的Category数据会放在数组最前面）

3.将合并后的分类数据（对象方法，类方法，属性，协议数据），插入到该类原来数据的前面（所以有分类的，会先调用分类）
--------------------------------------------------------------------------
```
## 2.+load加载处理过程阅读顺序标记以及注释理解：
### objc-os.mm中
#### _objc_init
#### load_images
#### call_load_methods
#### call_class_loads
#### call_category_loads
#### (*load_method)(cls, SEL_load)
```
-------------------------+load的调用过程------------------------------------
+load方法会在runtime加载类，分类时调用

每个类，每个分类的+load，在程序运行过程中只调用一次，即使main函数里不执行任何任务，也会调用
（因为+load是直接通过函数指针，指向那个函数，拿到函数的地址，找到函数的代码，直接调用，而且不是通过objc_msg而是直接调用）

调用顺序
1.先调用类的+load方法
按照编译顺序进行调用（先编译，先调用）
调用子类的+load之前会先调用父类的+load

2.再调用分类的+load
按照编译顺序进行调用（先编译，先调用）

+load方法一般系统自动调用，程序员不需要去调用
--------------------------------------------------------------------------
```
## 3.+initalize加载处理过程阅读顺序标记以及注释理解：
### objc-runtime-new.mm中
#### Method class_getInstanceMethod(Class cls, SEL sel)
#### lookUpImpOrNil(cls, sel, nil,NO)
#### IMP imp = lookUpImpOrForward(cls, sel, inst, initialize, cache, resolver);
#### _class_initialize (_class_getNonMetaClass(cls, inst));
#### call_category_loads
#### callInitialize(cls);
#### (*load_method)(cls, SEL_load)
```
-------------------------+initialize的调用过程----------------------------
+initialize会在类第一次接收消息的时候调用
如[Person alloc]  Person这个类第一次接收消息时,调用alloc
相当于objc_msgSend([person class],@selector(alloc));
+initialize是通过消息机制objc_msgSend，通过isa指针，找到对应的类对象或者元类对象，去里面的列表找+initialize方法进行

调用顺序：
objc_msgSend导致的注意细节！：
1.会自动先调用父类的+initialize，再调用子类的+initialize（前提是父类中要写+initialize)
每个类只会初始化一次

2.如果分类有+initialize 会覆盖类本身的+initialize
3.如果子类没有+initialize，会调用父类的+initialize（所以父类的initialize会被调用多次，因为class_megSend）
--------------------------------------------------------------------------
```
