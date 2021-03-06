---
layout: post
author: mxn
titile: FragmentTransactions和ActivityStateLoss
category: 技术博文
tag: android
---

有的时候我们会遇到如下错误，这篇文章就会分析出现这种错误的原因和解决方法。

    {% highlight java  %}
java.lang.IllegalStateException: Can not perform this action after onSaveInstanceState
    at android.support.v4.app.FragmentManagerImpl.checkStateLoss(FragmentManager.java:1341)
    at android.support.v4.app.FragmentManagerImpl.enqueueAction(FragmentManager.java:1352)
    at android.support.v4.app.BackStackRecord.commitInternal(BackStackRecord.java:595)
    at android.support.v4.app.BackStackRecord.commit(BackStackRecord.java:574)
    {% endhighlight %}

### exception出现原因

抛出这个异常，是因为这activity的状态保存之后调用了commit FragmentTransaction，导致了一个 Activity state loss异常。
在分析这个问题之前，我们先看看onSaveInstanceState()这个方法被调用时会发生什么。我们需要知道Android applications在Android runtime
中并不能完全控制自己的生命周期。系统可以在任何时间结束一个app进程来释放内存。后台的activities可能在没有警告的情况下被杀死。
由于这些不稳定的行为是始终对用户隐藏的，所以framework给了每一个Activity一个机会，在销毁之前通过调用onSaveInstanceState()保存自己的状态。
当这些保存的状态。在以后进行重新装载的时候，用户需要感觉后台切换到前台是无缝的，而不管Activity有没有销毁。

当framework调用onSaveInstanceState()，需要传递一个Bundle对象，来保存Activity的状态，Activity在里面记录了dialogs，fragments
和views的状态。当方法return的时候，系统包装了Bundle对象，并且把这个对象通过Binder接口传递到系统服务进程，这那里它可以安全的保存。
当系统未来打算重建Activity时，系统重新把这个Bundle传递回application，用来使用它恢复Activity的旧的状态。

那么为什么会报出这个异常呢，这个错误阻止了onSaveInstanceState()被调用时Bundle保存Activity的内容。这意味着当在onSaveInstanceState()
之后调用FragmentTransaction#commit()时，transaction不会被保存，因为在一开始它没有作为Activity的状态被记录。从用户的角度来看，这个transaction
会丢失，导致意外的U状态丢失。为了保证用户体验，系统会不惜一切代价避免状态丢失，简单的做法是当发生这种情况，抛出IllegalStateException异常。

<!-- more -->

### exception出现的时间

如果你之前遇到过这种异常，你可能会注意到在不同版本的系统上，有一些细小的不同。例如你可能发现在旧的版本上，不太容易出现这种错误，
或者在使用support library时比使用framework classes是更容易出现这种错误。但是并不代表support library是有bug的。

这些差别是因为android 3.0（Honeycomb）版本对Activity生命周期做了一些改变造成的。在Honeycomb之前，activities在被暂停之前是不可被杀死的。
也就意味着onSaveInstanceState()在onPause()之前调用。从Honeycomb开始，只有activity stopped之后才可以被杀死，意味着onSaveInstanceState()
在onStop()之前被调用，而不是onPause()之前。下面的表总结了区别。

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img97.png)


因为Activity生命周期上的细微的改变，support library需要根据不同版本改变行为。例如，在Honeycomb和以后的设备上，每一次在onSaveInstanceState()
之后调用commit()都会抛出异常，来警告开发者发生了状态丢失。然后在Honeycomb之前的设备上，并不是每一次都会抛出异常，因为它们调用onSaveInstanceState()的
时机更早，所以更容易发生状态丢失。所以android团队被迫做了妥协，为了旧版本更好的内部运作，旧版本必须接受并忍受在onPause()和onStop()之间发生的意外
状态丢失。下表总结了support library在不同版本中的行为。

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img98.png)


### 如何避免这种异常

在知道了它的原理之后，避免状态丢失就变的很简单。下面是一些关于使用FragmentTransactions的建议。

1.在Activity生命周期的方法里要小心的使用commit transactions。绝大多数应用都在第一次onCreate()时或者响应用户输入时调用commit transactions，
这样不会引发任何问题。但是如果transactions一旦放在别的生命周期方法，例如onActivityResult()，onStart()，onResume()就会变的棘手。
例如你不应该在FragmentActivity#onResume()中调用commit transactions，因为有些情况会导致这个方法在activity的状态恢复之前调用。

如果你的项目中需要在除了oncreate()方法之外的生命周期方法中使用commit transaction，那么应该在FragmentActivity#onResumeFragments()或
Activity#onPostResume()中调用。这两个方法保证会在Activity恢复它的原始数据之后再调用，因此它们都能避免状态丢失的错误。

2.避免在异步回调方法中操作transaction。例如AsyncTask#onPostExecute()和LoaderManager
.LoaderCallbacks#onLoadFinished()。当它们调用时，无法知道当前Activity的生命周期的状态。考虑一下下面的情形：

* 一个activity执行了AsyncTask
* 用户按了Home键导致onSaveInstanceState()和onStop()被调用
* AsyncTask执行完毕，onPostExecute()被调用，但是它不知道Activity已经被停止。
* FragmentTransaction在onPostExecute()中被执行commit，导致异常抛出。

一般来说，对于以上情形，应该避免在在异步回调方法中commit transaction。Google的工程师也同意这种做法。因为在异步回调中使用
commit FragmentTransactions可能会导致ui上的改变，而产生不好的用户体验。如果你的应用需要在异步回调中处理transaction，并且没有简单的办法
保证在onSaveInstanceState()之后调用，那么必须借助commitAllowingStateLoss()方法。

3.使用commitAllowingStateLoss()作为最后的办法。commit()和commitAllowingStateLoss()
唯一的区别就是commitAllowingStateLoss()在发生状态丢失时不会抛出异常。一般情况下最好不要使用commitAllowingStateLoss()，因为可能出现状态丢失。
更好的解决办法当然是确保在activity状态保存之前调用commit()，这样才会有更好的用户体验。除非状态丢失不可避免，否则不应该使用commitAllowingStateLoss()。


### 一种优雅的解决方案

下面的方法中创建一个自定义类，包装了android.os.Handler，在activity的onPause()的时候缓存信息，然后在resume里面拿出来。
确保任何改变fragment状态的操作，例如commit，dismiss等，在handler中操作。从PauseHandler class中取得handler。
我们还需要在 onPause()中调用PauseHandler.pause()，在onResume()中调用PauseHandler.resume()。
把Handler handleMessage()用processMessage()替代。提供一个storeMessage()的实现，永远返回true。

代码如下：

    {% highlight java  %}
/**
 * Message Handler class that supports buffering up of messages when the
 * activity is paused i.e. in the background.
 */
public abstract class PauseHandler extends Handler {
    /**
     * Message Queue Buffer
     */
    final Vector<Message> messageQueueBuffer = new Vector<Message>();
    /**
     * Flag indicating the pause state
     */
    private boolean paused;
    /**
     * Resume the handler
     */
    final public void resume() {
        paused = false;
        while (messageQueueBuffer.size() > 0) {
            final Message msg = messageQueueBuffer.elementAt(0);
            messageQueueBuffer.removeElementAt(0);
            sendMessage(msg);
        }
    }
    /**
     * Pause the handler
     */
    final public void pause() {
        paused = true;
    }
    /**
     * Notification that the message is about to be stored as the activity is
     * paused. If not handled the message will be saved and replayed when the
     * activity resumes.
     *
     * @param message
     *            the message which optional can be handled
     * @return true if the message is to be stored
     */
    protected abstract boolean storeMessage(Message message);
    /**
     * Notification message to be processed. This will either be directly from
     * handleMessage or played back from a saved message when the activity was
     * paused.
     *
     * @param message
     *            the message to be handled
     */
    protected abstract void processMessage(Message message);
    /** {@inheritDoc} */
    @Override
    final public void handleMessage(Message msg) {
        if (paused) {
            if (storeMessage(msg)) {
                Message msgCopy = new Message();
                msgCopy.copyFrom(msg);
                messageQueueBuffer.add(msgCopy);
            }
        } else {
            processMessage(msg);
        }
    }
}
    {% endhighlight %}

下面是PausedHandler如何使用的范例。点击按钮会发送一个延时的消息给handler，然后handler在ui线程中收到消息，显示一个DialogFragment。
我们点击按钮来打开一个dialog，这是按下home键，如果不使用PausedHandler则会抛出IllegalStateException。


    {% highlight java  %}
public class FragmentTestActivity extends Activity {
    /**
     * Used for "what" parameter to handler messages
     */
    final static int MSG_WHAT = ('F' << 16) + ('T' << 8) + 'A';
    final static int MSG_SHOW_DIALOG = 1;
    int value = 1;
    final static class State extends Fragment {
        static final String TAG = "State";
        /**
         * Handler for this activity
         */
        public ConcreteTestHandler handler = new ConcreteTestHandler();
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setRetainInstance(true);
        }
        @Override
        public void onResume() {
            super.onResume();
            handler.setActivity(getActivity());
            handler.resume();
        }
        @Override
        public void onPause() {
            super.onPause();
            handler.pause();
        }
        public void onDestroy() {
            super.onDestroy();
            handler.setActivity(null);
        }
    }
    /**
     * 2 second delay
     */
    final static int DELAY = 2000;
    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        if (savedInstanceState == null) {
            final Fragment state = new State();
            final FragmentManager fm = getFragmentManager();
            final FragmentTransaction ft = fm.beginTransaction();
            ft.add(state, State.TAG);
            ft.commit();
        }
        final Button button = (Button) findViewById(R.id.popup);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                final FragmentManager fm = getFragmentManager();
                State fragment = (State) fm.findFragmentByTag(State.TAG);
                if (fragment != null) {
                    // Send a message with a delay onto the message looper
                    fragment.handler.sendMessageDelayed(
                            fragment.handler.obtainMessage(MSG_WHAT, MSG_SHOW_DIALOG, value++),
                            DELAY);
                }
            }
        });
    }
    public void onSaveInstanceState(Bundle bundle) {
        super.onSaveInstanceState(bundle);
    }
    /**
     * Simple test dialog fragment
     */
    public static class TestDialog extends DialogFragment {
        int value;
        /**
         * Fragment Tag
         */
        final static String TAG = "TestDialog";
        public TestDialog() {
        }
        public TestDialog(int value) {
            this.value = value;
        }
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
        }
        @Override
        public View onCreateView(LayoutInflater inflater, ViewGroup container,
                Bundle savedInstanceState) {
            final View inflatedView = inflater.inflate(R.layout.dialog, container, false);
            TextView text = (TextView) inflatedView.findViewById(R.id.count);
            text.setText(getString(R.string.count, value));
            return inflatedView;
        }
    }
    /**
     * Message Handler class that supports buffering up of messages when the
     * activity is paused i.e. in the background.
     */
    static class ConcreteTestHandler extends PauseHandler {
        /**
         * Activity instance
         */
        protected Activity activity;
        /**
         * Set the activity associated with the handler
         *
         * @param activity
         *            the activity to set
         */
        final void setActivity(Activity activity) {
            this.activity = activity;
        }
        @Override
        final protected boolean storeMessage(Message message) {
            // All messages are stored by default
            return true;
        };
        @Override
        final protected void processMessage(Message msg) {
            final Activity activity = this.activity;
            if (activity != null) {
                switch (msg.what) {
                case MSG_WHAT:
                    switch (msg.arg1) {
                    case MSG_SHOW_DIALOG:
                        final FragmentManager fm = activity.getFragmentManager();
                        final TestDialog dialog = new TestDialog(msg.arg2);
                        // We are on the UI thread so display the dialog
                        // fragment
                        dialog.show(fm, TestDialog.TAG);
                        break;
                    }
                    break;
                }
            }
        }
    }
}
    {% endhighlight %}

这里在PausedHandler中增加了storeMessage()方法，这样即使activity停止了也可以立即处理信息，如果这个方法返回了fasle，那么
信息就被丢弃掉。

