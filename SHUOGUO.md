# X2C学习分享【硕果】

## 学习思路
   
   *  X2C产生的背景？
   *  X2C的原理？
   *  X2C的具体实现？
   *  X2C带来了什么？
   *  X2C解决问题的能力边界及优缺点？
   *  我们可以从X2C中学习到什么？
   
### X2C产生的背景  

   [README](README_CN.md)
 
### X2C的原理
 
   ```
 TODO：通过阅读相关Blog以及Android源码，对技术原理进行深入学习
   ```

   [Android LayoutInflater原理分析，带你一步步深入了解View(一) --- by guolin](https://blog.csdn.net/guolin_blog/article/details/12921889)

   *  加载布局的任务通常都是在Activity中调用**setContentView()**函数来完成的。其实setContentView()方法的内部也是使用**LayoutInflater**来加载布局的。
      ```JAVA
      ```
   *  **LayoutInflater**其实就是使用Android提供的pull解析方式来解析布局XML文件的。
   
      ```JAVA
      ```
   *  **LayoutInflater**调用的**inflate**函数中在**createViewFromTag()**函数的内部又会去调用**createView()**函数，然后使用反射的方式创建出View的实例并返回。
   
      ```JAVA
      /**
      * Low-level function for instantiating a view by name. This attempts to
      * instantiate a view class of the given <var>name</var> found in this
      * LayoutInflater's ClassLoader.
      *
      * <p>
      * There are two things that can happen in an error case: either the
      * exception describing the error will be thrown, or a null will be
      * returned. You must deal with both possibilities -- the former will happen
      * the first time createView() is called for a class of a particular name,
      * the latter every time there-after for that class name.
      *
      * @param name The full name of the class to be instantiated.
      * @param attrs The XML attributes supplied for this instance.
      *
      * @return View The newly instantiated view, or null.
      */
      public final View createView(String name, String prefix, AttributeSet attrs)
              throws ClassNotFoundException, InflateException {
          Constructor<? extends View> constructor = sConstructorMap.get(name);
          if (constructor != null && !verifyClassLoader(constructor)) {
              constructor = null;
              sConstructorMap.remove(name);
          }
          Class<? extends View> clazz = null;
   
          try {
              Trace.traceBegin(Trace.TRACE_TAG_VIEW, name);
   
              if (constructor == null) {
                  // Class not found in the cache, see if it's real, and try to add it
                  clazz = mContext.getClassLoader().loadClass(
                          prefix != null ? (prefix + name) : name).asSubclass(View.class);
   
                  if (mFilter != null && clazz != null) {
                      boolean allowed = mFilter.onLoadClass(clazz);
                      if (!allowed) {
                          failNotAllowed(name, prefix, attrs);
                      }
                  }
                  constructor = clazz.getConstructor(mConstructorSignature);
                  constructor.setAccessible(true);
                  sConstructorMap.put(name, constructor);
              } else {
                  // If we have a filter, apply it to cached constructor
                  if (mFilter != null) {
                      // Have we seen this name before?
                      Boolean allowedState = mFilterMap.get(name);
                      if (allowedState == null) {
                          // New class -- remember whether it is allowed
                          clazz = mContext.getClassLoader().loadClass(
                                  prefix != null ? (prefix + name) : name).asSubclass(View.class);
   
                          boolean allowed = clazz != null && mFilter.onLoadClass(clazz);
                          mFilterMap.put(name, allowed);
                          if (!allowed) {
                              failNotAllowed(name, prefix, attrs);
                          }
                      } else if (allowedState.equals(Boolean.FALSE)) {
                          failNotAllowed(name, prefix, attrs);
                      }
                  }
              }
   
              Object lastContext = mConstructorArgs[0];
              if (mConstructorArgs[0] == null) {
                  // Fill in the context if not already within inflation.
                  mConstructorArgs[0] = mContext;
              }
              Object[] args = mConstructorArgs;
              args[1] = attrs;
   
              final View view = constructor.newInstance(args);
              if (view instanceof ViewStub) {
                  // Use the same context when inflating ViewStub later.
                  final ViewStub viewStub = (ViewStub) view;
                  viewStub.setLayoutInflater(cloneInContext((Context) args[0]));
              }
              mConstructorArgs[0] = lastContext;
              return view;
   
          } catch (NoSuchMethodException e) {
              final InflateException ie = new InflateException(attrs.getPositionDescription()
                      + ": Error inflating class " + (prefix != null ? (prefix + name) : name), e);
              ie.setStackTrace(EMPTY_STACK_TRACE);
              throw ie;
   
          } catch (ClassCastException e) {
              // If loaded class is not a View subclass
              final InflateException ie = new InflateException(attrs.getPositionDescription()
                      + ": Class is not a View " + (prefix != null ? (prefix + name) : name), e);
              ie.setStackTrace(EMPTY_STACK_TRACE);
              throw ie;
          } catch (ClassNotFoundException e) {
              // If loadClass fails, we should propagate the exception.
              throw e;
          } catch (Exception e) {
              final InflateException ie = new InflateException(
                      attrs.getPositionDescription() + ": Error inflating class "
                              + (clazz == null ? "<unknown>" : clazz.getName()), e);
              ie.setStackTrace(EMPTY_STACK_TRACE);
              throw ie;
          } finally {
              Trace.traceEnd(Trace.TRACE_TAG_VIEW);
          }
      }
      ```

### X2C的具体实现

```
 TODO：通过调试Demo、阅读和分析源码，对X2C代码实现进行梳理
```
   *  在Activity的onCreate中调用**X2C.setContentView**函数，最终通过调用**X2C.getView**函数得到View的实例并返回。
   
      ```JAVA
      /**
       * 设置contentview，检测如果有对应的java文件，使用java文件，否则使用xml
       *
       * @param activity 上下文
       * @param layoutId layout的资源id
       */
      public static void setContentView(Activity activity, int layoutId) {
          if (activity == null) {
              throw new IllegalArgumentException("Activity must not be null");
          }
          View view = getView(activity, layoutId);
          if (view != null) {
              activity.setContentView(view);
          } else {
              activity.setContentView(layoutId);
          }
      }
      ```
    
   *  在Fragment的onCreateView中调用**X2C.inflate**函数，最终调通过用**X2C.getView函数**得到View的实例并返回。
       
      ```JAVA
      /**
       * 加载xml文件，检测如果有对应的java文件，使用java文件，否则使用xml
       *
       * @param context  上下文
       * @param layoutId layout的资源id
       */
      public static View inflate(Context context, int layoutId, ViewGroup parent) {
          return inflate(context, layoutId, parent, true);
      }
      
      public static View inflate(Context context, int layoutId, ViewGroup parent, boolean attach) {
          if (context == null) {
              throw new IllegalArgumentException("Context must not be null");
          }
          View view = getView(context, layoutId);
          if (view != null) {
              if (parent != null) {
                  parent.addView(view);
              }
              return view;
          } else {
              return LayoutInflater.from(context).inflate(layoutId, parent, attach);
          }
      }
      ```
        
   *  由上可见，常用的Activity和Fragment最后都会调用到**核心代码 X2C.getView**函数，该函数根据X2C的注解声称规则，利用传入的layoutId找到对应的文件，使用**反射**得到View的实例并返回。
   
      ```JAVA
      public static View getView(Context context, int layoutId) {
          IViewCreator creator = sSparseArray.get(layoutId);
          if (creator == null) {
              try {
                  int group = generateGroupId(layoutId);
                  String layoutName = context.getResources().getResourceName(layoutId);
                  layoutName = layoutName.substring(layoutName.lastIndexOf("/") + 1);
                  String clzName = "com.zhangyue.we.x2c.X2C" + group + "_" + layoutName;
                  creator = (IViewCreator) context.getClassLoader().loadClass(clzName).newInstance();
              } catch (Exception e) {
                  e.printStackTrace();
              }
    
              //如果creator为空，放一个默认进去，防止每次都调用反射方法耗时
              if (creator == null) {
                  creator = new DefaultCreator();
              }
              sSparseArray.put(layoutId, creator);
          }
          return creator.createView(context);
      }
    
      private static class DefaultCreator implements IViewCreator {
    
          @Override
          public View createView(Context context) {
              return null;
          }
      }
    
      private static int generateGroupId(int layoutId) {
          return layoutId >> 24;
      }
      ```
   *  在**X2C.getView**函数中使用反射的到View的实例，此处View的类文件是通过注解在编译期间生成的，文件一般在**build/generated/source/apt/debug/com/zhangyue/we/x2c/layouts**目录下可以找到
      ```JAVA
      /**
       * WARN!!! dont edit this file
       * translate from {@link  com.zhangyue.we.x2c.demo.R.layout.activity}
       * autho chengwei 
       * email chengwei@zhangyue.com
       */
      public class X2C127_Activity implements IViewCreator {
        @Override
        public View createView(Context ctx) {
          	Resources res = ctx.getResources();
      
              LinearLayout linearLayout0 = new LinearLayout(ctx);
              linearLayout0.setId(R.id.constraintLayout);
              linearLayout0.setGravity(Gravity.CENTER);
              linearLayout0.setOrientation(LinearLayout.VERTICAL);
      
              Button button1 = new Button(ctx);
              LinearLayout.LayoutParams layoutParam1 = new LinearLayout.LayoutParams((int)(TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP,150,res.getDisplayMetrics())),(int)(TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP,50,res.getDisplayMetrics())));
              button1.setBackgroundColor(res.getColor(R.color.colorAccent));
              button1.setTextSize(TypedValue.COMPLEX_UNIT_DIP,20);
              button1.setGravity(Gravity.CENTER);
              button1.setTextColor(Color.parseColor("#ffffff"));
              button1.setId(R.id.x2c);
              button1.setText("X2C");
              button1.setLayoutParams(layoutParam1);
              linearLayout0.addView(button1);
      
              Button button2 = new Button(ctx);
              LinearLayout.LayoutParams layoutParam2 = new LinearLayout.LayoutParams((int)(TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP,150,res.getDisplayMetrics())),(int)(TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP,50,res.getDisplayMetrics())));
              button2.setBackgroundColor(res.getColor(R.color.colorAccent));
              button2.setTextSize(TypedValue.COMPLEX_UNIT_DIP,20);
              button2.setGravity(Gravity.CENTER);
              button2.setTextColor(Color.parseColor("#ffffff"));
              button2.setId(R.id.xml);
              layoutParam2.topMargin= (int)(TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP,10,res.getDisplayMetrics())) ;
              button2.setText("XML");
              button2.setLayoutParams(layoutParam2);
              linearLayout0.addView(button2);
      
              Button button3 = new Button(ctx);
              LinearLayout.LayoutParams layoutParam3 = new LinearLayout.LayoutParams((int)(TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP,150,res.getDisplayMetrics())),(int)(TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP,50,res.getDisplayMetrics())));
              button3.setBackgroundColor(res.getColor(R.color.colorAccent));
              button3.setTextSize(TypedValue.COMPLEX_UNIT_DIP,20);
              button3.setGravity(Gravity.CENTER);
              button3.setTextColor(Color.parseColor("#ffffff"));
              button3.setId(R.id.sub);
              layoutParam3.topMargin= (int)(TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP,10,res.getDisplayMetrics())) ;
              button3.setText("subModule");
              button3.setLayoutParams(layoutParam3);
              linearLayout0.addView(button3);
      
              return linearLayout0;
        }
      }
      ```
   *  注解的使用也是比较简单，在Activity或Fragment的实现类加上**@Xml(layouts = "layoutIdName")**即可
      ```JAVA
      @Xml(layouts = "activity")
      public class MainActivity extends Activity {
      
          @Override
          protected void onCreate(Bundle savedInstanceState) {
              super.onCreate(savedInstanceState);
              X2C.setContentView(this, R.layout.activity);
              ButterKnife.bind(this);
          }
          @OnClick(R.id.x2c)
          void toX2C() {
              startActivity(new Intent(this, ActivityX2C.class));
          }
      
          @OnClick(R.id.xml)
          void toXML() {
              startActivity(new Intent(this, ActivityXML.class));
          }
      
          @OnClick(R.id.sub)
          void toSub() {
              startActivity(new Intent(this, ActivitySubModule.class));
          }
      }
      ```
   *  bala？
   *  bala
      
### X2C带来了什么

   ```
 TODO：通过对原理和实现的学习，得出结论
   ```

   ```
 balabalabalabalabala
 balabalabalabalabala
 balabalabalabalabala
   ```

### X2C解决问题的能力边界及优缺点

   ```
 TODO：通过前面的学习，总结出X2C中可以为我们所用的部分（以及结合issue，得出现有问题）
   ```

   ``` 
 balabalabalabalabala
 balabalabalabalabala
 balabalabalabalabala
   ```

### 我们可以从X2C中学习到什么

```
 TODO：提炼出学习过程中对我们的启发 如：使用注解、对程序优化的极致追求 等..
```

   *  bala
   *  bala？
   *  bala
   *  bala
   *  bala？
   *  bala