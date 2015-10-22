---
layout: post
title: "UIPickerView"
date: 2015-10-22 23:25:30 +0800
comments: true
categories: 
---
===
##UIPickerView
>UIPickerView是一个类似于选择器的东西。

* UIPickerViewDelegate
	* //pikerView 的长度
	(CGFloat)pickerView:(UIPickerView *)pickerView widthForComponent:(NSInteger)component;
	* //pickerView 的宽度 
	(CGFloat)pickerView:(UIPickerView *)pickerView rowHeightForComponent:(NSInteger)component;
	* //pickerView 里面的信息
	(NSString *)pickerView:(UIPickerView *)pickerView titleForRow:(NSInteger)row forComponent:(NSInteger)component;
	* //当被点击后执行
	(void)pickerView:(UIPickerView *)pickerView didSelectRow:(NSInteger)row inComponent:(NSInteger)component;
* UIPickerViewDataSource
	* //pickerView 的个数
	(NSInteger)numberOfComponentsInPickerView:(UIPickerView *)pickerView
	* //每个pickerView 的行数
	(NSInteger)pickerView:(UIPickerView *)pickerView numberOfRowsInComponent:(NSInteger)component
	

<p>pickerView初始化</p>
<pre><code>
@property (nonatomic, strong) UIPickerView *pickerView;


- (void)viewDidLoad {
// 显示选中框 default = NO
    self.pickerView.showsSelectionIndicator=YES;
    self.pickerView.dataSource = self;
    self.pickerView.delegate = self;
    [self.view addSubview:self.pickerView];
}


- (UIPickerView *)pickerView
{
    if (!_pickerView) {
        _pickerView = [[UIPickerView alloc] initWithFrame:CGRectMake(0, 0, 320, 480)];
    }
    return _pickerView;
}
</pre></code>


##See You