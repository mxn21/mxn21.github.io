---
layout: post
author: mxn
titile: activity LaunchMode 应用场景
category: 技术博文
tag: android
---

### 理解task

在看LaunchMode之前先理解一下Task。task是一个stack（栈），"Last in, First out"。一个task包含了一组activity实例。
一般情况下，当用户打开一个新的app的时候一个新的task就被创建了，第一个打开的activity被叫做task的root。安卓系统可以同时运行很多个
task，但是只有一个task在前台。

通过LaunchMode可以设置不同的activity关联到task的方式。

Activity的启动模式一种有四种，分别如下：

1、standard

2、singleTop

3、singleTask

4、singleInstance

这四种类型可以分为两大类：
'standard' 和 'singleTop'可以在同一个栈中加载很多activity。区别于下面一组在于，这两个模式不会移除其他Activity。

'singleTask' 和 'singleInstance'的Activity会使用单例模式，并且这个Activity会是一个新的栈的root。

launch mode可以在 AndroidManifest文件中配置，也可以使用Intent flags，比如FLAG_ACTIVITY_NEW_TASK,
FLAG_ACTIVITY_CLEAR_TOP 和 FLAG_ACTIVITY_SINGLE_TOP 。

其实还有一种情况比较特殊，不过很少使用，就是在一个app之间，我们给某一个Activity配置了taskAffinity属性，这个属性会影响singleTask等属性，这个可以大家自己去分析。


standard：不论当前任务栈中是否存在该Activity，都会新建一个Activity，如 任务栈为A->B，要启动B 那么任务栈为 A->B->B

singleTop：如果当前要创建的Activity就在任务栈的顶端，那么不会创建新的Activity，仅仅调用Activity的onNewIntent，
如果不在栈顶(或者栈中没有该Activity)，那么还是会创建新的Activity，如任务栈为A->B。启动B,任务栈变为 A->B。如果启动A,
那么任务栈为 A->B->A。

singleTask：如果当前任务中存在要启动的Activity，那么就不会创建新的Activity，如果不存在就会创建新的Activity，如任务栈为 A->B->C，启动B
，那么任务栈就会变为A->B。如任务栈为 A->B->C，启动D，那么任务栈就会变为D 。

singleInstance：将一个Activity的launchMode设置为该值时，表明这个Activity独自占用一个任务队列，这个队列中不让在加入其他的Activity。


<!-- more -->

### 应用场景

#### SingleTop

应用场景：网易新闻。
假设主界面为 MainActivity，显示新闻的界面是 DetailActivity，显然显示任何一条新闻都会使用 DetailActivity，即把新闻内容通过 Intent 传给 DetailActivity 就可以了。
假设你正在看新闻1(即在 DetailActivity)，此时手机收到服务器的推送：收到一条通知(新闻2)，点击通知就会跳转到 DetailActivity 并显示新闻2，当你点击通知时，因为目前栈顶的 Activity 就是 DetailActivity，因此这里就是使用 SingleTop 的地方，即点击通知后以 SingleTop 加载模式打开 DetailActivity 并显示新闻2，因此新闻1的 DetailActivity 就被覆盖掉了。
此后你点击返回键会回到主界面。

#### SingleTask

应用场景：微信的主界面(一般应用主界面都会以 SingleTask 启动)。
你打开微信主界面(在栈的最底部)后，进入朋友圈(在栈的顶部)，此时你点击 Home 键回桌面，并打开网易新闻。
假设你想将网易新闻的一条新闻分享给微信好友，那么就按照 分享->微信->好友A->分享给他->留在微信。接着会跳转微信的主界面，即不是你原本所在的朋友圈，并且微信的栈只剩下一个元素：主界面的 Activity。这里就使用了 SingleTask(即以 SingleTask 加载模式打开微信主界面)。

#### SingleInstance

应用场景：闹铃的响铃界面。
你以前设置了一个闹铃：上午6点。在上午5点58分，你启动了闹铃设置界面，并按 Home 键回桌面；在上午5点59分时，你在微信和朋友聊天；
在6点时，闹铃响了，并且弹出了一个对话框形式的 Activity(名为 AlarmAlertActivity) 提示你到6点了(这个 Activity 就是以
SingleInstance 加载模式打开的)，你按返回键，回到的是微信的聊天界面，这是因为 AlarmAlertActivity 所在的 Task 的栈只有他一个元素，
因此退出之后这个 Task 的栈空了。如果是以 SingleTask 打开 AlarmAlertActivity，那么当闹铃响了的时候，按返回键应该进入闹铃设置界面。


### 源码分析
下面我们就来看看和launchMode处理有关的代码吧

对launchMode处理的逻辑主要是放在了ActivityStack的startActivityUncheckedLocked方法中，这份方法的逻辑有些复杂，
我们来一部分一部分的分析：

第一部分：

    {% highlight java  %}
    if (sourceRecord == null) {
               // This activity is not being started from another...  in this
               // case we -always- start a new task.
               if ((launchFlags&Intent.FLAG_ACTIVITY_NEW_TASK) == 0) {
                   Slog.w(TAG, "startActivity called from non-Activity context; forcing Intent.FLAG_ACTIVITY_NEW_TASK for: "
                         + intent);
                   launchFlags |= Intent.FLAG_ACTIVITY_NEW_TASK;
               }
           } else if (sourceRecord.launchMode == ActivityInfo.LAUNCH_SINGLE_INSTANCE) {
               // The original activity who is starting us is running as a single
               // instance...  this new activity it is starting must go on its
               // own task.
               launchFlags |= Intent.FLAG_ACTIVITY_NEW_TASK;
           } else if (r.launchMode == ActivityInfo.LAUNCH_SINGLE_INSTANCE
                   || r.launchMode == ActivityInfo.LAUNCH_SINGLE_TASK) {
               // The activity being started is a single instance...  it always
               // gets launched into its own task.
               launchFlags |= Intent.FLAG_ACTIVITY_NEW_TASK;
           }
	{% endhighlight %}

首先判断sourceRecord是否为Null(桌面启动一个Activity或者通过Context启动一个Activity时sourceRecord为null)，如果在启动的Intent中没有FLAG_ACTIVITY_NEW_TASK那么就会在该Intent中添加该标记。

如果sourceRecord的launchMode设置的是singleinstance，那么就会在Intent添加FLAG_ACTIVITY_NEW_TASK，因为对于singleinstance的Activity，是不会和别人共享一个队列的。

如果被启动的Activity的launchMode是singleinstance或者singletask，那么也会在Intent中添加FLAG_ACTIVITY_NEW_TASK标记(上面我们已经说过)。


在看第二部分代码:

    {% highlight java  %}
    if (r.resultTo != null && (launchFlags&Intent.FLAG_ACTIVITY_NEW_TASK) != 0) {
        // For whatever reason this activity is being launched into a new
        // task...  yet the caller has requested a result back.  Well, that
        // is pretty messed up, so instead immediately send back a cancel
        // and let the new task continue launched as normal without a
        // dependency on its originator.
        Slog.w(TAG, "Activity is launching as a new task, so cancelling activity result.");
        sendActivityResultLocked(-1,
                r.resultTo, r.resultWho, r.requestCode,
            Activity.RESULT_CANCELED, null);
        r.resultTo = null;
    }
    {% endhighlight %}

通过这段代码可以当Intent中包含了Intent.FLAG_ACTIVITY_NEW_TASK标记时，是不能使用startActivityforResult方法启动Activity的，也就是说如果一个Activity是sinleTask或者singleInstance时，都不能通过startActivityForResult方法调起(可以调起，
但是无法传回值)

第三部分代码:

    {% highlight java  %}

    boolean addingToTask = false;
            boolean movedHome = false;
            TaskRecord reuseTask = null;
            if (((launchFlags&Intent.FLAG_ACTIVITY_NEW_TASK) != 0 &&
                    (launchFlags&Intent.FLAG_ACTIVITY_MULTIPLE_TASK) == 0)
                    || r.launchMode == ActivityInfo.LAUNCH_SINGLE_TASK
                    || r.launchMode == ActivityInfo.LAUNCH_SINGLE_INSTANCE) {
                // If bring to front is requested, and no result is requested, and
                // we can find a task that was started with this same
                // component, then instead of launching bring that one to the front.
                if (r.resultTo == null) {
                    // See if there is a task to bring to the front.  If this is
                    // a SINGLE_INSTANCE activity, there can be one and only one
                    // instance of it in the history, and it is always in its own
                    // unique task, so we do a special search.
                    ActivityRecord taskTop = r.launchMode != ActivityInfo.LAUNCH_SINGLE_INSTANCE
                            ? findTaskLocked(intent, r.info)
                            : findActivityLocked(intent, r.info);
                    if (taskTop != null) {
                        if (taskTop.task.intent == null) {
                            // This task was started because of movement of
                            // the activity based on affinity...  now that we
                            // are actually launching it, we can assign the
                            // base intent.
                            taskTop.task.setIntent(intent, r.info);
                        }
                        // If the target task is not in the front, then we need
                        // to bring it to the front...  except...  well, with
                        // SINGLE_TASK_LAUNCH it's not entirely clear.  We'd like
                        // to have the same behavior as if a new instance was
                        // being started, which means not bringing it to the front
                        // if the caller is not itself in the front.
                        ActivityRecord curTop = topRunningNonDelayedActivityLocked(notTop);
                        if (curTop != null && curTop.task != taskTop.task) {
                            r.intent.addFlags(Intent.FLAG_ACTIVITY_BROUGHT_TO_FRONT);
                            boolean callerAtFront = sourceRecord == null
                                    || curTop.task == sourceRecord.task;
                            if (callerAtFront) {
                                // We really do want to push this one into the
                                // user's face, right now.
                                movedHome = true;
                                moveHomeToFrontFromLaunchLocked(launchFlags);
                                moveTaskToFrontLocked(taskTop.task, r, options);
                                options = null;
                            }
                        }
                        // If the caller has requested that the target task be
                        // reset, then do so.
                        if ((launchFlags&Intent.FLAG_ACTIVITY_RESET_TASK_IF_NEEDED) != 0) {
                            taskTop = resetTaskIfNeededLocked(taskTop, r);
                        }
                        if ((startFlags&ActivityManager.START_FLAG_ONLY_IF_NEEDED)  != 0) {
                            // We don't need to start a new activity, and
                            // the client said not to do anything if that
                            // is the case, so this is it!  And for paranoia, make
                            // sure we have correctly resumed the top activity.
                            if (doResume) {
                                resumeTopActivityLocked(null, options);
                            } else {
                                ActivityOptions.abort(options);
                            }
                            return ActivityManager.START_RETURN_INTENT_TO_CALLER;
                        }
                        if ((launchFlags &
                                (Intent.FLAG_ACTIVITY_NEW_TASK|Intent.FLAG_ACTIVITY_CLEAR_TASK))
                                == (Intent.FLAG_ACTIVITY_NEW_TASK|Intent.FLAG_ACTIVITY_CLEAR_TASK)) {
                            // The caller has requested to completely replace any
                            // existing task with its new activity.  Well that should
                            // not be too hard...
                            reuseTask = taskTop.task;
                            performClearTaskLocked(taskTop.task.taskId);
                            reuseTask.setIntent(r.intent, r.info);
                        } else if ((launchFlags&Intent.FLAG_ACTIVITY_CLEAR_TOP) != 0
                                || r.launchMode == ActivityInfo.LAUNCH_SINGLE_TASK
                                || r.launchMode == ActivityInfo.LAUNCH_SINGLE_INSTANCE) {
                            // In this situation we want to remove all activities
                            // from the task up to the one being started.  In most
                            // cases this means we are resetting the task to its
                            // initial state.
                            ActivityRecord top = performClearTaskLocked(
                                    taskTop.task.taskId, r, launchFlags);
                            if (top != null) {
                                if (top.frontOfTask) {
                                    // Activity aliases may mean we use different
                                    // intents for the top activity, so make sure
                                    // the task now has the identity of the new
                                    // intent.
                                    top.task.setIntent(r.intent, r.info);
                                }
                                logStartActivity(EventLogTags.AM_NEW_INTENT, r, top.task);
                                top.deliverNewIntentLocked(callingUid, r.intent);
                            } else {
                                // A special case: we need to
                                // start the activity because it is not currently
                                // running, and the caller has asked to clear the
                                // current task to have this activity at the top.
                                addingToTask = true;
                                // Now pretend like this activity is being started
                                // by the top of its task, so it is put in the
                                // right place.
                                sourceRecord = taskTop;
                            }
                        } else if (r.realActivity.equals(taskTop.task.realActivity)) {
                            // In this case the top activity on the task is the
                            // same as the one being launched, so we take that
                            // as a request to bring the task to the foreground.
                            // If the top activity in the task is the root
                            // activity, deliver this new intent to it if it
                            // desires.
                            if ((launchFlags&Intent.FLAG_ACTIVITY_SINGLE_TOP) != 0
                                    && taskTop.realActivity.equals(r.realActivity)) {
                                logStartActivity(EventLogTags.AM_NEW_INTENT, r, taskTop.task);
                                if (taskTop.frontOfTask) {
                                    taskTop.task.setIntent(r.intent, r.info);
                                }
                                taskTop.deliverNewIntentLocked(callingUid, r.intent);
                            } else if (!r.intent.filterEquals(taskTop.task.intent)) {
                                // In this case we are launching the root activity
                                // of the task, but with a different intent.  We
                                // should start a new instance on top.
                                addingToTask = true;
                                sourceRecord = taskTop;
                            }
                        } else if ((launchFlags&Intent.FLAG_ACTIVITY_RESET_TASK_IF_NEEDED) == 0) {
                            // In this case an activity is being launched in to an
                            // existing task, without resetting that task.  This
                            // is typically the situation of launching an activity
                            // from a notification or shortcut.  We want to place
                            // the new activity on top of the current task.
                            addingToTask = true;
                            sourceRecord = taskTop;
                        } else if (!taskTop.task.rootWasReset) {
                            // In this case we are launching in to an existing task
                            // that has not yet been started from its front door.
                            // The current task has been brought to the front.
                            // Ideally, we'd probably like to place this new task
                            // at the bottom of its stack, but that's a little hard
                            // to do with the current organization of the code so
                            // for now we'll just drop it.
                            taskTop.task.setIntent(r.intent, r.info);
                        }
                        if (!addingToTask && reuseTask == null) {
                            // We didn't do anything...  but it was needed (a.k.a., client
                            // don't use that intent!)  And for paranoia, make
                            // sure we have correctly resumed the top activity.
                            if (doResume) {
                                resumeTopActivityLocked(null, options);
                            } else {
                                ActivityOptions.abort(options);
                            }
                            return ActivityManager.START_TASK_TO_FRONT;
                        }
                    }
                }
            }


    {% endhighlight %}


这段代码的逻辑有些复杂，但是功能很简单，就是为要启动的Activity寻找或者创建一个任务栈，现在我使用singleTask为例，跟踪一下上述代码流程。

假设后台任务栈为 A B C，前台任务中要启动一个Activity B，我们就使用这个需求来跟踪上面的代码逻辑。

首先注意默认的addingToTask是false，reuseTask为null，由于是singleTask的，所以((launchFlags&Intent.FLAG_ACTIVITY_NEW_TASK) != 0 && (launchFlags&Intent.FLAG_ACTIVITY_MULTIPLE_TASK) == 0)是成立的，即使这里不成立，后面的逻辑也是成立，所以必然进入该if语句

继续跟进代码，会发现调用了findTaskLocked方法返回一个ActivityRecord，你可以查看findTaskLocked的逻辑，你会知道这个ActiivtyRecord 就是对应的是Activity C，继续往下走，就会调用 moveTaskToFrontLocked将后台任务移动到前台，由于我们考虑的是launchMode是singleTask，所以将会进入


       {% highlight java  %}
        else if ((launchFlags&Intent.FLAG_ACTIVITY_CLEAR_TOP) != 0
                            || r.launchMode == ActivityInfo.LAUNCH_SINGLE_TASK
                            || r.launchMode == ActivityInfo.LAUNCH_SINGLE_INSTANCE)

        {% endhighlight %}

进入上面的if语句之后，就会调用performClearTaskLocked返回一个ActivityRecord，你可以去研究一下performClearTaskLocked的逻辑，这个方法传入了一个ActivityRecord r，就是将任务栈中r之上的AcrivityRecord 清除掉，并返回ActivityRecord r，这里返回的ActivityRecord 对应的就是Activity B。接着调用Activity B的onNewIntent方法。执行了这个else if，其他else if是都不会执行的，所以就直接执行到了

       {% highlight java  %}
       if (!addingToTask && reuseTask == null) {
                          // We didn't do anything...  but it was needed (a.k.a., client
                          // don't use that intent!)  And for paranoia, make
                          // sure we have correctly resumed the top activity.
                          if (doResume) {
                              resumeTopActivityLocked(null, options);
                          } else {
                              ActivityOptions.abort(options);
                          }
                          return ActivityManager.START_TASK_TO_FRONT;
                      }
       {% endhighlight %}

这里直接调用了resumeTopActivityLocked方法。到这里singleTask就分析完了，其他的启动模式大家可以按照这种思路进行分析。


第四部分代码：

       {% highlight java  %}
       if (r.packageName != null) {
                   // If the activity being launched is the same as the one currently
                   // at the top, then we need to check if it should only be launched
                   // once.
                   ActivityRecord top = topRunningNonDelayedActivityLocked(notTop);
                   if (top != null && r.resultTo == null) {
                       if (top.realActivity.equals(r.realActivity) && top.userId == r.userId) {
                           if (top.app != null && top.app.thread != null) {
                               if ((launchFlags&Intent.FLAG_ACTIVITY_SINGLE_TOP) != 0
                                   || r.launchMode == ActivityInfo.LAUNCH_SINGLE_TOP
                                   || r.launchMode == ActivityInfo.LAUNCH_SINGLE_TASK) {
                                   logStartActivity(EventLogTags.AM_NEW_INTENT, top, top.task);
                                   // For paranoia, make sure we have correctly
                                   // resumed the top activity.
                                   if (doResume) {
                                       resumeTopActivityLocked(null);
                                   }
                                   ActivityOptions.abort(options);
                                   if ((startFlags&ActivityManager.START_FLAG_ONLY_IF_NEEDED) != 0) {
                                       // We don't need to start a new activity, and
                                       // the client said not to do anything if that
                                       // is the case, so this is it!
                                       return ActivityManager.START_RETURN_INTENT_TO_CALLER;
                                   }
                                   top.deliverNewIntentLocked(callingUid, r.intent);
                                   return ActivityManager.START_DELIVERED_TO_TOP;
                               }
                           }
                       }
                   }

               } else {
                   if (r.resultTo != null) {
                       sendActivityResultLocked(-1,
                               r.resultTo, r.resultWho, r.requestCode,
                           Activity.RESULT_CANCELED, null);
                   }
                   ActivityOptions.abort(options);
                   return ActivityManager.START_CLASS_NOT_FOUND;
               }

               boolean newTask = false;
               boolean keepCurTransition = false;

               // Should this be considered a new task?
               if (r.resultTo == null && !addingToTask
                       && (launchFlags&Intent.FLAG_ACTIVITY_NEW_TASK) != 0) {
                   if (reuseTask == null) {
                       // todo: should do better management of integers.
                       mService.mCurTask++;
                       if (mService.mCurTask <= 0) {
                           mService.mCurTask = 1;
                       }
                       r.setTask(new TaskRecord(mService.mCurTask, r.info, intent), null, true);
                       if (DEBUG_TASKS) Slog.v(TAG, "Starting new activity " + r
                               + " in new task " + r.task);
                   } else {
                       r.setTask(reuseTask, reuseTask, true);
                   }
                   newTask = true;
                   if (!movedHome) {
                       moveHomeToFrontFromLaunchLocked(launchFlags);
                   }

               } else if (sourceRecord != null) {
                   if (!addingToTask &&
                           (launchFlags&Intent.FLAG_ACTIVITY_CLEAR_TOP) != 0) {
                       // In this case, we are adding the activity to an existing
                       // task, but the caller has asked to clear that task if the
                       // activity is already running.
                       ActivityRecord top = performClearTaskLocked(
                               sourceRecord.task.taskId, r, launchFlags);
                       keepCurTransition = true;
                       if (top != null) {
                           logStartActivity(EventLogTags.AM_NEW_INTENT, r, top.task);
                           top.deliverNewIntentLocked(callingUid, r.intent);
                           // For paranoia, make sure we have correctly
                           // resumed the top activity.
                           if (doResume) {
                               resumeTopActivityLocked(null);
                           }
                           ActivityOptions.abort(options);
                           return ActivityManager.START_DELIVERED_TO_TOP;
                       }
                   } else if (!addingToTask &&
                           (launchFlags&Intent.FLAG_ACTIVITY_REORDER_TO_FRONT) != 0) {
                       // In this case, we are launching an activity in our own task
                       // that may already be running somewhere in the history, and
                       // we want to shuffle it to the front of the stack if so.
                       int where = findActivityInHistoryLocked(r, sourceRecord.task.taskId);
                       if (where >= 0) {
                           ActivityRecord top = moveActivityToFrontLocked(where);
                           logStartActivity(EventLogTags.AM_NEW_INTENT, r, top.task);
                           top.updateOptionsLocked(options);
                           top.deliverNewIntentLocked(callingUid, r.intent);
                           if (doResume) {
                               resumeTopActivityLocked(null);
                           }
                           return ActivityManager.START_DELIVERED_TO_TOP;
                       }
                   }
                   // An existing activity is starting this new activity, so we want
                   // to keep the new one in the same task as the one that is starting
                   // it.
                   r.setTask(sourceRecord.task, sourceRecord.thumbHolder, false);
                   if (DEBUG_TASKS) Slog.v(TAG, "Starting new activity " + r
                           + " in existing task " + r.task);

               } else {
                   // This not being started from an existing activity, and not part
                   // of a new task...  just put it in the top task, though these days
                   // this case should never happen.
                   final int N = mHistory.size();
                   ActivityRecord prev =
                       N > 0 ? mHistory.get(N-1) : null;
                   r.setTask(prev != null
                           ? prev.task
                           : new TaskRecord(mService.mCurTask, r.info, intent), null, true);
                   if (DEBUG_TASKS) Slog.v(TAG, "Starting new activity " + r
                           + " in new guessed " + r.task);
               }

               mService.grantUriPermissionFromIntentLocked(callingUid, r.packageName,
                       intent, r.getUriPermissionsLocked());

               if (newTask) {
                   EventLog.writeEvent(EventLogTags.AM_CREATE_TASK, r.task.taskId);
               }
               logStartActivity(EventLogTags.AM_CREATE_ACTIVITY, r, r.task);
               startActivityLocked(r, newTask, doResume, keepCurTransition, options);
               return ActivityManager.START_SUCCESS;
       {% endhighlight %}

我刚刚使用singleTask模式分析时，并没有走到这里，大家可以另一种情况来分析，就可以走到这里。

后台任务栈 A B，前台任务栈 E F，在前台任务栈中要启动后台任务的一个Activity C，也就是说后台任务栈中不存在C的情况，或者在一个app内通过配置taskAffinity属性也可以走到这里。