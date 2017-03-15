---
layout: post
title:  "tableViewCell 注册的一些区别"
date:   2017-03-15 09:56:03 +0800
categories: jekyll update
---

# tableViewCell 注册的一些区别

##### 1.在 `viewIDidLoad` 中注册

* 不使用 `xib` 

```objective-c
[self.tableView registerClass:[UITableViewCell class] forCellReuseIdentifier:@"cell"];
```

```objective-c
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
  UITableViewCell * cell = [tableView dequeueReusableCellWithIdentifier:@"cell"] forIndexPath:indexPath];
}
```

* 使用 `xib`

```objective-c
[tableView registerNib:[UINib nibWithNibName:@"xxxxCell" bundle:nil] forCellReuseIdentifier:@"Cell"];
```

```objective-c
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
  xxxxCell *cell = [tableView dequeueReusableCellWithIdentifier:@"Cell" forIndexPath:indexPath];
}
```

##### 2.在创建 `cell` 的时候注册

* 不使用 `xib`

```objective-c
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
  UITableViewCell * cell = [tableView dequeueReusableCellWithIdentifier:@"cell"];
  //如果队列中没有该类型cell，则会返回nil，这个时候就需要自己创建一个cell,dequeueReusableCellWithIdentifier方法的区别所导致，新方法可以自动生成一个默认style的cell，不需要调用👇的方法。
  if (cell == nil) {
   cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:@"cell"];
 }
}
```

* 使用 `xib`

```objective-c
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
  xxxxCell *cell=[tableView dequeueReusableCellWithIdentifier:@"cell"];
  //如果队列中没有该类型cell，则会返回nil，这个时候就需要自己创建一个cell,dequeueReusableCellWithIdentifier方法的区别所导致，新方法可以自动生成一个默认style的cell，不需要调用👇的方法。
    if (cell == nil) {
        cell=[[[NSBundle mainBundle]loadNibNamed:@“xxxxCell" owner:self options:nil] lastObject];
    }
}	
```

