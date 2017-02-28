# Issue1-2 整洁的 Table View 代码

## 分离关注点（Separating Concerns）
当处理 table views 的时候，有许多各种各样的任务，这些任务穿梭于 models，controllers 和 views 之间。为了避免让 view controllers 做所有的事，我们将尽可能地把这些任务划分到合适的地方，这样有利于阅读、维护和测试。

### 搭建 Model 对象和 Cells 之间的桥梁
为了不向 data source 暴露 cell 的设计，最好分解出来，放到 cell 类的一个 category 中。


```objc
@implementation PhotoCell (ConfigureForPhoto)

- (void)configureForPhoto:(Photo *)photo
{
    self.photoTitleLabel.text = photo.name;
    NSString* date = [self.dateFormatter stringFromDate:photo.creationDate];
    self.photoDateLabel.text = date;
}

@end

//有了上述代码后，我们的 data source 方法就变得简单了。
- (UITableViewCell *)tableView:(UITableView *)tableView
         cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    PhotoCell *cell = [tableView dequeueReusableCellWithIdentifier:PhotoCellIdentifier];
    [cell configureForPhoto:[self itemAtIndexPath:indexPath]];
    return cell;
}
```

### 在 Cell 内部控制 Cell 的状态
努力把 view 层和 controller 层的实现细节分离开。delegate 肯定得清楚一个 view 该显示什么状态，但是它不应该了解如何修改 view 结构或者给某些 subviews 设置某些属性以获得正确的状态。所有这些逻辑都应该封装到 view 内部，然后给外部提供一个简单的 API。

### 总结

Table view controllers（以及其他的 controller 对象！）应该在 model 和 view 对象之间扮演协调者和调解者的角色。它不应该关心明显属于 view 层或 model 层的任务。你应该始终记住这点，这样 delegate 和 data source 方法会变得更小巧，最多包含一些简单的样板代码。

这不仅减少了 table view controllers 那样的大小和复杂性，而且还把业务逻辑和 view 的逻辑放到了更合适的地方。Controller 层的里里外外的实现细节都被封装成了简单的 API，最终，它变得更加容易理解，也更利于团队协作。


