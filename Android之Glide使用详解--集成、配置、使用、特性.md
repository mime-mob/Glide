**一、Glide介绍**
在Android开发中，图片加载是必不可少的，Glide作为谷歌推荐的图片库，现在越来越火。
Glide 是一个 Android 上的图片加载和缓存库，它不仅能实现平滑的图片列表滚动效果，还支持远程图片的获取、大小调整和展示，并且可以加载Gif动态图，可谓功能强大。在我看来，可能现在大部分小伙伴还是比较喜欢Image Loader，毕竟用了很多年，也习惯了，但是我们思维不能被它束缚，而且官方已经声明不再维护该库，难道已经完美了吗？Glide毕竟是谷歌推荐，已经开源，说明至少是稳定的，而且比较轻量，不管是功能还是性能上都优于其他的（个人意见）。

**二、Glide集成**
1.项目中集成Glide
在gradle中添加Glide库：
```
dependencies {
    compile 'com.github.bumptech.glide:glide:3.7.0'
    compile 'com.android.support:appcompat-v7:23.1.1'
}
```
Glide的集成离不开v4包，所以必须添加support包。
2.Glide集成其他库
Glide包含一些小的、可选的集成库，目前Glide集成库当中包含了访问网络操作的Volley和OkHttp：
（1）Volley集成
第一步、添加依赖
```
dependencies {
    compile 'com.github.bumptech.glide:volley-integration:1.2.2'
    compile 'com.mcxiaoke.volley:library:1.0.5'
}
```
第二步、创建Volley集成库的GlideModule
```
<meta-data
 android:name="com.bumptech.glide.integration.volley.VolleyGlideModule"
    android:value="GlideModule" />
```
然后改变混淆文件：
```
-keep class com.bumptech.glide.integration.volley.VolleyGlideModule
#or
-keep public class * implements com.bumptech.glide.module.GlideModule
```
（2）OkHttp集成
第一步、添加依赖
```
dependencies {
    compile 'com.github.bumptech.glide:okhttp-integration:1.2.2'
    compile 'com.squareup.okhttp:okhttp:2.0.0'
}
```
第二步、创建Volley集成库的GlideModule
```
<meta-data
    android:name="com.bumptech.glide.integration.okhttp.OkHttpGlideModule"
    android:value="GlideModule" />
```
然后改变混淆文件：
```
-keep class com.bumptech.glide.integration.okhttp.OkHttpGlideModule
#or
-keep public class * implements com.bumptech.glide.module.GlideModule
```
（3）集成转换器（此处只介绍集成库，使用后面再讲）
第一步、gradle添加依赖
```
repositories {
    jcenter()
    mavenCentral()  // GPUImage for Android
}

dependencies {
    compile 'jp.wasabeef:glide-transformations:2.0.1'
    // If you want to use the GPU Filters
    compile 'jp.co.cyberagent.android.gpuimage:gpuimage-library:1.3.0'
}
```
通过这个转换器库，可以实现各式各样的图片，非常强大。

**三、Glide配置**
Glide如同ImageLoader一样，也是可以配置一些属性的，Glide可以在GlideModel中统一配置其属性。
1.第一步：
```
public class GlideModelConfig implements GlideModule {

    int diskSize = 1024 * 1024 * 100;
    int memorySize = (int) (Runtime.getRuntime().maxMemory()) / 8;  // 取1/8最大内存作为最大缓存

    @Override
    public void applyOptions(Context context, GlideBuilder builder) {
        // 定义缓存大小和位置
        builder.setDiskCache(new InternalCacheDiskCacheFactory(context, diskSize));  //内存中
        builder.setDiskCache(new ExternalCacheDiskCacheFactory(context, "cache", diskSize)); //sd卡中

        // 默认内存和图片池大小
        MemorySizeCalculator calculator = new MemorySizeCalculator(context);
        int defaultMemoryCacheSize = calculator.getMemoryCacheSize(); // 默认内存大小
        int defaultBitmapPoolSize = calculator.getBitmapPoolSize(); // 默认图片池大小
        builder.setMemoryCache(new LruResourceCache(defaultMemoryCacheSize)); // 该两句无需设置，是默认的
        builder.setBitmapPool(new LruBitmapPool(defaultBitmapPoolSize));

        // 自定义内存和图片池大小
        builder.setMemoryCache(new LruResourceCache(memorySize));
        builder.setBitmapPool(new LruBitmapPool(memorySize));

        // 定义图片格式
        builder.setDecodeFormat(DecodeFormat.PREFER_ARGB_8888);
        builder.setDecodeFormat(DecodeFormat.PREFER_RGB_565); // 默认
    }

    @Override
    public void registerComponents(Context context, Glide glide) {
    }
}
```
2.第二步：
这GlideModel可以在AndroidManifest.xml文件中注册，以便Glide能够找到你的Module
```
<meta-data
            android:name="com.example.jianglei.glidedemo.GlideModelConfig"
            android:value="GlideModule" />
```
3.第三步：
将上面的实现了加入到proguard.cfg当中，代码混淆：
```
-keepnames class * com.example.jianglei.glidedemo.GlideModelConfig
```
可能遇到的问题：
Glide允许一个应用当中存在多个GlideModules，但是Glide并不会按照一个特殊的顺序去调用已注册的GlideModules，如果一个应用的多个依赖工程当中有多个相同的Modules，就有可能会产生冲突。 
如果一个冲突是不可避免的，应用应该默认去定义一个自己的Module,用来手动地处理这个冲突，在进行Manifest合并的时候，可以用下面的标签排除冲突的Module：
```
<meta-data android:name=”com.example.jianglei.glidedemo.GlideModelConfig” tools:node=”remove”/>
```

**四、Glide使用详情**
1.基本使用：
```
Glide.with(context)  
    .load("xxxx.png")  
    .into(imageView);
```
Glide的with()可以接受的类型有如下：
```
Context context;
Activity activity;
FragmentActivity fragmentActivity;
Fragment fragment;
```
load()是加载目标资源，可以接受的参数类型有如下：
```
Uri uri;
String uriString;
File file;
Integer resourceId;
byte[] model;
String model;
```
into()就是加载资源完成后作什么处理，它接受三种参数：
```
// 显示在控件上
into(ImageView imageView);

// 通过回调获得加载结果，可能在项目中，一个图片在多个地方使用，可以在回调中获得该图片的Bitmap操作
into(Target target);
Glide.with(mContext).load(url).asBitmap().into(new SimpleTarget<Bitmap>() {
            @Override
            public void onResourceReady(Bitmap resource, GlideAnimation glideAnimation) {
                image.setImageBitmap(resource);
            }
        });
Glide.with(mContext).load(url).asBitmap().into(new Target<Bitmap>() {
            @Override
            public void onLoadStarted(Drawable placeholder) {
                // 设置加载过程中的Drawable
            }

            @Override
            public void onLoadFailed(Exception e, Drawable errorDrawable) {
                // 设置加载失败的Drawable
            }

            @Override
            public void onResourceReady(Bitmap resource, GlideAnimation<? super Bitmap> glideAnimation) {
                // 设置加载成功的Bitmap
            }

            @Override
            public void onLoadCleared(Drawable placeholder) {
                // 设置加载被取消时的Drawable
            }

            @Override
            public void getSize(SizeReadyCallback cb) {}

            @Override
            public void setRequest(Request request) {}

            @Override
            public Request getRequest() {
                return null;
            }

            @Override
            public void onStart() {}

            @Override
            public void onStop() {}

            @Override
            public void onDestroy() {}
        });
        
// 指定期望的图片大小，返回一个Bitmap对象，要在子线程中获取，我觉得这个可以在应用使用，比如在主页我需要显示这个图片，我可以在欢迎页面加载这个图片，到了主页可以直接使用，至于为何要在线程中使用，有待研究。
into(int w, int h);
new Thread(new Runnable() {
        @Override
        public void run() {
            Bitmap bitmap = Glide.with(context)  
                .load(url)  
                .into(200， 200)
                .get();
        }
    }).start();
```
除了with(),load(),into()三个基本的方法，Glide还有很多使用的Api，下面一一道来。
2.设置bitmap或者gif
```
.asBitmap();
.asGif();
```
这两句话，有时候不设置是可以的，但是有时候配合其他属性一起设置的时候，如果没有这个属性的话，其他属性无法实现，所以大家还是规范一些，写完整些。
3.设置图片大小 
```
.override(int w, int h); 
```
指定加载bitmap的大小，比如原图是500 x 500，into(100， 100)，加载出的bitmap就是100 x 100，这样就可以适配各种所需的UI大小，Glide已经为你做好了比例缩放，经过我的试验，如果图片是720 x 1280，将其设置成720 x 500，该图片将会以500这个尺寸比例缩放。
```
Glide.with(this).load(R.mipmap.login).asBitmap().override(720, 500).into(imageView);
```
![这里写图片描述](http://img.blog.csdn.net/20160428144239980)                                       ![这里写图片描述](http://img.blog.csdn.net/20160428144301164)
如上图所示，左边为未使用override方法，右边使用override方法宽不变，高变成500，所以图片以短的为标准缩放图片。

4.加载缩略图
```
thumbnail(0.1f);
```
它是在你into的view中先加载设置的缩略图，然后才会加载大图，注：参数范围为0~1。

5.设置占位图或者加载错误图：
```
.placeholder(R.drawable.placeholder)  
.error(R.drawable.imagenotfound) 
```
在实际项目中，为了优化显示，比如头像在下载过程中，会使用一张默认的图片先在UI上面显示，则用placeholder，若加载成功，则显示下载图片，否则就显示加载失败的图片，使用error。

6.加载完成动画 
```
.animate(Animator animator);//或者int animationId 
```
初次加载出Bitmap时展示的动画，可以是属性动画，也可以是Tween动画，可以是加载代码动画，也可以是动画资源文件，方便的很。
注：这个动画只在初次加载出来时使用，已经加载过了，下载再从缓存中取是不会动画的。

7.图片适配scaleType
```
.centerCrop(); // 长的一边撑满
.fitCenter(); // 短的一边撑满
```
![这里写图片描述](http://img.blog.csdn.net/20160428144358087)        ![这里写图片描述](http://img.blog.csdn.net/20160428144438915)
如上两个图，左侧使用centerCrop，发现它是以他的宽来适配当前view大小，右侧使用fitCenter，可以看出是以长来适配当前view的大小，这哥是跟imageView的差不多理解。

8.暂停\回复请求 
```
Glide.with(context).resumeRequests(); 
Glide.with(context).pauseRequests(); 
```
当列表在滑动的时候，调用pauseRequests()取消请求，滑动停止时，调用resumeRequests()恢复请求。

9.在后台线程当中进行加载和缓存
```
downloadOnly(int width, int height)
downloadOnly(Y target)// Y extends Target<File>
into(int width, int height)
```
Glide的downloadOnly()是将Image的二进制文件下载到硬盘缓存当中，以便在后续使用，可以在UI线程当中异步下载，也可以在异步线程当中同步调用下载，值得注意的是，如果在同步线程当中， 
downloadOnly使用一个target作为参数，而在异步线程当中则是使用width和height。
在后台线程当中下载图片，可以通过如下的方式：
```
new Thread(new Runnable() {
        @Override
        public void run() {
            FutureTarget<File> future = Glide.with(applicationContext)
                                             .load(yourUrl)
                                             .downloadOnly(500, 500);
            File cacheFile = future.get();
        }
    }).start();
```
上面的调用只能在异步线程当中，如果在main Thread当中调用.get()，会阻塞主线程，会导致Crash。
这个其实就是可以获取该Image的缓存路径，目前还没想好怎么封装一个获取缓存路径的通用类，有待研究。至于Glide的缓存策略，这里也简要介绍下，也是配置：
缓存策略 
```
.diskCacheStrategy(DiskCacheStrategy.ALL) //这个是设置缓存策略。
DiskCacheStrategy.NONE：不缓存 
DiskCacheStrategy.SOURCE：缓存原始图片 
DiskCacheStrategy.RESULT：缓存压缩过的结果图片 
DiskCacheStrategy.ALL：两个都缓存
```
10.转换器
第七点中的centerCrop和fitCenter是Glide默认的两种转换器，现在已经有了Glide集成库，提供各式各样的Api可以将图片转为各种形状，例如圆形，圆角型等等，库呢在上面已经介绍过啦，下面就来使用这些库看看效果。
（1）单独转换器效果（毛玻璃为例）
```
Glide.with(this).load(R.mipmap.login).bitmapTransform(new BlurTransformation(this, 20)).into(imageView);
```
先来看看效果：
![这里写图片描述](http://img.blog.csdn.net/20160426163506471)
可以看到毛玻璃效果，BlurTransformation就是库提供的类，20这个数字是毛玻璃的模糊程度设置，注意范围是1~25，否则图片将不会显示出来。
（2）复合转换器效果
```
Glide.with(this).load(R.mipmap.login).bitmapTransform(new BlurTransformation(this, 20), new CropCircleTransformation(this)).into(imageView);
```
同样先来看看效果：
![这里写图片描述](http://img.blog.csdn.net/20160426163844297)
可以看到，图片不仅变成毛玻璃，还变成了圆形。
我们可以来看看bitmapTransform这个方法：
```
public DrawableRequestBuilder<ModelType> bitmapTransform(Transformation<Bitmap>... bitmapTransformations) {
        GifBitmapWrapperTransformation[] transformations =
                new GifBitmapWrapperTransformation[bitmapTransformations.length];
        for (int i = 0; i < bitmapTransformations.length; i++) {
            transformations[i] = new GifBitmapWrapperTransformation(glide.getBitmapPool(), bitmapTransformations[i]);
        }
        return transform(transformations);
    }
```
可以通过看代码发现，bitmapTransform这个方法可以传入多种转换器，实现不同的效果，变通性很强。
下面我来列一下多种Transformations，具体的我也不展示了，留给大家去尝试：
Crop
默认：CropTransformation,
圆形：CropCircleTransformation,
方形：CropSquareTransformation, 
圆角：RoundedCornersTransformation

Color
颜色覆盖：ColorFilterTransformation, 
置灰：GrayscaleTransformation

Blur
毛玻璃：BlurTransformation

Mask
还未弄明白：MaskTransformation

尝试完这些，你就可以发现Glide的强大之处！
当然，除了上面库提供的这些，还可以自定义转换器。
第一步、编写转换器类 
继承BitmapTransformation:
```
private static class MyTransformation extends BitmapTransformation {
    public MyTransformation(Context context) {
       super(context);
    }
    @Override
    protected Bitmap transform(BitmapPool pool, Bitmap toTransform, 
            int outWidth, int outHeight) {
       Bitmap myTransformedBitmap = ... // 在这里自定义图片转换，跟平时自己裁剪截图什么的一样
       return myTransformedBitmap;
    }
    @Override
    public String getId() {
        // Return some id that uniquely identifies your transformation.
        return "com.example.myapp.MyTransformation";
    }
}
```
第二步、在Glide方法链当中用.transform(…)替换fitCenter()/centerCrop()
```
Glide.with(context)
    .load(Url)
    .transform(new MyTransformation(context))
    .into(imageView);
```
在上面使用过程当中没有设置尺寸值，那么转换器转换的图片尺寸怎么确定呢，Glide实际上已经足够智能根据view的尺寸来确定转换图片的尺寸了，如果需要自定义尺寸，而不是用view和target当中的尺寸，那么可以使用override(int,int)设置相关的宽和高。

以上就是Glide的常用的基本用法，可能有些api没有列出，大家可以自己去研究。

**五、Glide总结**
1.Glide使用
从第四节中介绍Glide使用中，大家可看到Glide Api众多，而且设置方便，语言简单，但是在使用过程中要注意使用顺序，大家可以试试看，写一短句然后“.”一下，看看能提示哪些Api，这些都可以看出来，这里就不多说了。
2.Glide加载图片生命周期 
对于图片请求会在onStop的时候自动暂停，然后在onStart的时候重新启动，gif的动画也会在onStop的时候停止，以免在后台消耗电量， 此外，当设备的网络状态发生改变的时候，所有失败的请求会自动重启，保证数据的正确性，还是比较人性化、自动化的。
3.Glide缓存
Glide提供多种缓存机制，对于图片原图和Resize的图片可以自由缓存，它相比于Picasso，内存消耗要小很多。
4.Glide使用场景
Glide使用最好是在主线程中使用，因为它需要context，在非主线程中使用Glide可能会导致内存泄露或者更严重的Crash，相信大家多Context的使用应该是非常谨慎的，非要在非主线程使用Glide的话就将context换成getApplicationContext。可以参考下[https://github.com/bumptech/glide/issues/138这个问题](https://github.com/bumptech/glide/issues/138)。
5.Glide在别的库的ImageView的表现
如果你刚好使用了这个圆形Imageview库或者其他的一些自定义的圆形ImageView，而你又刚好设置了占位的话，那么，你就会遇到一个问题，就是有的图片第一次加载的时候只显示占位图，第二次才显示正常的图片，这个问题有三个方案：
（1）不设置占位图，当然不推荐这个方法
（2）如果自定义ImageView的效果在Glide的Transformation API中有，那就使用Glide的Transformation API去实现，毕竟这是专门为Glide设计的库；如果没有，可以自定义Glide的Transformation，这个在上面的文章中已经写过啦。
（3）只使用Glide加载，拿到Bitmap之后再去设置图片：
```
Glide.with(mContext)
    .load(url) 
    .placeholder(R.drawable.default)
    .into(new SimpleTarget<Bitmap>(width, height) {
        @Override 
        public void onResourceReady(Bitmap bitmap, GlideAnimation anim) {
            // setImageBitmap(bitmap) on imageView 
        } 
    };
```

**以上就是我使用Glide的一些理解，Glide使用方便、速度快、内存小、效果多样，相信在ImageLoader帝国之后会成为主流图片库。**

**不说了，搬砖去了！**
![这里写图片描述](http://img.blog.csdn.net/20160428161514760)




