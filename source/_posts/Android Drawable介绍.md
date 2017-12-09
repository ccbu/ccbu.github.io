title: Android Drawable介绍
author: ccbu
tags: []
categories:
  - android
date: 2017-12-06 22:01:00
---

Android系统中将可绘制对象被抽象为Drawable,不同的绘制资源对应着不同的Drawable类型。Android FrameWork提供了常用的Drawable，Android控件的绘制资源基本都是通过Drawable形式实现的。一般情况下，开发者是不会直接接触Drawable的具体实现的，Drawable资源一般都放在res/drawable目录下，用户通过图片，xml格式的Drawable资源来使用。 Android内置的比较常用的Drawable类型包括：ColorDrawable、GradientDrawable、ShapeDrawable、BitmapDrawable、 NinePatchDrawable、InsetDrawable、ClipDrawable、ScaleDrawable、RotateDrawable、AnimationDrawable、LayerDrawable、LevelListDrawable、StateListDrawable、TransitionDrawable。 一般情况下，除了图片资源是直接放在res/drawable下（android studio中图片放在res/minmap下），其他的Drawable都是以xml格式实现的，开发者通过在xml中使用shape,selector,level-list等标签来实现对应的Drawable，从而实现相应的可绘制资源的定义，最终view通过绘制这些Drawable来实现我们想要的显示效果。

Drawable中xml标签与Drawable对象的对应关系如以下表格所示:

| xml标签 | Drawable对象 |
| ---- | ---- |
| &lt;selector&gt; | StateListDrawable |
| &lt;level-list&gt; | LevelListDrawable |
| &lt;layer-list&gt; | LayerDrawable |
| &lt;transition&gt; | TransitionDrawable |
| &lt;color&gt; | ColorDrawable |
| &lt;shape&gt; | GradientDrawable |
| &lt;scale&gt; | ScaleDrawable |
| &lt;vector&gt; | VectorDrawable |
| &lt;clip&gt; | ClipDrawable |
| &lt;rotate&gt; | RotateDrawable |
| &lt;anination-list&gt; | AnimationDrawable |
| &lt;inset&gt; | InsetDrawable |
| &lt;bitmap&gt; | BitmapDrawable |
| &lt;nine-patch&gt; | NinePatchDrawable |

下面我们通过Framework的源代码来探索一下Drawable的加载过程，一般情况下，我们通过getResources().getDrawable(int id);的方式去加载。顺着这条线往下看，getDrawable->loadDrawable->loadDrawableForCookie->Drawable.createFromXml->createFromXmlInner,来看android\graphics\drawable\Drawable.java的createFromXmlInner函数的实现代码。
```
public static Drawable createFromXmlInner(Resources r, XmlPullParser parser, AttributeSet attrs,
            Theme theme) throws XmlPullParserException, IOException {
        final Drawable drawable;

        final String name = parser.getName();
        if (name.equals("selector")) {
            drawable = new StateListDrawable();
        } else if (name.equals("animated-selector")) {
            drawable = new AnimatedStateListDrawable();
        } else if (name.equals("level-list")) {
            drawable = new LevelListDrawable();
        } else if (name.equals("layer-list")) {
            drawable = new LayerDrawable();
        } else if (name.equals("transition")) {
            drawable = new TransitionDrawable();
        } else if (name.equals("ripple")) {
            drawable = new RippleDrawable();
        } else if (name.equals("color")) {
            drawable = new ColorDrawable();
        } else if (name.equals("shape")) {
            drawable = new GradientDrawable();
        } else if (name.equals("vector")) {
            drawable = new VectorDrawable();
        } else if (name.equals("animated-vector")) {
            drawable = new AnimatedVectorDrawable();
        } else if (name.equals("scale")) {
            drawable = new ScaleDrawable();
        } else if (name.equals("clip")) {
            drawable = new ClipDrawable();
        } else if (name.equals("rotate")) {
            drawable = new RotateDrawable();
        } else if (name.equals("animated-rotate")) {
            drawable = new AnimatedRotateDrawable();
        } else if (name.equals("animation-list")) {
            drawable = new AnimationDrawable();
        } else if (name.equals("inset")) {
            drawable = new InsetDrawable();
        } else if (name.equals("bitmap")) {
            //noinspection deprecation
            drawable = new BitmapDrawable(r);
            if (r != null) {
               ((BitmapDrawable) drawable).setTargetDensity(r.getDisplayMetrics());
            }
        } else if (name.equals("nine-patch")) {
            drawable = new NinePatchDrawable();
            if (r != null) {
                ((NinePatchDrawable) drawable).setTargetDensity(r.getDisplayMetrics());
             }
        } else {
            throw new XmlPullParserException(parser.getPositionDescription() +
                    ": invalid drawable tag " + name);
        }

        drawable.inflate(r, parser, attrs, theme);
        return drawable;
    }
```
上面的对应关系就从这里来的，现在是不是一下子明朗了呢。在函数结尾，Drawable通过条用inflate函数解析AttributeSet中的属性信息，并设置Drawable相应的属性。看一下BitmapDrawable吧，
```
public void inflate(Resources r, XmlPullParser parser, AttributeSet attrs, Theme theme)
            throws XmlPullParserException, IOException {
        super.inflate(r, parser, attrs, theme);

        final TypedArray a = obtainAttributes(r, theme, attrs, R.styleable.BitmapDrawable);
        updateStateFromTypedArray(a);
        verifyState(a);
        a.recycle();
    }
```
继续看updateStateFromTypedArray
```
private void updateStateFromTypedArray(TypedArray a) throws XmlPullParserException {
        final Resources r = a.getResources();
        final BitmapState state = mBitmapState;

        // Account for any configuration changes.
        state.mChangingConfigurations |= a.getChangingConfigurations();

        // Extract the theme attributes, if any.
        state.mThemeAttrs = a.extractThemeAttrs();

        final int srcResId = a.getResourceId(R.styleable.BitmapDrawable_src, 0);
        if (srcResId != 0) {
            final Bitmap bitmap = BitmapFactory.decodeResource(r, srcResId);
            if (bitmap == null) {
                throw new XmlPullParserException(a.getPositionDescription() +
                        ": <bitmap> requires a valid src attribute");
            }

            state.mBitmap = bitmap;
        }

        state.mTargetDensity = r.getDisplayMetrics().densityDpi;

        final boolean defMipMap = state.mBitmap != null ? state.mBitmap.hasMipMap() : false;
        setMipMap(a.getBoolean(R.styleable.BitmapDrawable_mipMap, defMipMap));

        state.mAutoMirrored = a.getBoolean(
                R.styleable.BitmapDrawable_autoMirrored, state.mAutoMirrored);
        state.mBaseAlpha = a.getFloat(R.styleable.BitmapDrawable_alpha, state.mBaseAlpha);

        final int tintMode = a.getInt(R.styleable.BitmapDrawable_tintMode, -1);
        if (tintMode != -1) {
            state.mTintMode = Drawable.parseTintMode(tintMode, Mode.SRC_IN);
        }

        final ColorStateList tint = a.getColorStateList(R.styleable.BitmapDrawable_tint);
        if (tint != null) {
            state.mTint = tint;
        }

        final Paint paint = mBitmapState.mPaint;
        paint.setAntiAlias(a.getBoolean(
                R.styleable.BitmapDrawable_antialias, paint.isAntiAlias()));
        paint.setFilterBitmap(a.getBoolean(
                R.styleable.BitmapDrawable_filter, paint.isFilterBitmap()));
        paint.setDither(a.getBoolean(R.styleable.BitmapDrawable_dither, paint.isDither()));

        setGravity(a.getInt(R.styleable.BitmapDrawable_gravity, state.mGravity));

        final int tileMode = a.getInt(R.styleable.BitmapDrawable_tileMode, TILE_MODE_UNDEFINED);
        if (tileMode != TILE_MODE_UNDEFINED) {
            final Shader.TileMode mode = parseTileMode(tileMode);
            setTileModeXY(mode, mode);
        }

        final int tileModeX = a.getInt(R.styleable.BitmapDrawable_tileModeX, TILE_MODE_UNDEFINED);
        if (tileModeX != TILE_MODE_UNDEFINED) {
            setTileModeX(parseTileMode(tileModeX));
        }

        final int tileModeY = a.getInt(R.styleable.BitmapDrawable_tileModeY, TILE_MODE_UNDEFINED);
        if (tileModeY != TILE_MODE_UNDEFINED) {
            setTileModeY(parseTileMode(tileModeY));
        }

        // Update local properties.
        initializeWithState(state, r);
    }
```
updateStateFromTypedArray函数中，BitmapDrawable获取到src属性后，通过BitmapFactory加载图像文件，并读取用户配置的属性，设置BitmapDrawable的其他属性。其他的Drawable的实现方式类似。