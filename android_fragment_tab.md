#Android Fragment实现底部Tab

Fragment加入Android的Intent（哈哈）就是要解决各种屏幕中，尤其是大屏幕只使用Activity难以
适配的问题。其中用来解决Tab的问题也就显得非常方便了。今天就用fragment实现这个tab。个人Activity翻译称活动，Fragment
翻译成碎片，感觉非常奇怪。所以专有名词就直接用了，不知道意思你也知道是干嘛用的。

	默认的环境是Android4.0，Android Studio 1.4。

###一、Fragment的生命周期。
每次都是从生命周期开始的。这里只是简单介绍，起一个承上启下的作用。
虽然说Fragment的生命周期是和其依附的Activity直接相关的，但是还有有自己的一套东西的。
尤其是和Activity互操作以及界面生成直接相关的部分。
<br />
>   **1.** &nbsp;onAttach(Activity) Fragment和Activity相互关联的时候调用一次。<br/>
>   **2.** &nbsp;onCreate(Bundle) 初次创建Fragment的时候调用。<br/>
>	**3.** &nbsp;onCreateView(LayoutInflater, ViewGroup, Bundle) 创建并返回和Fragment相关的视图。	  <br/>
> 	**4.** &nbsp;onActivityCreated(Bundle) 告诉Fragment他的Activity已经创建完毕。<br/>
>	**5.** &nbsp;onViewStateRestored(Bundle) 告诉Fragment，全部的View的状态已经复原完毕。<br/>
>	**6.** &nbsp;onStart() 使Fragment对用户可见。<br/>
>	**7.** &nbsp;onResume() 使Fragment可操作。<br/>
>	**8.** &nbsp;onPause() Fragemnt不可操作。可能是其Activity被暂停，也可能是因为activity中执行了对这个fragment的修改操作。<br/>
> 	**9.** &nbsp;onStop() Fragment对用户不可见。可能是其activity被终止，也可能是activity中执行了对这个fragment的修改操作<br/>
>	**10.** onDestroyView() 允许fragment清空全部与其view相关的资源。<br/>
>	**11.** onDestroy() 做fragment最后的状态清除。<br/>
>	**11.** onDetach() 在fragment不再和activity关联之前的最后一刻调用。<br/>

显然fragment生命周期的回调方法比activity的多一些。虽然有些名字是一样的，但是其含义却不一定
完全相同。所以，我们这里都列出来。

有了Fragment以后我们要如何使用呢。现在，新建一个Activity。然后在*onCreate(Bundle)*方法
中调用*getFragmentManager()*，你就会得到一个FragmentManager。
  >`FragmentManager fm = getFragmentManager();`
查找一个fragment：
>`fm.`
  
  
  
  
  
  
  
  
