# ReusableNestingScrollview

ReusableNestingScrollview是一个解决滚动视图中SubView全局复用和回收问题的组件。区别于传统的类似解决方案，RNS无需继承特殊ScrollView，只通过扩展Delegate的方式实现，从而更广泛的支持WKWebView、UIWebView等滚动视图，更加通用和独立。

> 组件服务于[HybridPageKit](https://github.com/dequan1331/HybridPageKit)，一个资讯类内容底层页完善的通用组件。

###1.Features

1.	无需继承，适用于WebView等全部滚动视图
	1.	区别传统组件的实现方式，无需继承特殊ScrollView，通过代理分发的方式，使子View滚动复用回收的功能能应用到任何滚动视图中
	2. 支持全部滚动视图（UIScrollview、WKWebView、UIWebView等等）
	3. 使用简单，10+行 关键函数代码完成ScrollView代理基本逻辑

			使用场景举例：WebView展示简单内容，复杂业务逻辑使用NativeView，实现Hybrid的内容页面，并且支持Native滚动复用回收。

2.	更加完善的区域和状态，灵活处理业务逻辑.
	1.	自定义增加WorkRange区域。子View增加到三种状态，更加灵活的处理业务逻辑
	2. 回收池 ： 子View清除状态，进入全局回收池
	3. 准备区 ： 子View进入准备区，用于懒加载，数据预拉取等逻辑
	4. 展示区 ： 子View在屏幕中展示，用于渲染展示等逻辑

	
			使用场景举例：一张Gif图片，进入准备区执行数据拉取，不播放，进入展示区进行播放，离开展示区进入准备区不立即回收但停止播放。

3.	数据驱动，View无状态只负责渲染展示
	
	1. view不保存数据信息，不保存frame信息，只负责渲染
	2. model实现protocol，参与全部的逻辑、复用回收计算等

4.	全局跨View复用及容错保护
	
	1.	全局View回收池，支持同类型View 当前页面、跨页面复用回收
	2.	自动回收失去父页面的View，防止内存泄露
	3.	回收池线程安全，自定义回收view阈值等
	

###2.Usage
	
只需三步，轻松实现
	
1.	扩展view相关联数据Model实现protocol

		@interface TestModel : NSObject <RNSModelProtocol>
		
		
		-(NSString *)getUniqueId{
		    return _uniqueId;
		}
		-(void)setComponentFrame:(CGRect)frame{
		    _componentFrame = frame;
		}
		-(CGRect)getComponentFrame{
		    return _componentFrame;
		}
		-(Class)getComponentViewClass{
		    return _componentViewClass;
		}
		-(__kindof RNSComponentContext *)getCustomContext{
		    return nil;
		}

2.	扩展ScrollView delegate

	
		_handler = [[RNSHandler alloc]initWithScrollView:scrollView
	                          externalScrollViewDelegate:self 
	                            scrollWorkRange:200.f 
	                            componentViewStateChangeBlock:^(RNSComponentViewState state, 
	                            NSObject<RNSModelProtocol> *componentItem, __kindof UIView *componentView) {
	        
	        // 复用回收View状态变化处理
	        // 进入准备区 & 进入展示 & 离开展示 & 离开准备区
	    }];

3.	增加、修改数据时reload

		[_handler reloadComponentViewsWithProcessBlock:^(NSMutableDictionary<NSString *,NSObject<RNSModelProtocol> *> *componentItemDic) {        
        	
        	//修改、重新设置componentItemDic中model的origin
    	}];