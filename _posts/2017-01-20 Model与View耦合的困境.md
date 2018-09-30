---
date: 2017-01-20 16:09
status: public
title: '2017-01-20 Model与View耦合的困境'
---

    iOS最为熟悉的模式莫过于MVC，我们整天都在写ViewController，View，Model。我想谈一谈自己在项目开发过程对于Model与View耦合问题认识。

## 现实中的MVC
在现实开发过程中，我们经常能够见到几千行甚至更为庞大的View Controller。这有时候让我们非常头疼，文件太大连拉个滚动条都觉得了累更别说去维护它了。在UIKit的体系结构设计下，我们非常容易写出这种怪兽级别Controller，有一部分原因的确是Apple MVC设计本身的问题。我们可以看到Apple的View是非常“瘦的”，几乎不包含任何显示以外的逻辑。我们设计的数据Model的时候通常不包含任何逻辑的“瘦”Model。在这种设计之下，我们习惯于把所有逻辑代码都放到ViewController里面，然后它自然不断地长大。这样的结构在项目比较复杂的时候将会产生难以维护的代码。

## View与Model的耦合
我们先来看一段常见的ViewController代码：

```objective-c
 UITableViewCell *cell = ...;
 cell.model = model;
 return cell;
``` 

这种代码应该是随处可见，看起来并没有什么大问题。将数据结构传给view，然后让View自己去渲染。但其实这种写法本身就已经违背了Apple MVC的定义。View的渲染应该是交给View Controller去做才符合Apple MVC的定义，也就是说我们需要把View的渲染代码放到ViewController里，然而这就加剧了ViewController的代码量，这也不是我们看到的。

那么直接将Model传给View，有什么问题呢？View知道Model的细节，也就是View与Model存在紧耦合问题。大家都知道耦合是不好的，我举个一个自己遇到的问题。

我们需要显示两个View:
一个简单用户信息：UserInfoCell
```objective-c
  @interface UserInfoCell : UIView
  @property(nonatomic,weak) UIImageView *image;
  @property(nonatomic,weak) UILabel *nameLabel;

  -(void)setModel:(UserInfo *)model;
  @end
```

一个用户信息详细信息的UserDetailView：
  
```objective-c
  @interface UserDetailView : UIView
  @property(nonatomic,weak) UIImageView *image;
  @property(nonatomic,weak) UILabel *nameLabel;
  @property(nonatomic,weak) UILabel *sexLabel;
  @property(nonatomic,weak) UILabel *ageLabel;
  @property(nonatomic,weak) UILabel *moreLabel;
 ...
    
  -(void)setModel:(UserInfo *)model;
  @end
```
这两个View其实都是呈现用户信息的逻辑，我们定义这样一个Model:
 
```objective-c
  @interface UserInfo : NSObject
  @property(nonatomic,copy) NSString *name;
  @property(nonatomic,copy) NSString *imageURL;
    ...
  @end
```
看起来非常直接我们只需要调用setModel方法将model注入到view，让后view负责去渲染。
```objective-c
  UserInfo *info = ...;
  UserInfoCell * cell = ...;
  cell.model = info;
```
  
```objective-c
  UserInfo *info = ...;
  UserDetailView * view = ...;
  view.model = info;
```
这个时候假设我们需要在DetailView 加入一些额外的信息，比如增加等级。我们需要直接更改UserInfo，没有什么大问题。但假如我想在Cell中加入额外的信息，比如我要实现选择态，一个cell选择时候显示一种不同的颜色，那这个选择态和color的值我们应该放到哪里去呢？根据MVC的概念我们应该放到Model里面，因为View只是Model的呈现，这样实现比较自然。但是这些值不应该加入到我们这里的Model（UserInfo）当中。我们当然可以使用额外的变量，做到既不往UserInfo里面加东西也继续使setModel这种模式，这并不是推荐的做法。

通过这个简单的例子，我想说明model和view耦合的问题。view的改动直接需要影响到model结构，这不是一个好的做法。

## 解耦的尝试
很多解耦都是通过增加一个抽象层来解决的。一开始我尝试在model层面增加一层抽象接口，希望View不依赖特定的Model，只是依赖接口。比如UserInfo做成一个protocol，做成类似于：
```objective-c
-(void)setModel:(id<UserInfo>)model;
```
我觉得这里没有出问题，出问题的是我让不同的view依赖了相同的接口,这导致了一个巨大的坑！
```objective-c
UserInfoCell -> 依赖protocol UserInfo
UserDetailView -> 依赖protocol UserInfo
UserInfoImp -> 实现UserInfo
```
这样的话，我每次改动一个view如果改动UserInfo接口，导致所有实现或者依赖UserInfo接口的类都要改动，这太可怕了！这是我的一次失败的想法。后来我去查阅了不少App设计的资料，找到了一种较为合理的方案。

## MVVM

MVVM(Model ViewModel View)（网上有很多关于MVVM介绍的资料，我这里只谈谈自己对它实现View与Model的解耦的一些想法），在MVVM模式中加入了一层ViewModel来实现View与Model的解耦。View依赖于ViewModel并不知道背后的Model是什么。
```objective-c
View -> ViewModel -> Model
```
利用这种结构，我前面提到的问题都可以迎刃而解。当更改View的我们只需要改动ViewModel，至于ViewModel里面有什么View不关心，这样给了我们很多灵活性。在MVVM里面UIView/UIViewController都算是View这一层，ViewModel负责View的渲染和各种业务逻辑，这同时也做到了ViewController的瘦身。 MVVM的好处有很多，我这里不展开讨论了。

##总结
View与Model的紧耦合给我们开发带来了很多困扰，我相信MVVM是一种很好的解耦模式。当View和Model不再耦合，一切都变得开朗了起来，不得不说设计在项目越来越复杂的时候变得非常重要。有时间还是要多总结。