---
title: UIView+FDCollapsibleConstraints
date: 2016-06-03 10:24:19
categories: "iOS"
tags: 
    - UITableView
---
本篇要推荐的是一个AutoLayout辅助工具UIView+FDCollapsibleConstraints.
##工具的作用:
UIView+FDCollapsibleConstraints很好地解决了动态布局中某些子视图的隐藏与显示问题,重点就是动态布局。
- [UIView+FDCollapsibleConstraints的gtihub地址](https://github.com/forkingdog/UIView-FDCollapsibleConstraints)
- 单个View的效果图:
![](http://upload-images.jianshu.io/upload_images/1801509-3f5ccc67eaedfedc.gif?imageMogr2/auto-orient/strip)

- 再看看动态布局中啥样:

![autolayout.gif](http://upload-images.jianshu.io/upload_images/1801509-9c7f5bc3fac88320.gif?imageMogr2/auto-orient/strip)

##看了它的强大之处,下面介绍它是怎样使用的
- 首先,在xib中正常约束完一个cell就好了
- 接着,给需要折叠的控件的自身高度约束: height约束 拉一条线
![autolayout2.gif](http://upload-images.jianshu.io/upload_images/1801509-004bf7b25baf4fed.gif?imageMogr2/auto-orient/strip)

- 然后,与它底部的控件的 top约束拉一条线
![autolayout3.gif](http://upload-images.jianshu.io/upload_images/1801509-e488d94f71e4db59.gif?imageMogr2/auto-orient/strip)
- 现在大部分的工作已经做完了,是不是很轻松

![944612A1-2ED4-4E9C-92E7-AFB816B4DC10.png](http://upload-images.jianshu.io/upload_images/1801509-353dab26a827d6fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 接下来我们要做的就是去代码中设置它的隐藏与现实
<pre><code>
_ActivityView.fd_collapsed = YES;//表示隐藏

_ActivityView.fd_collapsed = NO;//表示显示
</pre></code>

- 下面来看一下效果图

![autolayout4.gif](http://upload-images.jianshu.io/upload_images/1801509-37f0e720fa5658a3.gif?imageMogr2/auto-orient/strip)
是不是很尼玛叼

需要注意的是动态布局还需要配合另一个工具:UITableView+FDTemplateLayoutCell一起使用。

简单阐述一下它的作用:
对 UITableViewCell 利用 AutoLayout 自动高度计算和 UITableView 滑动优化。
- [UITableView+FDTemplateLayoutCell的github地址](https://github.com/forkingdog/UITableView-FDTemplateLayoutCell)
- 同样简单好用
<pre><code>
-(CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
{
return [tableView fd_heightForCellWithIdentifier:@"FDTableViewCell" configuration:^(FDTableViewCell *cell) {
//这边要写的与cellforRaw中的赋值一样
cell.entity = self.entities[indexPath.row];
}];
}
</pre></code>
- 同时他还有缓存高度的方法:
- 多种标识时,根据index缓存高度
<pre><code>-(CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath 
{
return [tableView fd_heightForCellWithIdentifier:@"identifer" cacheByIndexPath:indexPath configuration:^(id cell) {
// configurations
}];
}
</pre></code>
- 只有唯一标识的cell的时候,我们可以根据数据唯一字段(如用户id)来缓存高度,不过我没用过= =
<pre><code>- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
Entity *entity = self.entities[indexPath.row];
return [tableView fd_heightForCellWithIdentifier:@"identifer" cacheByKey:entity.uid configuration:^(id cell) {
// configurations
}];
}
</pre></code>

使劲夸我不要客气。