转自
# BaseiOSProject
一个iOS通用的基础工程，下载下来直接以此开始你的项目（偷师于5年iOS经验上司）
作者: Thebloodelves
地址: http://www.jianshu.com/p/fe8d765bc6dc
16届毕业生跟上司(五年iOS经验), 在持续半年的项目中总结的一点项目架构感悟, 虽文章用简单的语言阐述了自己对项目架构的理解, 但这种接地气的真实, 不正是我们需要的吗? 
欢迎大家在下方留言回复您对文章的看法哦!!!
写作原因

因为第一份工作有幸和5年iOS经验上司一起从头开始写项目（项目持续了半年），所以对于项目架构有点感悟，在这里献给大家（90%还原上司项目基础架构），先给一个码云地址（github不知道抽什么风，提交不上去）
https://git.oschina.net/liyongshi.com/BaseiOSProject.git
你们下载下来后可以直接以此开始写你们的项目（已经拿来写过三个公司项目了：快活、快照和帮帮管理助手），所以肯定是没问题的，常用的第三方、分类、工具、网络封装(基于AFN)和本地缓存（realm）等常用的都已经做好了（UtilKits.h中可查看）；并且我已经把业务的前两层做好了（就是登陆、欢迎界面、首页转换逻辑，只供参考你们可以修改为你们的，第三步的需要自己跟着写下代码），你们可以按照这个思路来搭建页面和业务；这个系列总体分为两部分：介绍和使用；当然了，每个人都有每个人的搭建思路，互相借鉴嘛！

1先瞄一下目录结构


目录结构

UtilKits目录：顾名思义，这个目录用来存放分类、非pod导入的第三方库和设备信息等“辅助”元素，比如你还可以建立一个文件夹ProductInfo表示工程信息等等，然后全局文件（分类、AFN等）在UtilKits.h中导入；

AppDelegate目录：里面就是放了AppDelegate.h和AppDelegate.m文件，另起文件夹保留目录层级；

AppCustoms目录：自己写的轮子放到这里，比如图片多选等

ModelManager目录：存放模型、请求和模型管理，如上图中的Models中放模型，IdentityHttp是登录相关的请求（因为不用带入用户信息），IdentityManager操作缓存登录数据（读取、保存）和登录相关行为（退出）；其实看UserManager更一目了然，但是太长了截图截不完，你们可以看看UserManager

InterfaceService目录：网路请求封装和地址宏

MainViewController目录：这里就是你的界面了，按照权限来存放，比如MainViewController管理登录（LoginController）、欢迎界面（WelcomeController）和登录后的界面（BusinessController），那么这三个文件夹就作为MainViewController目录的子目录，采用addChildViewController进行管理

好了，目录结构介绍完了，你们可以各取所需；比如你想要个定位管理器，那么你在和ModelManager同层目录建一个LocationManager即可，然后你有其他的分类分别放入对应的文件夹即可；当然了还有很多比如第三方登录那些，你自己在相应位置加上就可以了（因为不一定所有项目都有三方登录，但是所有工程都有这个基础工程中的内容，这里给的的不就是一个基础工程吗）

2部分值得说的思想

A
 本地数据库采用realm（你可能会问了，现在很多数据库啊，为啥这都要说），我们知道CoreData的强大是因为它有个NSFetchedResultsController和能直接存入对象（来自上司原话），存/取对象目前来说是基本要求了，那么你知道NSFetchedResultsController吗（不知道的去谷歌吧），然后我找了一下除了CoreData之外还有FMDB和realm有类似的库支持，但是FMDB不能直接存取对象我直接抛弃了，然后我找到了realm，它有一个专有库RBQFetchedResultsController来实现 CoreData的NSFetchedResultsController效果，这个非常有用，让我们可以轻松的做到界面实时显示最新数据（这才是我选realm的原因，前提是很难掌握CoreData）；

B
本地数据存取每个用户是一个单独的文件夹，比如用户id为1的有一个名为1文件夹，id为2的有一个名为2的文件夹，然后公共的数据可以放到名为0的文件夹然后单独一个manager管理（因为usermanager是绑定了固定用户私有数据的）；有的软件有游客身份，其实就是读取的默认文件夹0/default的数据（按照我这种思想），读取的时候可以看到UserManager中loadUserWithNo：函数就可以读取到对应用户数据了，而且这样做可以很容易迁移数据（我经历过旧数据迁移的疼，从最开始杜绝问题）；

C
网络请求返回的数据和界面显示需要的数据不直接关联，他们和本地数据库直接关联，这是非常重要的一个思想，我们最开始的时候请求回调中直接把数据赋值给tableview的datasource，然后reloaddata还记得吗？这很容易造成显示错误，最明显的就是几个页面都对同一个对象数组进行操作会很难把控，所以我们正确的思路应该是：数据请求得到的数据我们直接存入数据库然后就可以结束了，数据库得到数据后发送一个回调（RBQFetchedResultsController的作用），然后所有RBQFetchedResultsController监听的页面重新从本地获取最新数据再赋值给tableview的datasource后reloaddata，这样的话请求就是请求没有其他的操作，数据变化监听由数据库特性发出；

D
权限（包括业务层次结构和manager提供的接口合理性等），我们在公司都是各司其职，不能越权，那么同样的软件也是有生命的（可以，很强势）；至少我们不能犯这种低级错误：我们的软件基本都是有退出登录功能的，那么你至少不应该在退出登录点击事件执行self.window.rootViewcontroller = [LoginViewcontroller new]，因为self.window.rootViewcontroller是固定的MainViewController,这个操作应该是发通知到MainViewController让它移除BusinessController（退出按钮肯定在这里面）并加载LoginController，这就是所谓的越权了；还有就是如果tableviewcell上的一个按钮点击后导航控制器要push到指定界面，你至少应该是tableviewcell的代理对象（一直到UIViewController）再push（虽然有人提出一个self-manager模式，但是我是怎么都不情愿那么做的）到指定界面；其他的还有很多，做之前想想应该是谁来做怎么做就可以了；

E
少用轮子尽量拆轮子，1：轮子大多数是综合考虑的，所以体积会很大而且里面的逻辑可能还会和你现有的逻辑冲突，比如解决键盘遮挡的IQKeyboardManager是检测键盘的弹起于消失然后使window的frame改变了，那么如果你自己对window有其他的操作就会冲突，2：用着是挺舒服的，但是你会慢慢的忘记很多知识，比如图片多选我们喜欢网上去找轮子但是又和自己的需求不一样于是又去找另外一个，其实自己做比找快多了其实我们都会做只是懒，其实你把轮子看懂自己拆分取其中一部分也是好的

3用此基础工程做个小demo实现界面显示实时刷新

其实大部分我们都写过，这里我只讲一下RBQFetchedResultsController怎么使用，RBQFetchedResultsController能解决什么问题？
1：界面实时刷新，我们模型改变了通常是发一个通知然后要实时的界面接收通知，那么你有想过一个界面有很多模型的恐怖吗，而且前面也说了这种数据变化监听应该由数据库特性发出，
2：和第一个有点异曲同工，也就是请求和显示分离，请求就是请求，不要去管显示，这样做有好处，也就是你可以在应用任何地方请求网络，其他所有地方都自动会刷新；好了下面我们来写个简单的demo，我们在AppDelegate.m中填充假数据：


填充假数据

然后我们在BusinessController文件夹中创建一控制器FirstController（带xib，为了方便），在中间展示当前用户的名字，下面放一个输入框，输入框下边一个按钮点击后修改用户名字为输入框的内容，然后创建一个RBQFetchedResultsController代理设置成自己以便获取最新的用户信息，导航右边按钮不断创建自身并push以测试效果，界面和代码如下：

demo界面


FirstController.m上面代码



FirstController.m下面代码

上面的RBQFetchedResultsControllerDelegate基本还原CoreData的FetchedResultsControllerDelegate（数据库操作会触发这些回调，很强势），然后我们BusinessController（登录后的界面）中创建一个导航视图，rootviewcontroller设置成FirstController，代码就像这样：

BusinessController.m代码

然后我们启动模拟器，多点下右上角的“+”号创建几个FirstController实例，然后我们输入内容点击按钮，你再退回来看看是不是所有界面都变了？效果如下：


效果图

这部分的代码工程里面没有，你们跟着写一下实践一下；好了，你现在可以拿着这个基础工程开始你自己的项目了，对了realm有几个坑，我在
https://github.com/TheBloodElf/iOSDevNotices/issues
中记录了一些，如果你遇到了你可以先去看看；最后，希望这个基础工程可以为你减少项目开发的时间
