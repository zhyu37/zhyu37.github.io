---
layout: post
title: "ios8之UIAlertController"
date: 2015-10-13 17:16:02 +0800
comments: true
categories: 
---
=====
##UIAlertController
>UIAlertController 作为ios8新出现的control还是很值得学习的，它融合了之前的UIAlertView 和  UIActionSheet.

<P>先来写个简单的小例子</P>

<pre><code>
UIAlertController *alertController = [UIAlertController alertControllerWithTitle:@"title" message:@"message" preferredStyle:UIAlertControllerStyleAlert];

UIAlertAction *cancelAction = [UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
            
}];

 UIAlertAction *commitAction = [UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDestructive handler:^(UIAlertAction * _Nonnull action) {
 
}];

[alertController addAction:cancelAction];
[alertController addAction:commitAction];
[self presentViewController:alertController animated:YES completion:nil];
</pre></code>

![样例](/Users/zhanghaoyu/octopress/source/_posts/image/UIAlertConller1.png)
<p>在UIAlertController中添加一个输入框</p>

<pre><code>
@property (nonatomic, strong) UIAlertAction *secureTextAlertAction;
</pre></code>

<pre><code>
UIAlertController *alertController = [UIAlertController alertControllerWithTitle:@"title" message:@"message" preferredStyle:UIAlertControllerStyleAlert];

 // Add the text field for the secure text entry.
    [alertController addTextFieldWithConfigurationHandler:^(UITextField *textField) {
        // Listen for changes to the text field's text so that we can toggle the current
        // action's enabled property based on whether the user has entered a sufficiently
        // secure entry.
        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(handleTextFieldTextDidChangeNotification:) name:UITextFieldTextDidChangeNotification object:textField];
        //每输入一个字符就会变成点
  		 textField.secureTextEntry = YES;
 }];

UIAlertAction *cancelAction = [UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
        [[NSNotificationCenter defaultCenter] removeObserver:self name:UITextFieldTextDidChangeNotification object:alertController.textFields.firstObject];
    }];

 UIAlertAction *commitAction = [UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDestructive handler:^(UIAlertAction * _Nonnull action) {
 	[[NSNotificationCenter defaultCenter] removeObserver:self name:UITextFieldTextDidChangeNotification object:alertController.textFields.firstObject];
}];

// The text field initially has no text in the text field, so we'll disable it.
    commitAction.enabled = NO;
    
// Hold onto the secure text alert action to toggle the enabled/disabled state when the text changed.
    self.secureTextAlertAction = commitAction;

[alertController addAction:cancelAction];
[alertController addAction:commitAction];
[self presentViewController:alertController animated:YES completion:nil];

</pre></code>


<pre><code>
- (void)handleTextFieldTextDidChangeNotification:(NSNotification *)notification
{
    UITextField *textField = notification.object;
    
   // Enforce a minimum length of >= 5 characters for secure text alerts.
    self.secureTextAlertAction.enabled = textField.text.length >= 5;
}
</pre></code>
###See You