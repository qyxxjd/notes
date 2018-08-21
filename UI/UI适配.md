> 一种优雅的屏幕UI适配方案，原理是通过：屏幕密度、DPI、字体缩放系数等计算，从而达到各种机型和原始UI保持高度一致的效果

- 简单易用的API
- 支持所有界面适配、部分界面适配
- 支持按宽度、高度进行适配
- 支持系统字体动态改变后的UI适配

#### 基本示例
```kotlin
class YouApp: Application()  {
    override fun onCreate()  {
        super.onCreate()
        // 简单易用，才是真爱
        WindowDensity.apply(this)
    }
}
```

#### 扩展用法
```kotlin
WindowDensity.apply(this, // Application
        true, // 是否适配所有Activity
        WindowDensity.ORIENTATION_WIDTH, // 适配方案：按宽度适配 或者 按高度适配
        960F, // 如果按屏幕高度适配，这里传入原始UI的高度
        540F // 如果按屏幕宽度适配，这里传入原始UI的宽度
)

// 如果你的项目需要适配部分页面，就是上面方法的第二个参数为false时，在需要适配的Activity添加一行代码
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
		// 关键代码
        WindowDensity.setOrientation(this)
		// or
		// WindowDensity.setOrientation(this, WindowDensity.ORIENTATION_HEIGHT)
    }
}
```

#### 源码实现
```kotlin

/**
 * 屏幕密度适配方案
 *
 * @author Classic
 * @version v1.0, 2018/8/17 上午9:40
 */
@Suppress("unused")
object WindowDensity {
    private val TAG = WindowDensity::class.java.simpleName

    /** 按屏幕方向：高度 进行适配 */
    const val ORIENTATION_HEIGHT = "height"
    /** 按屏幕方向：宽带 进行适配 */
    const val ORIENTATION_WIDTH = "width"

    /** 屏幕方向 */
    @Retention(AnnotationRetention.SOURCE)
    @StringDef(ORIENTATION_HEIGHT, ORIENTATION_WIDTH)
    annotation class Orientation

    /**
     * 目标UI的宽度，需要根据UI的宽度修改此参数。默认适配的UI宽度为:720px/xdpi
     */
    private const val TARGET_UI_WIDTH = 360F
    /**
     * 目标UI的高度，需要根据UI的高度修改此参数。默认适配的UI高度为:1280px/xdpi
     */
    private const val TARGET_UI_HEIGHT = 640F

    /** 状态栏高度 */
    private var sStatusBarHeight: Int = 0
    /** 屏幕高度 */
    private var sWindowHeightPixels: Int = 0
    /** 屏幕宽度 */
    private var sWindowWidthPixels: Int = 0
    /** 屏幕密度 */
    private var sDensity: Float = 0F
    /** 字体的缩放系数。正常情况下和屏幕密度相等，但是调节系统字体大小后会改变这个值 */
    private var sScaledDensity: Float = 0F
    private var sTargetHeight: Float = 0F
    private var sTargetWidth: Float = 0F

    /**
     * 应用屏幕适配方案
     *
     * @param application Application
     * @param adaptAllActivity 是否适配所有Activity
     * @param orientation 屏幕密度适配方向
     * @param targetHeight UI原图的高度
     * @param targetWidth UI原图的宽度
     */
    fun apply(application: Application, adaptAllActivity: Boolean = true,
              @Orientation orientation: String = ORIENTATION_WIDTH,
              targetHeight: Float = TARGET_UI_HEIGHT, targetWidth: Float = TARGET_UI_WIDTH) {
        sTargetHeight = targetHeight
        sTargetWidth = targetWidth
        sStatusBarHeight = getStatusBarHeight(application)
        application.resources.displayMetrics.apply {
            sWindowHeightPixels = heightPixels
            sWindowWidthPixels = widthPixels
            resetScaledAndDensity(this)
        }

        // 监听字体变化事件。修复系统字体修改后导致的UI适配问题
        application.registerComponentCallbacks(object : ComponentCallbacks {
            override fun onConfigurationChanged(newConfig: Configuration?) {
                if (newConfig != null && newConfig.fontScale > 0) {
                    resetScaledAndDensity(application.resources.displayMetrics)
                }
            }
            override fun onLowMemory() { }
        })
        if (adaptAllActivity) {
            application.registerActivityLifecycleCallbacks(object : Application.ActivityLifecycleCallbacks{
                override fun onActivityPaused(activity: Activity?) { }
                override fun onActivityResumed(activity: Activity?) { }
                override fun onActivityStarted(activity: Activity?) { }
                override fun onActivityDestroyed(activity: Activity?) { }
                override fun onActivitySaveInstanceState(activity: Activity?, outState: Bundle?) { }
                override fun onActivityStopped(activity: Activity?) { }
                override fun onActivityCreated(activity: Activity?, savedInstanceState: Bundle?) {
                    setOrientation(activity, orientation)
                }
            })
        }
    }

    /**
     * 重置字体缩放系数、屏幕密度
     */
    private fun resetScaledAndDensity(displayMetrics: DisplayMetrics) {
        sDensity = displayMetrics.density
        sScaledDensity = displayMetrics.scaledDensity
    }

    /**
     * 设置屏幕密度适配方向
     *
     * @param activity Activity
     * @param orientation 适配方向
     * @see ORIENTATION_HEIGHT
     * @see ORIENTATION_WIDTH
     */
    fun setOrientation(activity: Activity?, @Orientation orientation: String = ORIENTATION_WIDTH) {
        activity?.apply {
            val targetDensity: Float = when (orientation) {
                ORIENTATION_HEIGHT -> (sWindowHeightPixels - sStatusBarHeight) / sTargetHeight
                else -> sWindowWidthPixels / sTargetWidth
            }
            // 重新计算字体缩放系数
            val targetScaledDensity = targetDensity * (sScaledDensity / sDensity)
            val targetDensityDpi = (160 * targetDensity).toInt()

            // 将适配后的数据赋值到目标Activity
            resources.displayMetrics.apply {
                density = targetDensity
                scaledDensity = targetScaledDensity
                densityDpi = targetDensityDpi
            }
        }
    }

    /**
     * 获取状态栏高度
     *
     * @param context Context
     * @return 状态栏高度
     */
    private fun getStatusBarHeight(context: Context): Int {
        var result = 0
        val resourceId = context.resources.getIdentifier("status_bar_height", "dimen",
                "android")
        if (resourceId > 0) {
            result = context.resources.getDimensionPixelSize(resourceId)
        }
        return result
    }
}
```
