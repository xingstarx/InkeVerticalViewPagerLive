# InkeVerticalViewPagerLive
仿映客viewPager上下滑动切换直播demo

##简述
经过几天对映客APP的分析研究，明白了映客上下滑动切换新的直播的实现原理，真心觉得做的挺赞的。。还是很流畅的，性能也很不错。
##思路

首先是一个主Activity，里面只有一个ViewPager，当然了这个viewPager得是VerticalViewPager的。可以参考这两个项目，https://github.com/kaelaela/VerticalViewPager，
https://github.com/castorflex/VerticalViewPager， 请记住ViewPager使用的adapter得是PagerAdapter，而不能是FragmentPagerAdapter

```java
class PagerAdapter extends android.support.v4.view.PagerAdapter {

        @Override
        public int getCount() {
            return mVideoUrls.size();
        }

        @Override
        public boolean isViewFromObject(View view, Object object) {
            return view == object;
        }

        @Override
        public Object instantiateItem(ViewGroup container, int position) {
            View view = LayoutInflater.from(container.getContext()).inflate(R.layout.view_room_item, null);
            view.setId(position);
            container.addView(view);
            return view;
        }

        @Override
        public void destroyItem(ViewGroup container, int position, Object object) {
            container.removeView(container.findViewById(position));
        }
    }
```


大致上就是这样的。 view_room_item.xml的布局，其实很简单，就是一个临时的布局，而不是我们真实的布局，贴一下布局代码吧
```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ImageView
        android:id="@+id/anchor_img"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@drawable/bg_room_change"
        android:scaleType="fitXY" />
</FrameLayout>
```

至于我们的真实业务的布局在哪呢，我们接下来会说道。viewPager上下切换中，创建的都是这个假的临时的布局，而且在整个viewPager起作用的过程中，始终就只有这一个真实业务的布局，viewPager默认会加载三个view，另外两个view都是假的，或者说是临时的view，而在这里，真实业务的布局就是我们的聊天室，以及texttureview

view_room_container.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/room_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.pili.pldroid.player.widget.PLVideoTextureView
        android:id="@+id/texture_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

    <FrameLayout
        android:id="@+id/fragment_container"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</RelativeLayout>
```
我们是在什么地方把真实业务布局添加到viewPager中呢，可以看下面的一段代码
```java
mViewPager.setOnPageChangeListener(new ViewPager.SimpleOnPageChangeListener() {
            @Override
            public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
                Log.e(TAG, "mCurrentId == " + position + ", positionOffset == " + positionOffset +
                        ", positionOffsetPixels == " + positionOffsetPixels);
                mCurrentItem = position;
            }
        });

mViewPager.setPageTransformer(false, new ViewPager.PageTransformer() {
            @Override
            public void transformPage(View page, float position) {
                ViewGroup viewGroup = (ViewGroup) page;
                Log.e(TAG, "page.id == " + page.getId() + ", position == " + position);

                if ((position < 0 && viewGroup.getId() != mCurrentItem)) {
                    View roomContainer = viewGroup.findViewById(R.id.room_container);
                    if (roomContainer != null && roomContainer.getParent() != null && roomContainer.getParent() instanceof ViewGroup) {
                        ((ViewGroup) (roomContainer.getParent())).removeView(roomContainer);
                    }
                }
                // 满足此种条件，表明需要加载直播视频，以及聊天室了
                if (viewGroup.getId() == mCurrentItem && position == 0 && mCurrentItem != mRoomId) {
                    if (mRoomContainer.getParent() != null && mRoomContainer.getParent() instanceof ViewGroup) {
                        ((ViewGroup) (mRoomContainer.getParent())).removeView(mRoomContainer);
                    }
                    loadVideoAndChatRoom(viewGroup, mCurrentItem);
                }
            }
         });
```

在PageTransformer里面，我们需要判断，每时每刻都只有一个真实业务的view，具体的判断条件可以参考这个demo，也可以自己做实验印证，主要的思路还是当滑动停止下来后，transformPage()方法中，当前选中的view的position==0，这个时候我们是需要加载直播视频和聊天室的，
而view不是我们选中的view的时候，也就是`position < 0 && viewGroup.getId() == mCurrentItem` 移除之前的真实业务布局。具体可以看上面的代码


大致原理就是如此了，请运行demo吧，有疑问，请提issue。

##Thanks


