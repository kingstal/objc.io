# Issue1-4 View Controller 容器



```objc
- (void)viewDidLoad
{
    [super viewDidLoad];

    //Setup controllers
    _startMapViewController = [RGMapViewController new];
    [_startMapViewController setAnnotationImagePath:@"man"];
    [self addChildViewController:_startMapViewController];          //  1
    [topContainer addSubview:_startMapViewController.view];         //  2
    [_startMapViewController didMoveToParentViewController:self];   //  3
    [_startMapViewController addObserver:self
                              forKeyPath:@"currentLocation"
                                 options:NSKeyValueObservingOptionNew
                                 context:NULL];

    _startGeoViewController = [RGGeoInfoViewController new];        //  4
}

我们实例化了 _startMapViewController，用来显示起始位置，并设置了用于标注的图像。

1._startMapViewcontroller 被添加成 root view controller 的一个 child。这会自动在 child 上调用 willMoveToParentViewController: 方法。
2. child 的 view 被添加成 container view 的 subview。
3. child 被通知到它现在有一个 parent view controller。
4. 用来显示地理位置的 child view controller 被实例化了，但是还没有被插入到任何 view 或 controller 层级中。

```

### 布局

Root view controller 定义了两个 container views，它决定了 child view controller 的大小。Child view controllers 不知道会被添加到哪个容器中，因此必须适应大小。


```objc
- (void) loadView
{
    mapView = [MKMapView new];
    mapView.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
    [mapView setDelegate:self];
    [mapView setMapType:MKMapTypeHybrid];

    self.view = mapView;
}
```
现在，它们就会用 super view 的 bounds 来进行布局。这样增加了 child view controller 的可复用性；如果我们把它 push 到 navigation controller 的栈中，它仍然会正确地布局。

### 过场动画

Apple 已经针对 view controller 容器做了细致的 API，我们可以构造我们能想到的任何容器场景的动画。Apple 还提供了一个基于 block 的便利方法，来切换屏幕上的两个 controller views。方法 transitionFromViewController:toViewController:(...) 已经为我们考虑了很多细节。

```objc
- (void) flipFromViewController:(UIViewController*) fromController
               toViewController:(UIViewController*) toController
                  withDirection:(UIViewAnimationOptions) direction
{
    toController.view.frame = fromController.view.bounds;                           //  1
    [self addChildViewController:toController];                                     //
    [fromController willMoveToParentViewController:nil];                            //

    [self transitionFromViewController:fromController
                      toViewController:toController
                              duration:0.2
                               options:direction | UIViewAnimationOptionCurveEaseIn
                            animations:nil
                            completion:^(BOOL finished) {

                                [toController didMoveToParentViewController:self];  //  2
                                [fromController removeFromParentViewController];    //  3
                            }];
}

1. 在开始动画之前，我们把 toController 作为一个 child 进行添加，并通知 fromController 它将被移除。如果 fromController 的 view 是容器 view 层级的一部分，它的 viewWillDisappear: 方法就会被调用。
2. toController 被告知它有一个新的 parent，并且适当的 view 事件方法将被调用。
3. fromController 被移除了。
这个为 view controller 过场动画而准备的便捷方法会自动把老的 view controller 换成新的 view controller。然而，如果你想实现自己的过场动画，并且希望一次只显示一个 view，你需要在老的 view 上调用 removeFromSuperview，并为新的 view 调用 addSubview:。错误的调用次序通常会导致 UIViewControllerHierarchyInconsistency 警告。例如：在添加 view 之前调用 didMoveToParentViewController: 就触发这个警告。
```

为了能使用 UIViewAnimationOptionTransitionFlipFromTop 动画，我们必须把 children's view 添加到我们的 view containers 里面，而不是 root view controller 的 view。否则动画将导致整个 root view 都翻转。

### 通信

View controllers 应该是可复用的、自包含的实体。Child view controllers 也不能违背这个经验法则。为了达到目的，parent view controller 应该只关心两个任务：布局 child view controller 的 root view，以及与 child view controller 暴露出来的 API 通信。它绝不应该去直接修改 child view tree 或其他内部状态。

Child view controller 应该包含管理它们自己的 view 树的必要逻辑，而不是把它们看作单纯呆板的 views。这样，就有了更清晰的关注点分离和更好的可复用性。

从 child 到 parent view controller 消息传递的技术，不论是采用 KVO，通知，或者是委托模式，child view controller 都应该独立和可复用。


