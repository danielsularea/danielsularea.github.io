---
title: How to hide the status bar and navigation bar in android
layout: post
---

This article applies to API >= 16. When something requires a different API level it will be highlighted. 

In some cases you want your activity to be full screen and that means hiding some system elements - the status bar and the navigation bar. One way of doing it is to call the method `setSystemUiVisibility()` on the decor view. Remember that the decor view is the topmost view and you can get a reference to it by calling `getWindow().getDecorView()`. 

The `setSystemUiVisibility()` takes an `int` as an argument, which you will build by doing a [bitwise or](https://en.wikipedia.org/wiki/Bitwise_operation#OR) operation on a few constants defined by the framework. The constants of interest are:

* `View.SYSTEM_UI_FLAG_FULLSCREEN` hides the status bar.
* `View.SYSTEM_UI_FLAG_HIDE_NAVIGATION` hides the navigation bar

The method call then looks like this:  

```language-java-line-numbers
decorView = getWindow().getDecorView();
decorView.setSystemUiVisibility(
    View.SYSTEM_UI_FLAG_HIDE_NAVIGATION | 
    View.SYSTEM_UI_FLAG_FULLSCREEN);
```

It's important **where** you call this method because when the visibility of your activity changes, these flags will reset. Calling it in `onCreate()` and then pressing on the Home button and navigating back to your activity will show the navigation and status bars. Instead if you place it in `onResume()` they will remain hidden because `onResume()` will be called again when the activity starts. See the [activity lifecycle](http://developer.android.com/reference/android/app/Activity.html#ActivityLifecycle) for more info.  

It's best practice to also hide the ActionBar when hiding the system bars and you can do so by calling `getActionBar().hide()` and `getActionBar().show()` respectively. If you are using the support action bar then replace the above call with `getSupportActionBar()`. 

The above flags however will get reset on touch events. To manage this and to allow your activity to manage touch events while keeping the status and navigation bars hidden, from API 19 onwards there's a new flag called **immersive**. 

* `View.SYSTEM_UI_FLAG_IMMERSIVE` will reset only when you swipe downwards from where the system bar is (or swipe upwards from where the navigation bar is). 

* `View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY` doesn't reset the flags. It just briefly shows the system bar and navigation bar then goes back to being hidden. 

Here's an example of before/after using sticky mode:

![immersive](/images/2015/immersive.jpg)

This has to be combined with either `View.SYSTEM_UI_FLAG_HIDE_NAVIGATION` or `View.SYSTEM_UI_FLAG_FULLSCREEN` (normally both) because the entire point of immersive is to be able to intercept events without resetting all the flags, and that is what this flag does. 

You can also attach a listener to be informed of the system UI visibility changes (like when you swipe to show the status bar or navigation bar). 

Here's how your `MainActivity` will look:

```language-java-line-numbers
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        View decorView = getWindow().getDecorView();
        decorView.setOnSystemUiVisibilityChangeListener(new View.OnSystemUiVisibilityChangeListener() {
            @Override
            public void onSystemUiVisibilityChange(int visibility) {
                if (visibility == View.VISIBLE) {
                    // system ui is vibisle
                } else {
                  // system ui are not visible
                }
            }
        });
    }

    @Override
    protected void onResume() {
        super.onResume();

        getWindow().getDecorView().setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_HIDE_NAVIGATION |
                View.SYSTEM_UI_FLAG_FULLSCREEN |
                View.SYSTEM_UI_FLAG_IMMERSIVE
        );
        getSupportActionBar().hide();
    }

```

Notice the immersive flag being **or**-ed with the other two. At this point your activity will display on full screen and when you swipe down the status and navigation bars will show, calling the listener. 

If you use the **sticky** immersive flag then the listener **won't** be called and status and navigation bars will only show for a brief moment. 

Below is a working example with a toggle button which allows you to hide or show the system bars.

`activity.xml` layout:

```language-html-line-numbers
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ToggleButton
        android:id="@+id/toggle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerVertical="true"
        android:layout_centerHorizontal="true" />

</RelativeLayout>

```

`MainActivity` file:

```language-java-line-numbers

    ToggleButton toggle;
    private boolean isHidden;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        toggle = (ToggleButton) findViewById(R.id.toggle);
        toggle.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (isHidden) showSystemUi();
                else hideSystemUi();
            }
        });
    }

    @Override
    protected void onResume() {
        super.onResume();

        hideSystemUi();
    }

    private void hideSystemUi() {
        getWindow().getDecorView().setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_HIDE_NAVIGATION |
                        View.SYSTEM_UI_FLAG_FULLSCREEN |
                        View.SYSTEM_UI_FLAG_IMMERSIVE |
        );
        getSupportActionBar().hide();
        isHidden = true;
        toggle.setText("Hide");
    }

    private void showSystemUi() {
        getWindow().getDecorView().setSystemUiVisibility(
                View.SYSTEM_UI_FLAG_VISIBLE
        );

        getSupportActionBar().show();
        isHidden = false;
        toggle.setText("Show");
    }

```

This will result in the following

![hide_show](/images/2015/hide_show.png)

The toggle button will hide or show the system bars resizing the content accordingly to the available layout. 

If you don't want the content to be resized there are additional flags which you can pass to `setSystemUiVisibility`

* `SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN` sets the layout as if the status bar is hidden even when it's not.

* `SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION` sets the layout as if the navigation bar is hidden even when it's not. 

When the status and navigation bars are shown they will overlap over your content.