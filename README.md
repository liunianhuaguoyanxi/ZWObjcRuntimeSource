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

## 2.+load加载处理过程阅读顺序标记以及注释理解：
### objc-os.mm中
#### _objc_init
#### load_images
#### call_load_methods
#### call_class_loads
#### call_category_loads
#### (*load_method)(cls, SEL_load)

## 2.+initalize加载处理过程阅读顺序标记以及注释理解：
### objc-runtime-new.mm中
#### Method class_getInstanceMethod(Class cls, SEL sel)
#### lookUpImpOrNil(cls, sel, nil,NO)
#### IMP imp = lookUpImpOrForward(cls, sel, inst, initialize, cache, resolver);
#### _class_initialize (_class_getNonMetaClass(cls, inst));
#### call_category_loads
#### callInitialize(cls);
#### (*load_method)(cls, SEL_load)
