# RecycleView-auto-play-video

使用的视频播放框架 https://github.com/lipangit/JiaoZiVideoPlayer/issues


##原理

``` xml

1.添加OnScrollListener 监听 ，循环遍历 可见区域（就是 lastVisibleItem - firstVisibleItem的个数）
内的播放器控件
最后通过 getLocalVisibleRect（rect） 方法计算出哪个播放器完全显示出来，然后startVideo（）进行播放

2.在适当的时机跳出循环，只处理可见区域内的第一个播放器
```
 [getLocalVisibleRect原理可以参照这个](http://www.cnblogs.com/ai-developers/p/4413585.html)

##可能存在的缺点：
``` xml
1.对于RecyleView 视频资源回收的时机，
   回收时应该调用什么函数
```

使用的JiaoZiVideoPlayer版本
``` xml
    compile 'cn.jzvd:jiaozivideoplayer:6.2.7'
```
自己项目中的sdk版本
``` xml
 compileSdkVersion 23
 buildToolsVersion '25.0.3'
 minSdkVersion 16
 targetSdkVersion 23
```
自己项目中RecycleView实现视频自动播放的代码如下
``` java
      FeedScrollListener listener = new FeedScrollListener();
      recyclerView.addOnScrollListener(listener);
```
``` java
    private class FeedScrollListener extends RecyclerView.OnScrollListener{

        private int firstVisibleItem = 0;
        private int lastVisibleItem = 0;
        private int visibleCount = 0;

        /**
         * 视频状态标签
         */
        private enum VideoTagEnum {

            /**
             * 自动播放视频
             */
            TAG_AUTO_PLAY_VIDEO,

            /**
             * 暂停视频
             */
            TAG_PAUSE_VIDEO
        }

        @Override
        public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
           // 根据自己需要，来决定是否需要调用 super
           // super.onScrollStateChanged(recyclerView, newState);

            // 自己定义的 wifi 下自动播放视频 boolean 开关
            if (setPrefs_.wifiAutoPlayVideo().get()) {
                switch (newState) {
                    case RecyclerView.SCROLL_STATE_IDLE:
                        autoPlayVideo(recyclerView, VideoTagEnum.TAG_AUTO_PLAY_VIDEO);
                    default:
                        autoPlayVideo(recyclerView, VideoTagEnum.TAG_PAUSE_VIDEO);
                        break;
                }
            } else {
                JZVideoPlayer.releaseAllVideos();
            }
        }

        @Override
        public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
          // 根据自己需要，来决定是否需要调用 super
        //    super.onScrolled(recyclerView, dx, dy);

            RecyclerView.LayoutManager layoutManager = recyclerView.getLayoutManager();

            if (layoutManager instanceof LinearLayoutManager) {

                RecycleViewFeedAdapter adapter = (RecycleViewFeedAdapter) recyclerView.getAdapter();

                LinearLayoutManager linearManager = (LinearLayoutManager) layoutManager;

                firstVisibleItem = linearManager.findFirstVisibleItemPosition();
                lastVisibleItem = linearManager.findLastVisibleItemPosition();
                visibleCount = lastVisibleItem - firstVisibleItem + adapter.getHeaderLayoutCount();// 这加上了Header的个数
            }

        }

        /**
         * 循环遍历 可见区域的播放器
         * 然后通过 getLocalVisibleRect(rect)方法计算出哪个播放器完全显示出来
         * <p>
         * getLocalVisibleRect相关链接：http://www.cnblogs.com/ai-developers/p/4413585.html
         *
         * @param view
         * @param handleVideoTag 视频需要进行状态 
         */
        private void autoPlayVideo(RecyclerView view, VideoTagEnum handleVideoTag) {
            for (int i = 0; i < visibleCount; i++) {
                if (view != null && view.getChildAt(i) != null && view.getChildAt(i).findViewById(R.id.feed_video_player) != null) {
                    JZVideoPlayerStandard homeGSYVideoPlayer = (JZVideoPlayerStandard) view.getChildAt(i).findViewById(R.id.feed_video_player);

                    Rect rect = new Rect();
                    homeGSYVideoPlayer.getLocalVisibleRect(rect);
                    int videoheight = homeGSYVideoPlayer.getHeight();
                    if (rect.top == 0 && rect.bottom == videoheight) {
                        handleVideo(handleVideoTag, homeGSYVideoPlayer);
                        // 借助跳出循环，达到只处理可见区域内的第一个播放器
                        break ;
                    }
                }
            }

        }

        /**
         * 视频状态处理
         * @param handleVideoTag 视频需要进行状态
         * @param homeGSYVideoPlayer JZVideoPlayer播放器
         */
        private void handleVideo(VideoTagEnum handleVideoTag, JZVideoPlayerStandard homeGSYVideoPlayer) {
            switch (handleVideoTag) {
                case TAG_AUTO_PLAY_VIDEO:
                    if ((homeGSYVideoPlayer.currentState != JZVideoPlayerStandard.CURRENT_STATE_PLAYING)) {
                        // 进行播放
                        homeGSYVideoPlayer.startVideo();
                    }
                    break;
                case TAG_PAUSE_VIDEO:
                    if ((homeGSYVideoPlayer.currentState != JZVideoPlayerStandard.CURRENT_STATE_PAUSE)) {
                        // 模拟点击播放Button,实现暂停视频
                        homeGSYVideoPlayer.startButton.performClick();
                    }
                    break;
                default:
                    break;
            }
        }


    }
```


