title: Activity启动模式
author: John Doe
tags: []
categories:
  - android
date: 2017-12-04 22:17:00
---

### Activity启动模式

Android系统是通过Activity栈的方式来管理Activity的，而按照栈的特性，系统每创建一个新的Activity都会把其实例放入栈中，而按back键退出时，则从栈顶移除该Activity。而这种管理方式太过简单粗暴，当重复启动一个Activity时，会创建很多个实例，因此在很多情况下无法满足需求，因此，Android系统使用了Activity启动模式并配合Activity栈来完成对系统中各个Activity的管理。

Activity启动模式有以下四种：
* standard
* singleTop
* singleTask
* singleInstance

在开发过程中，我们根据应用场景及需要的不同选择合适的启动模式。设置Activity的启动模式，只需要在AndroidManifest.xml里对应的<activity>标签设置android:launchMode属性。

```
<activity   
    android:launchMode="standard" /> 
```
**standard** 模式
默认模式，可以不用写配置。在这种模式下，每启动一个Activity都会创建一个新的实例，也不会理会该实例是否已经存在。一般情况下，谁启动了改模式的Activity，该Activity就会放入启动它的Activity所在的任务栈中。同时被启动的Activity的onCreate()，onStart()，onResume()方法都会被调用。也就是说这种模式下，同一个任务栈中允许有多个该Activity的实例，该Activity的多实例也可以同时存在于不同的任务栈中。

**singleTop** 模式
可以有多个实例，但是不允许多个相同Activity叠加。即，如果已经有一个Activity在当前任务栈的栈顶了，再去启动一个相同的Activity，那么则不会创建新的实例，而是直接调用其onNewIntent方法。该模式与standard模式不同之处在于如果当前任务栈顶已经有了该Activity的实例，那么就不会再去创建新的实例，其他特性则与standard模式完全相同。

**singleTask** 模式
只有一个实例。在同一个应用程序中启动的时候，若Activity不存在，则会在当前任务栈中创建一个新的实例，若存在，则会把任务栈中其他在其之上的其它Activity全部destory掉并调用其onNewIntent方法。如果是在别的应用程序中启动它，则会新建一个任务栈，并在该任务栈中启动这个Activity，singleTask允许别的Activity与其在一个任务栈中共存，也就是说，如果我在这个singleTask的实例中再打开新的Activity，这个新的Activity还是会在singleTask的实例的任务栈中。

**singleInstance** 模式
只有一个实例，并且这个实例独立运行在一个task中，这个task只有这个实例，不允许有别的Activity存在。以singleInstance模式启动的Activity在整个系统中是单例的，如果在启动这样的Activiyt时，已经存在了一个实例，那么会把它所在的任务调度到前台，重用这个实例。

**note:** 在详细分析了四种启动模式后，我们需要知道的是，加入启动模式后，启动一个Activity可能有两种方式，一种是创建一个新的Activity实例并启动，此时Activity的onCreate()，onStart()，onResume()方法都会被调用;而当Activity是复用已经存在的实例时，则会将其置于栈顶的同时去调用其onNewIntent方法。此时Activity的onCreate()，onStart()方法是不会被调用的，但onResume()会被调用。

### 使用Intent的flag设置启动模式

当你启动一个Activity时，你也可以动态的设置intent的flag，然后通过startActivity()方法启动activity，从而修改其启动的activity与它的task的关联模式。具体可以使用的flag有：

*  **FLAG_ACTIVITY_NEW_TASK**:对应之前的“singleTask”，在新的task中启动activity，如果一个你需要的activity的task已经存在，则将它推向前台，恢复其上一个状态，它通过onNewIntent()收到这个新的intent。

*  **FLAG_ACTIVITY_SINGLE_TOP**:对应之前的“singleTop”,如果被启动的activity是当前顶部的activity，则已经存在的实例会收到onIntent(),而不会重新去创建这个实例。

*  **FLAG_ACTIVITY_CLEAR_TOP**:这个行为在launchMode属性中没对应的属性值，若被启动的activity已经在当前task中运行，则不会创建它的新实例，而是的**销毁在它之上**的其他所有的activities，然后通过 onNewIntent()传递一个新的intent给这个恢复了的activity，它一般会与FLAG_ACTIVITY_NEW_TASK一起使用。值得注意的是，**如果activity的启动模式是"standard"，它自己也将被移除，然后一个新的实例将被启动。这是因为当启动模式是"standard"时，为了接收新的intent必须创建新的实例**。

### 任务共用性的处理(Handling affinities)

一般来说，singleTask就是开启一个新的Task，但是在实际使用过程中，我们有时会发现，有时候并不是这样的，这是因为我们定义了affinities,也就是任务公用性。

affinity定义了一个Activity将被分配到哪一个Task中。默认情况下，同一个app中得所有activity有一个同样的affinity，因此，默认情况下同一个应用程序中得所有activity都在同一个task中。一个Task的affinity由这个Task的根Actiivty决定。然而，我们可以修改Activity默认的affinity，这样，不同应用程序的Activity可以共用同样的affinity，或者同一个程序的不同Activity分配不同的affinity。一个应用程序默认的affinity就是应用程序的包名，所以，如果我们想定义一个不同的affinity，必须和默认的affinity不同。我们可以通过<application>中得android:taskAffinity修改整个程序的affinity，也可以通过<activity>的android:taskAffinity对单个Activity的affinity修改。

而affinity的使用通常有以下两种情况，

1.  当android:launchMode是singleTask或者Intent中包含FLAG_ACTIVITY_NEW_TASK:

默认情况下，我们调用startActivity()，会实例化一个Activity，放入到与调用者相同的task。但是如果这个Activity的的启动模式是singleTask，或者启动它的Intent包含了

FLAG_ACTIVITY_NEW_TASK时，系统会进行如下的步骤：

*   判断这个Activity有没有实例已经存在了，有的话，直接传递Intent到它的onNewIntent()方法中。

*   如果不存在，系统查找是否有与这个Activity相同affinity的Task已经存在，如果存在，那么就将这个Activity启动到这个Task中。

*   如果不存在这样的Task，那么系统就会创建一个新的Task，并且将这个Activity启动这个Task中，作为根Activity。

**因此，从现在看来，只是单纯的用singleTask指定Activity，是不能开辟一个新的Task的，因为我们并没有给他指定affinity。而官方文档对于singleTask的描述，都是基于我们使用了不同的affinity的前提下，只不过是省略了这个描述。所以，我们要明白，singleTask的正确用法，应该是结合affinity使用的。**

2.  当一个Activity设置allowTaskReparenting属性为true:

这个属性定义了一个Activity，表示是否可以从一个启动它的task，切换到与它相同affinity的task中里去（当这个task切换到前台的时候）。true表示可以移动，false表示它必须呆在启动他得task里。

通常情况下，当一个Activity启动了，那么它就会存在于启动它的task中，并且在整个生命周期中都留在这个task中。但是，我们可以通过这个属性，做出如下改变，当这个Activity当前的Task处于后台，这个时候如果有一个该Activity具有相同affinity的Task被启动到前台，那么这个Activity就可以从它之前的Task，移动到这个新的Task显示。

通常，它的作用是将app中得Activity与app的main task结合起来，举个例子，如下：

有一个e-mail程序，他需要调用浏览器程序的某个Activity（假设为Activity A）来显示一些数据，这个Activity A的该属性设置为true。现在，e-mail程序调用了这个Activity A，在用户看来，好像这个Activity A就是e-mial程序的一部分，因为这个Activity A和这个e-mail程序在同一个task中。现在将e-mail退出到后台，启动浏览器程序，因为Activity  A和浏览器程序有相同的affinity，所以Activity A从e-mail程序的Task移动到浏览器程序的Task，并显示在前台。当我们下次再启动e-mail程序时，Activity A就不会存在，因为他已经移动到浏览器程序的Task里去了。

### 清理Back Stack

如果用户离开一个task很久，系统就会清理这个task中除了根activity之外的所有activities。当用户返回到这个task，只有根activity会被恢复。但是我们可以设置一些activity的属性，用来改变这一行为：

*   alwaysRetainTaskState

如果这个属性在task的根activity中被设置为true，那么上面描述的默认行为不会发生，即便过了很长时间，task仍将会保持所有的activities。

*   clearTaskOnLaunch

如果这个属性在task的根activity中被设置为true，每次用户离开这个task，整个task都会被清到只剩根activity，这样用户只会永远返回到它最初的状态，即便离开的时间很短。

*   finishOnTaskLaunch

这个属性和上一个很像，不同的是，它只作用于单个activity，而不是整个task。它可以引起任何activity离开，包括根activity。当它被设置为true时，这个activity只在当前会话中属于这个task，如果用户离开后再返回，它也不会再出现。

### 开启一个Task

我们可以通过一个Activity指定一个intent过滤器，如下面：

```
<activity ...>
    <intent-filter ... >
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.inp tent.category.LAUNCHER" />
    </intent-filter>
    ...
</activity>

```
这时这个activity就作为根activity存在于一个task中，这个activity也是进入这个task的入口点，同时，它的图标和标签也会被显示在应用启动界面上，这时用户就可以启动这个activity并且再次回到这个任务。因此，从这里我们可以看到，要使用“singleTask”与“singleInstance”，就必须这个activity应当也有ACTION_MAIN与CATEGORY_LAUNCHER过滤器。因为假如没有设置这两个过滤器的话，当一个intent启动一个"singleTask"的activity,在新的Task中进行初使化，运行一段时间后，用户突然按上了home键回到桌面，此时这个Task就被移到后台并且不可见，因为这个activity没有设置过滤器，所以不是应用启动的activity，那么用户也就无法返回到这个Task中了。
