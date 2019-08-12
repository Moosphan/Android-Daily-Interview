## 第 02 期

> [**Activity 与 Fragment 之间常见的几种通信方式？**](https://github.com/Moosphan/Android-Daily-Interview/issues/2)

开始介绍方法之前，先借用之前 lx36301766 写在答案里的一句话，我个人觉得写的很到位：
> 说到底就是两个普通的 Java 对象相互都持有对方的引用，直接回调就成，哪还需要什么别的通信方式，很简单的问题别搞复杂了。

大家都知道java是面向对象的编程语言。只要拿到对象，想做什么操作其实很简单。好开始正文。

从两个方面开始说：
* Activity 与 Fragment 通信
* Fragment 与 Activity 通信

### Activity与Fragment通信
#### 1. 广播

这个就不写，相信大家都会。

#### 2. 对象调用 public 方法


```java
public class TestActivity extends AppCompatActivity{
    
    private TestFragmet testFragment;
    
    public void test(){
        testFragment = new TestFragmet();
        testFragment.doTest();
    }
}
```

```java
public class TestFragmet extends Fragment{
    private TestActivity activity;
    
    public void doTest(){
        Log.d(TAG,"收到消息");
    }
}
```
#### 3. 接口
```java
    public class TestActivity extends Activity{

        private ITest iTest;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            iTest = new TestFragment();
            iTest.doTest();
        }
    }
```
```java

    public class TestFragment extends Fragment implements ITest{

        @Override
        public void doTest() {
            Log.i(TAG, "doTest: ");
        }
    }

    public interface ITest{
        void doTest();
    }
```
#### 4. 观察者
这里模拟一个场景，用户在 Activity 里登录状态发生了改变，Fragment 展示退出登录的 UI。我真实项目里也是采用的观察者模式管理登录状态,因为登录状态的改变牵连到一系列的改变。

```java
    public class TestActivity extends Activity {

        final int LOGIN_OUT = 0;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            LoginManager.loginStatusChanged(LOGIN_OUT);
        }
    }
```
```java


    public class TestFragment extends Fragment implements LoginObserver {
        @Override
        public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {

            /**
             * 订阅事件
             */
            LoginManager.registerLoginObserver(this);
            return super.onCreateView(inflater, container, savedInstanceState);
        }

        @Override
        public void onDestroy() {
            super.onDestroy();
            /**
             * 反订阅事件
             */
            LoginManager.unregisterLoginObserver(this);
        }

        @Override
        public void loginStatusChange(int status) {
            Log.i(TAG, "loginStatusChange: "+status);
        }
    }

```
```java

    public class LoginManager {

        private static ArrayList<LoginObserver> loginObserverList = new ArrayList<>();

        public static void registerLoginObserver(LoginObserver loginObserver){
            loginObserverList.add(loginObserver);
        }

        public static void unregisterLoginObserver(LoginObserver loginObserver){
            loginObserverList.remove(loginObserver);
        }

        public static void loginStatusChanged(int loginstatus){
            for (LoginObserver observer:loginObserverList){
                if (observer!=null){
                    observer.loginStatusChange(loginstatus);
                }
            }
        }
    }

```
```java
    public interface LoginObserver{

        void loginStatusChange(int status);
    }
```
#### 5. EventBus
也没什么好说的，相信大家都用过。
### Fragment与Activity通信

#### 1. 广播
同上
#### 2. 对象调用 public 方法
```java
    public class TestActivity extends Activity {
        
        public void doTest(){
            Log.i(TAG, "doTest: ");
        }
    }
```
```java
    public class TestFragment extends Fragment {

        private TestActivity activity;
        @Override
        public void onAttach(Context context) {
            super.onAttach(context);
            if (getActivity() instanceof TestActivity){
                activity = getActivity();
            }
        }

        @Override
        public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
            return super.onCreateView(inflater, container, savedInstanceState);
        }

        void test(){
            activity.doTest();
        }
    }
```
#### 3. 接口
```java
    public class TestActivity extends Activity implements ITest{
        
        @Override
        public void doTest() {
            Log.i(TAG, "doTest: ");
        }
    }
```
```java
    public class TestFragment extends Fragment {

        private ITest iTest;
        @Override
        public void onAttach(Context context) {
            super.onAttach(context);
            if (context instanceof ITest){
                iTest = (ITest) context;
            }
        }

        @Override
        public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
            return super.onCreateView(inflater, container, savedInstanceState);
        }

        void test(){
            iTest.doTest();
        }
    }

```
```java

    public interface ITest{
        void doTest();
    }
```
#### 4. 观察者
同1.4，只不过这次是 Fragment 改变用户登录状态，Activiy 观察事件而已。
```java
    public class TestActivity extends Activity implements LoginObserver {



        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            LoginManager.registerLoginObserver(this);
        }

        @Override
        protected void onDestroy() {
            super.onDestroy();
            LoginManager.unregisterLoginObserver(this);
        }

        @Override
        public void loginStatusChange(int status) {

        }
    }
```
```java
    public class TestFragment extends Fragment {

        final int LOGIN_OUT = 0;

        @Override
        public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
            return super.onCreateView(inflater, container, savedInstanceState);
        }


        void doTest(){
            LoginManager.loginStatusChanged(LOGIN_OUT);
        }
    }
```
```java
    public class LoginManager {

        private static ArrayList<LoginObserver> loginObserverList = new ArrayList<>();

        public static void registerLoginObserver(LoginObserver loginObserver){
            loginObserverList.add(loginObserver);
        }

        public static void unregisterLoginObserver(LoginObserver loginObserver){
            loginObserverList.remove(loginObserver);
        }

        public static void loginStatusChanged(int loginstatus){
            for (LoginObserver observer:loginObserverList){
                if (observer!=null){
                    observer.loginStatusChange(loginstatus);
                }
            }
        }
    }
```
```java
    public interface LoginObserver{

        void loginStatusChange(int status);
    }

```
#### 5. EventBus
自行百度吧，这个很简单。


### 题外话
大家可能发现接口和对象调用 public 方法出奇的相似，那么为什么要多次一举写个接口呢？大家可以点击一下TestFragment#test() 方法里的 `iTest.doTest();` 会发现跳转到了 ITest 接口里，而点击 TestFragment#test() 里的 activity.doTest(); 却是跳转到了TestActivity里。到这里相信大家就看出接口的一个妙用，隐藏实现，当然不是完全隐藏，你可以通过搜索哪里实现继续查看。当然接口还有其他功能，就不赘述了。


很少写博客，肯定有解释不清楚的地方，欢迎大家提宝贵的意见给我，也可提交合并请求。
