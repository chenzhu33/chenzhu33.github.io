# Fresco Source Code Introduction

## 1. API Introduction

*Drawee & Image Pipeline & Powerful Functions and Confiurations*

##### 1. INITIALIZE

```
    public class MyApplication extends Application {
    	@Override
    	public void onCreate() {
    		super.onCreate();
    		Fresco.initialize(this);
    	}
    }
```

##### 2. SimpleDraweeView & XML ATTRIBUTES

```
    <com.facebook.drawee.view.SimpleDraweeView
      android:id="@+id/my_image_view"
      android:layout_width="20dp"
      android:layout_height="20dp"
      fresco:fadeDuration="300"
      fresco:actualImageScaleType="focusCrop"
      fresco:placeholderImage="@color/wait_color"
      fresco:placeholderImageScaleType="fitCenter"
      fresco:failureImage="@drawable/error"
      fresco:failureImageScaleType="centerInside"
      fresco:retryImage="@drawable/retrying"
      fresco:retryImageScaleType="centerCrop"
      fresco:progressBarImage="@drawable/progress_bar"
      fresco:progressBarImageScaleType="centerInside"
      fresco:progressBarAutoRotateInterval="1000"
      fresco:backgroundImage="@color/blue"
      fresco:overlayImage="@drawable/watermark"
      fresco:pressedStateOverlayImage="@color/red"
      fresco:roundAsCircle="false"
      fresco:roundedCornerRadius="1dp"
      fresco:roundTopLeft="true"
      fresco:roundTopRight="false"
      fresco:roundBottomLeft="false"
      fresco:roundBottomRight="true"
      fresco:roundWithOverlayColor="@color/corner_color"
      fresco:roundingBorderWidth="2dp"
      fresco:roundingBorderColor="@color/border_color"
    />
```

```
    Uri uri = Uri.parse("https://raw.githubusercontent.com/facebook/fresco/gh-pages/static/logo.png");
    SimpleDraweeView draweeView = (SimpleDraweeView) findViewById(R.id.my_image_view);
    draweeView.setImageURI(uri);
```

*All will be done by fresco, such as:*
- *show placeholder*
- *cache*
- *download*
- *remove from memory when not show*

**wrap_content**

*Notify that Drawees **doesn't** support **wrap_content** *

**Aspect Ratio**

*Fresco support fixed aspect ratio to display image*

```
    <com.facebook.drawee.view.SimpleDraweeView
        android:id="@+id/my_image_view"
        android:layout_width="20dp"
        android:layout_height="wrap_content"
        fresco:viewAspectRatio="1.33"
        <!-- other attributes -->
```

```
    mSimpleDraweeView.setAspectRatio(1.33f);
```

##### 3. DraweeHierarchy

##### 4. ControllerBuilder

##### 5. ProgressiveRendering

##### 6. DraweeHolder & MultiDraweeHolder 

##### 7. ImageRequest

##### 8. Image Pipeline