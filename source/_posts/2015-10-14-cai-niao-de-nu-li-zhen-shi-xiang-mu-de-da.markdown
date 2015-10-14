---
layout: post
title: "菜鸟的努力——真实项目的搭"
date: 2015-10-14 23:33:58 +0800
comments: true
categories: 
---
====
##菜鸟的努力——真实项目的搭建
>作为我的第一个真实的商业项目，让我来解析一下项目的搭建。
>这个项目的class文件下分为`Other` `HTTP` `一些主要的模块`
>让我来重点解析一下前两个模块

<p>Other文件夹相当于是项目的后勤部分，一些需要的辅助东西都可以在这里找到</p>

<p>Base</p>
<p>Base--Controller</p>
<p>Base--Controller--BaseViewColler</p>
<pre><code>
在这个类里面主要是对UIViewController的修改

- (void)viewDidLoad
{
    [super viewDidLoad];
    //设定好的背景颜色
    self.view.backgroundColor = kColorBackground;
}

- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
	//当点击背景的时候自动丧失焦点
    [self.view endEditing:YES];
}
</pre></code>

<Base--Controller--BaseNavigationController>
<pre><code>
在这个类里面 主要是对导航栏的修改  和  返回按钮的  自定义

//初始化时候调用
+ (void)initialize{
    [self setupNavTheme];
    [self setupBarButton];
}

+ (void)setupBarButton{
    UIBarButtonItem *barButton = [UIBarButtonItem appearance];
    
    NSMutableDictionary *disabledAttrs = [NSMutableDictionary dictionary];
    disabledAttrs[NSForegroundColorAttributeName] = [UIColor lightGrayColor];
    [barButton setTitleTextAttributes:disabledAttrs forState:UIControlStateDisabled];
}

+ (void)setupNavTheme{
    UINavigationBar *navBar = [UINavigationBar appearance];
    
    navBar.tintColor = [UIColor whiteColor];
    //背景颜色
    navBar.barTintColor = kColorNavBar;
    // 设置标题文字颜色
    NSMutableDictionary *attrs = [NSMutableDictionary dictionary];
    attrs[NSForegroundColorAttributeName] = [UIColor whiteColor];
    attrs[NSFontAttributeName] = [UIFont systemFontOfSize:18];
    [navBar setTitleTextAttributes:attrs];
    
    // 2.设置BarButtonItem的主题
    UIBarButtonItem *item = [UIBarButtonItem appearance];
    // 设置文字颜色
    NSMutableDictionary *itemAttrs = [NSMutableDictionary dictionary];
    itemAttrs[NSForegroundColorAttributeName] = [UIColor whiteColor];
    itemAttrs[NSFontAttributeName] = [UIFont systemFontOfSize:14];
    [item setTitleTextAttributes:itemAttrs forState:UIControlStateNormal];
}


- (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated{
    if (self.viewControllers.count > 0) {
        viewController.hidesBottomBarWhenPushed = YES;
        
       UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
        [button setBackgroundImage:[UIImage imageNamed:@"nav_back"] forState:UIControlStateNormal];
        [button setBackgroundImage:[UIImage imageNamed:@"nav_back"] forState:UIControlStateHighlighted];
        button.frame = (CGRect){CGPointZero, button.currentBackgroundImage.size};
        [button addTarget:self action:@selector(back) forControlEvents:UIControlEventTouchUpInside];
        ;
        
        
       viewController.navigationItem.leftBarButtonItem = [[UIBarButtonItem alloc] initWithCustomView:button];
        
    }
    [super pushViewController:viewController animated:YES];
}

- (void)back{
    [self popViewControllerAnimated:YES];
}
</pre></code>

<p>Base--Controller--BaseTabBarController</p>

<pre><code>
在这个类里 主要是 写了 这个项目中几个标签的初始化

/**
 *  整个生命周期只会调用一次
 */
+ (void)initialize
{
    [UITabBar appearance].translucent = NO;
    [UITabBar appearance].barTintColor = kColorTabBar;
}

- (instancetype)init
{
    if (self = [super init])
    {
        [self setupChildViewControllers];
    }
    return self;
}

//#pragma mark - Life Circle
- (void)viewDidLoad
{
    [super viewDidLoad];
}

//#pragma mark - private methods
/**
 *  初始化子控制器
 */
- (void)setupChildViewControllers
{
   	// 初始化子控制器
	// 治未病
    TDCTreatViewController *treatVC = [[TDCTreatViewController alloc] init];
    BaseNavigationController *treatNavVC = [self generateViewController:treatVC withTabBarItemTitle:@"治未病" tabBarItemTitleNormalColor:kTabBarItemTitleNormalColor tabBarItemNormalImage:@"tabBar_treat" tabBarItemTitleSelectedColor:kTabBarItemTitleSelectedColor tabBarItemSelectedImage:@"tabBar_treat_selected"];
    
   // 更多
    TDCMoreViewController *discoverVC = [[TDCMoreViewController alloc] init];
    BaseNavigationController *discoverNavVC = [self generateViewController:discoverVC withTabBarItemTitle:@"更多" tabBarItemTitleNormalColor:kTabBarItemTitleNormalColor tabBarItemNormalImage:@"tabBar_more" tabBarItemTitleSelectedColor:kTabBarItemTitleSelectedColor tabBarItemSelectedImage:@"tabBar_more_selected"];
    
   // 我的
    TDCMineViewController *mineVC = [[TDCMineViewController alloc] init];
    BaseNavigationController *mineNavVC = [self generateViewController:mineVC withTabBarItemTitle:@"我的" tabBarItemTitleNormalColor:kTabBarItemTitleNormalColor tabBarItemNormalImage:@"tabBar_mine" tabBarItemTitleSelectedColor:kTabBarItemTitleSelectedColor tabBarItemSelectedImage:@"tabBar_mine_selected"];
    
    
   self.viewControllers = @[treatNavVC,discoverNavVC,mineNavVC];
}

/**
 *  快速创建TaBarCotroller的子控制器
 *
 *  @param vc                           子控制器
 *  @param tabBarItemTitle              子控制器tabBarItem的文字
 *  @param tabBarItemTitleNormalColor   子控制器tabBarItem的文字颜色
 *  @param tabBarItemNormalImage        子控制器tabBarItem的图片
 *  @param tabBarItemTitleSelectedColor 子控制器tabBarItem的选中文字颜色
 *  @param tabBarItemSelectedImage      子控制器tabBarItem的选中图片
 *
 *  @return 封装好了的一个自控制器
 */
- (BaseNavigationController *)generateViewController:(UIViewController *)vc withTabBarItemTitle:(NSString *)tabBarItemTitle tabBarItemTitleNormalColor:(UIColor *)tabBarItemTitleNormalColor tabBarItemNormalImage:(NSString *)tabBarItemNormalImage
    tabBarItemTitleSelectedColor:(UIColor *)tabBarItemTitleSelectedColor  tabBarItemSelectedImage:(NSString *)tabBarItemSelectedImage
{
    // 用vc压入导航控制器栈
    BaseNavigationController *navVC = [[BaseNavigationController alloc] initWithRootViewController:vc];
    
                              
   // 设置文字
    navVC.tabBarItem.title = tabBarItemTitle;
    
   // 设置文字颜色
    [navVC.tabBarItem setTitleTextAttributes:@{NSForegroundColorAttributeName:kTabBarItemTitleNormalColor,NSFontAttributeName:kTabBarItemTitleFont} forState:UIControlStateNormal];
    [navVC.tabBarItem setTitleTextAttributes:@{NSForegroundColorAttributeName:kTabBarItemTitleSelectedColor,NSFontAttributeName:kTabBarItemTitleFont} forState:UIControlStateSelected];
    
   // 设置图片
   [navVC.tabBarItem setImage:[[UIImage imageNamed:tabBarItemNormalImage] imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal]];
    [navVC.tabBarItem setSelectedImage:[[UIImage imageNamed:tabBarItemSelectedImage] imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal]];
    
   return navVC;
}

</pre></code>

##See You