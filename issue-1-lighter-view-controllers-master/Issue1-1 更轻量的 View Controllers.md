# Issue1-1 更轻量的 View Controllers

### 把 Data Source 和其他 Protocols 分离出来
TableView 一般是围绕 view controller 所管理的 photos 数组做一些事情。我们可以尝试把数组相关的代码移到单独的类中。我们使用一个 block 来设置 cell，也可以用 delegate 来做这件事。

```objc
@implementation ArrayDataSource

- (id)itemAtIndexPath:(NSIndexPath*)indexPath {
    return items[(NSUInteger)indexPath.row];
}

- (NSInteger)tableView:(UITableView*)tableView
 numberOfRowsInSection:(NSInteger)section {
    return items.count;
}

- (UITableViewCell*)tableView:(UITableView*)tableView
        cellForRowAtIndexPath:(NSIndexPath*)indexPath {
    id cell = [tableView dequeueReusableCellWithIdentifier:cellIdentifier
                                              forIndexPath:indexPath];
    id item = [self itemAtIndexPath:indexPath];
    configureCellBlock(cell,item);
    return cell;
}

@end

// 单独使用一个类来管理 tableView 的 dataSource
TableViewCellConfigureBlock configureCell = ^(PhotoCell *cell, Photo *photo) {
        [cell configureForPhoto:photo];
    };
NSArray *photos = [AppDelegate sharedDelegate].store.sortedPhotos;
self.photosArrayDataSource = [[ArrayDataSource alloc] initWithItems:photos
                                                        cellIdentifier:PhotoCellIdentifier
                                                    configureCellBlock:configureCell];
self.tableView.dataSource = self.photosArrayDataSource;
[self.tableView registerNib:[PhotoCell nib] forCellReuseIdentifier:PhotoCellIdentifier];
```

### 将业务逻辑移到 Model 中
把一些代码移动到 Model 类的 category 中会变得更加清晰

### 创建 Store 类
有些代码不能被轻松地移动到 model 对象中，但明显和 model 代码紧密联系，对于这种情况，我们可以使用一个 Store

### 把网络请求逻辑移到 Model 层

不要在 view controller 中做网络请求的逻辑。取而代之，你应该将它们**封装到另一个类中**。这样，你的 view controller 就可以在之后通过使用回调（比如一个 completion 的 block）来请求网络了。这样的好处是，缓存和错误控制也可以在这个类里面完成。

### 把 View 代码移到 View 层

不应该在 view controller 中构建复杂的 view 层次结构。你可以使用 Interface Builder 或者把 views 封装到一个 UIView 子类当中。

### 通讯
当一个 view controller 想把某个状态传递给多个其他 view controllers 时，就会出现这样的问题。较好的做法是把状态放到一个单独的对象里，然后把这个对象传递给其它 view controllers，它们观察和修改这个状态。这样的好处是消息传递都在一个地方（被观察的对象）进行，而且我们也不用纠结嵌套的 delegate 回调。


