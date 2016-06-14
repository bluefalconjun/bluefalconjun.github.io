##[Adding New Device](http://source.android.com/source/add-device.html) 

--------
**当前页面中对设备(device)和产品(product)的描述,仅对企业编译/产品团队创建全新项目开发有效.**

----------
#### 理解编译层级:
指定的系统架构(**arch**)上可以有多个板型(**board**). 基于指定板型上可以有多个产品(**product**).
在每个编译级别上均可以单独定义并且指定当前级别的赋值.

> **Product Layer:**

实例:

    myProduct, myProduct_eu, MyProduct_eu_fr

描述:
**product**层定义相关的产品功能(software/module),例如那些模块/应用会被编译引用.支持那些语言/环境.多语言环境的配置等. 这是整个**product**的产品名称.**product-specific**变量在**product**定义Makefile中.  **product**可从另一个**product**继承.常见做法是创建一个包含所有**product**共有功能的**base product**. 然后基于这个**product**创建**product**变种. 例如:可以从**base product**创建两个**product**,分别使用不同的**radio**模块(CDMA/GSM).

> **Board/Device Layer:**

实例:

    sardine,trout,goldfish.
描述:

**device/board**层定义构成**device**的物理设备(**device**的工业设计).例如:北美的设备使用QWERTY键盘,法国则使用AZERTY键盘. 该层通常也包含不同的硬件原理图设计的区别. 包含不同板级的外设配置. 它的名字可以采用不同的代码名来区分不同的**board/device**配置.

> **Arch Layer:**

实例:

    arm,x86,mips,arm64,x86_64,mips64.
描述:
**arch**层定义了处理器的配置,即使用哪种系统/指令架构.


----------
####使用编译变量:
当编译指定的**product**时,通常做法是定义一个最终产品的最小变量集合.
在模块(**module**)定义中,通过对**`LOCAL_MODULE_TAGS`**标签赋值来指定. 这个标签可以赋值为单个/多个变量(**optional** /**debug**.**eng**).
如果当前模块没有对**`LOCAL_MODULE_TAGS`**赋值以定义标签,默认为**optional**.标签为**optional**的模块,仅当它被**product**配置宏**`PRODUCT_PACKAGES`**引用时才会被编译/安装到系统中.

当前定义的编译变量(build-variant):

> **eng:**

    缺省的编译选项.
	装入标签的模块: eng / debug.
	装入加入到PRODUCT_PACKAGES的模块.
	ro.secure=0
	ro.debuggable=1
	ro.kernel.android.checkjni=1
	adb默认开启.

> **user:**

    类似为最终release编译方式:
	只有标签为user的module才会被安装.
	装入加入到PRODUCT_PACKAGES的模块.
	ro.secure=1
	ro.debuggable=0
	adb默认关闭.

> **userdebug:**

	同user相同.除了:
	也装入标签为debug的模块.
	ro.debuggable=1.
	adb默认开启.


----------
####建立product

以Nexus6为例描述如何设置一个**product**的Makefile.
实际代码参考: [device/moto/shamu](https://android.googlesource.com/device/moto/shamu/+/android-5.1.1_r13)

**编写Makefile:**

1. 为**product**创建 device/**company_name**/**device_name**目录. 例如: device/moto/shamu. 这个目录包含了为了编译device所需的Makefile和源代码.
2. 创建**device.mk** 这个Makefile定义**device**所需的文件和模块.
3. 创建**product**定义的Makefile(**aosp_shamu.mk**)来指定基于当前**device**的特定的product. 
    参见shamu的Makefile, 这个**product**继承自 device/moto/shamu/device.mk 和 vendor/moto/shamu/device-vendor.mk. 同时指定product相关信息,例如name.brand.model.
4. 创建**AndroidProducts.mk**来引用当前**product**的Makefile. 例子中只有一个**product**的Makefile被引用. 参见 device/moto/shamu/AndroidProducts.mk.
5. 创建**BoardConfig.mk**文件来包含**Board**指定的配置. 参考 [device/moto/shamu/BoardConfig.mk](https://android.googlesource.com/device/moto/shamu/+/android-5.1.1_r13/BoardConfig.mk)
6. 创建**vendorsetup.sh**文件,使用**`add_lunch_combo`** **`product_name-build-variant`** 将**product**加入到系统中.
7. 按照该顺序,从**base device**中创建多个**product**变种.

----------
**aosp_shamu.mk:**

    # Inherit from the common Open Source product configuration
	$(call inherit-product, $(SRC_TARGET_DIR)/product/aosp_base_telephony.mk)
	
	PRODUCT_NAME := aosp_shamu
	PRODUCT_DEVICE := shamu
	PRODUCT_BRAND := Android
	PRODUCT_MODEL := AOSP on Shamu
	PRODUCT_MANUFACTURER := motorola
	PRODUCT_RESTRICT_VENDOR_FILES := true
	
	$(call inherit-product, device/moto/shamu/device.mk)
	$(call inherit-product-if-exists, vendor/moto/shamu/device-vendor.mk)
	
	PRODUCT_NAME := aosp_shamu
	
	PRODUCT_PACKAGES += \
	    Launcher3

----------
**AndroidProduct.mk:**

	#
	# This file should set PRODUCT_MAKEFILES to a list of product makefiles
	# to expose to the build system.  LOCAL_DIR will already be set to
	# the directory containing this file.
	# 当前文件通过引用product makefile的列表,来向编译系统申明其内容. LOCAL_DIR已经设置到包含当前文件的目录上.
	# This file may not rely on the value of any variable other than
	# LOCAL_DIR; do not use any conditionals, and do not look up the
	# value of any variable that isn't set in this file or in a file that
	# it includes.
	# 这个文件不依赖于任何除了LOCAL_DIR之外的变量. 不要在该文件中使用条件/查找变量等操作.
	
	PRODUCT_MAKEFILES := \
	    $(LOCAL_DIR)/aosp_shamu.mk


----------

下表是Product定义变量表. 可以在Product Makefile中指定变量:

    PRODUCT_AAPT_CONFIG:		创建package时的aapt配置.
	PRODUCT_BRAND:				如果存在软件配置的运营商的商标.
	PRODUCT_CHARACTERISTICS:	aapt的指定配置来允许加入变量指定的资源到package中.eg: tablet,nosdcard
	
	PRODUCT_COPY_FILES:			以source_path:destination_path\n关键字组成的列表. 编译product时source path内容会copy到dest path.
	
	PRODUCT_DEVICE:				device/board的工业设计的名字. 同board名字相同. 编译系统通过它来定位BoardConfig.mk. eg: tuna
	
	PRODUCT_LOCALES:			以下划线分隔的双字符语言代码. 双字符代码对描述多中用户语言配置,例如UI语言,时区,货币类型. 可设置多个支持的语言代码.当前标签中的第一个为product默认语言. eg: en_GB(english GreatBritish.) de_DE es_ES fr_CA
	
	PRODUCT_MANUFACTURER:		生产厂家名字,例如:acme
	PRODUCT_MODEL:				最终产品的终端用户可见型号.
	PRODUCT_NAME:				完整产品的终端用户可见名字. 将会出现在Settings > About screen.	
	
	PRODUCT_OTA_PUBLIC_KEYS:	OTA使用的公匙列表.
	PRODUCT_PACKAGES:			装入系统的apk和模块列表.例如:Calendar Contacts
	PRODUCT_PACKAGE_OVERLAYS:	指定使用默认资源或者product指定重载. eg: vendor/acme/overlay
	
	PRODUCT_PROPERTY_OVERRIDES: 以格式为"key=value"的系统属性定义列表.


test update.