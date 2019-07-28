---
title: How to drag and drop in android
layout: post
---

![Android](/images/2015/Android-logo-PSD.jpg)

Android provides support for drag and drop since API level 11. In this post we'll see how drag&drop works.

At a high level, the drag and drop operation works as follows: you first start by recognizing the drag - in this case we'll use a long click. Afterwards you set listeners on the views that are to be informed of the object you are dragging's position. That's it.

Start by creating a new project in your favorite IDE. The layout is simple, we have a `RelativeLayout` and an `ImageView` we'd like to drag. 

```language-html-line-numbers
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:id="@+id/layoutTo"
    tools:context=".MainActivity">

    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@drawable/android"
        android:id="@+id/iv_android"
        android:layout_alignParentTop="true"
        android:layout_centerHorizontal="true" />

</RelativeLayout>
```

PS: For this example you can copy the little android image from the `/resources/mipmap` directory and paste it in the `drawable` directory.

Your MainActivity's `onCreate()` method will look like this:


```language-java-line-numbers
...
private ImageView androidIv;
private RelativeLayout layoutTo;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    layoutTo = (RelativeLayout) findViewById(R.id.layoutTo);
    layoutTo.setOnDragListener(this);

    androidIv = (ImageView) findViewById(R.id.androidIv);
    androidIv.setTag("android");
    androidIv.setOnLongClickListener(new View.OnLongClickListener() {

        @Override
        public boolean onLongClick(View v) {
            ClipData.Item item = new ClipData.Item((String) androidIv.getTag());
            ClipData dragData = new ClipData((String) androidIv.getTag(),
                    new String[] { ClipDescription.MIMETYPE_TEXT_PLAIN },
                    item);
            View.DragShadowBuilder shadow = new View.DragShadowBuilder(v);

            // Start the drag
            v.startDrag(
                    dragData,
                    shadow,
                    v,
                    0
            );

            return true;
        }
    }); 
```

We get a reference to the `RelativeLayout` and the `ImageView`. We set the `onLongClickListener` on the image view and implement the interface `View.OnLongClickListener()`. Before we can start the drag we need to do a couple of things. 

First, create a new `ClipData`, which is a class that represents the dragged data. Next, we need a `View.DragShadowBuilder`. This is the image you see when you move the object around the screen. You can pass in the same view and it will use that as the drag shadow image centered on your finger. It can also be null but in that case you won't see anything even though the drag operation has started.You can override this to create your own drag shadow. 

Then start the drag operation with the `startDrag()` call on the view from `onClick()`. It takes 4 parameters:

1. The Clipdata 
2. The drag shadow (as said above can be null)
3. An extra object which you can use to send extra information. We'll pass in our image view. 
4. Extra flags. Set this to 0 as no flags are defined by the framework.

Now you can move the object, but you want the underlying view to be informed where the object is so you know where the object gets dropped. 

We got the reference for the `RelativeLayout` and we're going to set a listener on it, `View.OnDragListener`. I implemented it in the class - as in ``MainActivity extends Activity implements View.OnDragListener`` - so you can use `relativeLayout.setOnDragListener(this)`. 

Implementing this interface requires the implementation of `onDrag(View, DragEvent)`. The method looks like this

```language-java-line-numbers
@Override
public boolean onDrag(View v, DragEvent event) {
    final int action = event.getAction();
    ImageView iv = (ImageView) event.getLocalState();
    
    switch (action) {
        case DragEvent.ACTION_DRAG_STARTED:
            iv.setVisibility(View.INVISIBLE);
            break;

        case DragEvent.ACTION_DRAG_ENTERED:
            break;

        case DragEvent.ACTION_DRAG_LOCATION:  
            return true;

        case DragEvent.ACTION_DRAG_EXITED:
            break;

        case DragEvent.ACTION_DROP:
            // Get X and Y and move the image to this new location
            iv.setVisibility(View.VISIBLE);
            iv.setX(event.getX() - (iv.getWidth()/2));
            iv.setY(event.getY() - (iv.getHeight()/2));

            break;

        case DragEvent.ACTION_DRAG_ENDED:
            break;
    }

    return true;
}   
```

The third parameter we passed to the `onDrag()` method is set on the DragEvent object and we get it via `event.geLocalState()`. Next we get the coordinates and update the view's location. In order to do that correctly we need to subtract half of its width/height since `setX()` and `setY()` considers the upper-left corner and our coordinates refer to the exact middle of the image. We also change the visibility in `ACTION_DRAG_STARTED` to invisible and set it back to visible in `ACTION_DROP`. 

And that's it. Since we just dragged and dropped the image on a single underlying view we don't need a lot of the cases here. The `ACTION_ENTERED` and `ACTION_EXITED` is used to inform the view that the dragged object enter/exited its bounds. Since our view occupies the entire screen we don't use it. 

Note that these cases follow a logical sequence and if you want to receive further event calls you **must** return true from this method, otherwise false signals to the framework that you don't intend to further process the drag and drop. 

![drag-and-drop](/images/2015/drag-drop.jpg)

The first is the image in the original position. The second is the image while it's being dragged and the final one is the image placed in the new position.