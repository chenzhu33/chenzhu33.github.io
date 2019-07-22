# Picasso Source Code Introduction

## 1. API Introduction

##### 1. DOWNLOAD

```    
    Picasso.with(context).load(url).into(view);
```

##### 2. RESOURCE LOADING

*Resources, assets, files, content providers are all supported as image sources.*

```
    Picasso.with(context).load(R.drawable.landing_screen).into(imageView1);
    Picasso.with(context).load("file:///android_asset/DvpvklR.png").into(imageView2);
    Picasso.with(context).load(new File(...)).into(imageView3);
```

##### 3. PLACE HOLDERS

*A request will be retried three times before the error placeholder is shown.*

```
    Picasso.with(context)
        .load(url)
        .placeholder(R.drawable.user_placeholder)
        .error(R.drawable.user_placeholder_error)
        .into(imageView);
```

##### 4. TRANSFORMATION & CUSTOM TRANSFORMATION

```
    Picasso.with(context)
        .load(url)
        .resize(50, 50)
        .centerCrop()
        .centerInside()
        .rotate(30)          
        .into([imageView/Target])
```

```
    public class CropSquareTransformation implements Transformation {
      @Override public Bitmap transform(Bitmap source) {
        int size = Math.min(source.getWidth(), source.getHeight());
        int x = (source.getWidth() - size) / 2;
        int y = (source.getHeight() - size) / 2;
        Bitmap result = Bitmap.createBitmap(source, x, y, size, size);
        if (result != source) {
          source.recycle();
        }
        return result;
      }
    
      @Override public String key() { return "square()"; }
    }

    Picasso.with(context)
        .load(url)
        .transform(new CropSquareTransformation)
        .into(imageView)
```

##### 5. CACHE

*Cache are allowed to be custom defined*

```
    Picasso.with(context)
        .load(url)
        .memoryPolicy(MemoryPolicy.NO_CACHE, MemoryPolicy.NO_STORE)
        .networkPolicy(NetworkPolicy.NO_CACHE, NetworkPolicy.NO_STORE)
        .into(imageView);
```

##### 6. OTHER FUNCTIONS

```
    Picasso.with(context)
        .load(url)
        .noFade()
        .onlyScaleDown()
        .priority(Picasso.Priority.HIGH)
        .config(Bitmap.Config.ALPHA_8)
        .into(imageView);
```

##### 7. DEBUG INDICATORS

*For development you can enable the display of a colored ribbon which indicates the image source. Call setIndicatorsEnabled(true) on the Picasso instance.*

![image](https://square.github.io/picasso/static/debug.png)

##### 8. PICASSO GLOBAL CONFIGURATIONS

*Picasso enables to config some global settings*

```
    Picasso sPicasso = new Picasso.Builder(context)
        .downloader(new OkHttpDownloader(context, ConfigConstants.MAX_CACHE_DISK_SIZE)) // disk cache size
        .memoryCache(new LruCache(ConfigConstants.MAX_CACHE_MEMORY_SIZE)) // memory cache size
        .defaultBitmapConfig(Bitmap.Config.ARGB_4444) // bitmap config
        .executor(new CustomExecutor()) // custom thread executor
        .downloader(new CustomDownloader()) // custom file downloader
        .requestTransformer(new CustomRequestTransformer()) // Transform a request before it is submitted to be processed.
        .requestHandler(new CustomRequestHandler()) // allows you to extend Picasso to load images in ways that are not supported by default in the library.
        .loggingEnabled(true) // log
        .indicatorsEnabled(true) // debug indicator
        .build();
```

## 2. Project Design & Structure




## 3. 