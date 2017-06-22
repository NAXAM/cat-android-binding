CAT - Xamarin Binding Library
===

A logging library for Android with focus on convenient usage and extensibility.

Download
--------
```
Install-Package Naxam.Cat.Droid
```

Usage
-----

The class `Cat` serves as entry point and provides many logging methods you may know from `android.util.Log`. The default logging class uses the class name as logging tag.

```c#
public void LogSomething() {
	Cat.D("Hello world");
}
```

Each logging method has the option to pass arguments for a formatted log message using `Java.Lang.String.Format()`.

 ```c#
 public void LogSomething() {
 	Cat.D("%s < %d == %b", "1", 4, true); // "1 < 4 == true"
 }
 ```

Advanced
--------

`Cat` internally uses `CatLazy` which extends from `CatLog`. If you want to use another logging tag, you can create an instance of `CatSimple`.

```java
public class MainActivity extends Activity {

    private static final CatLog CAT = new CatSimple("CustomTag");

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        CAT.d("log with another tag");
    }
}
```

`CatGlobal` provides useful options to change the default logging behavior. E.g. you can disable a specific tag or even package.

```java
CatGlobal.setPackageEnabled("net.vrallev", false); // also disables "net.vrallev.*
CatGlobal.setTagEnabled("MyTag", false);
```

It's even possible to add more log targets. Implement the `CatPrinter` interface and add the printer to a specific `CatLog` instance. `AndroidLog` is the default printer and writers all log entries into LogCat.

```java
public class FilePrinter implements CatPrinter {

    private final Context mContext;
    private final DateFormat mDateFormat;

    public FilePrinter(Context context) {
        mContext = context;
        mDateFormat = new SimpleDateFormat("HH:mm:ss.SSS", Locale.getDefault());
    }

    @Override
    public void println(int priority, @NonNull String tag, @NonNull String msg, @Nullable Throwable t) {
        msg = mDateFormat.format(new Date()) + "\t\t" + tag + "\t\t" + msg + '\n';

        try {
            FileUtils.writeFile(getLogFile(), msg, true);
        } catch (IOException e) {
            Cat.e(e);
        }
    }

    public synchronized File getLogFile() {
        return new File(mContext.getCacheDir(), "log.txt");
    }
}

public class FileActivity extends Activity {

    private static final FilePrinter FILE_PRINTER = new FilePrinter(App.instance());

    private static final CatLog CAT = new CatSimple(FileActivity.class) {{
            addPrinter(FILE_PRINTER);
    }};

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_file);

        CAT.d("Hello world in the log file");
    }
}
```

Attributions
===

    @vrallev: https://github.com/vRallev