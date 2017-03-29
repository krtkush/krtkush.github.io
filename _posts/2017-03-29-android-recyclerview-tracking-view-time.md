---
layout: post
comments: true
title: Android RecyclerView - Tracking time spent viewing items
tags: [Code, RecyclerView, Android]
---

There are multiple analytics tools for providing insightful information with respect to user behaviour, bugs and crashes on an app. One parameter that seemed to be missing in most free analytic tools is the time spent looking at a certain view in Android and then providing the raw data about it.

In this post I'll explain how to track the time a user spends looking at certain items in a RecyclerView.

We assume that any item, within the RecyclerView, with a certain minimum visibility, for a certain time on the screen is being viewed by the user. For example, suppose an item with 100% of its height and width within the screen of the device for greater than 3 seconds is being shown; we assume that that such an item has captured the users attention and hence, should be tracked.

We'll need a RecyclerView and add `OnGlobalLayoutListener` and `OnScrollListener` to it to make our tracking work.

`OnGlobalLayoutListener` is needed to detect when the RecyclerView has successfully been populated by the adapter for the first time and when the developer wants the tracking to stop. This is needed because in normal cases we'll rely on the `OnScrollListener` to track items but in the cases mentioned above, `OnScrollListener` will not be activated.

`OnScrollListener` is required for tracking all the items the user might see on scrolling down and back up.

**Defining the Data Structure**

Before we start tracking we need to define the data structure under which we'll be storing all the data. We'll make a simple POJO and call it `TrackingData`.

    public class TrackingData {

      // Duration for which the view has been viewed.
      private long viewDuration;

      // ID for the view that was viewed (we'll use the position of the item here).
      private String viewId;

      // Percentage of the height visible
      private double percentageHeightVisible;

      public double getPercentageHeightVisible() {
        return percentageHeightVisible;
      }

      public void setPercentageHeightVisible(double percentageHeightVisible) {
        this.percentageHeightVisible = percentageHeightVisible;
      }

      public long getViewDuration() {
        return viewDuration;
      }

      public void setViewDuration(long viewDuration) {
        this.viewDuration = viewDuration;
      }

      public String getViewId() {
        return viewId;
      }

      public void setViewId(String viewId) {
        this.viewId = viewId;
      }
    }

**Defining tracking logic**

We'll make a class, called `ViewTracker`, that'll accept the instance of the RecyclerView and will provide the method to start & stop the tracking and to get the tracking data.

    public class ViewTracker {

        // Time from which a particular view has been started viewing.
        private long startTime = 0;

        // Time at which a particular view has been stopped viewing.
        private long endTime = 0;

        // Flag is required because 'addOnGlobalLayoutListener'
        // is called multiple times.
        // The flag limits the action inside 'onGlobalLayout' to only once.
        private boolean firstTrackFlag = false;

        // ArrayList of view ids that are being considered for tracking.
        private ArrayList<Integer> viewsViewed = new ArrayList<>();

        // ArrayList of TrackingData class instances.
        private ArrayList<TrackingData> trackingData = new ArrayList<>();

        // The minimum amount of area of the list item that should be on
        // the screen for the tracking to start.
        private double minimumVisibleHeightThreshold;

        // Start the tracking process.
        public void startTracking() {

          // Track the views when the data is loaded into
          // recycler view for the first time.
          recyclerView.getViewTreeObserver()
                  .addOnGlobalLayoutListener(new ViewTreeObserver
                    .OnGlobalLayoutListener() {
            @Override
            public void onGlobalLayout() {

                  if(!firstTrackFlag) {

                      startTime = System.currentTimeMillis();

                      int firstVisibleItemPosition = ((LinearLayoutManager)
                            recyclerView.getLayoutManager())
                            .findFirstVisibleItemPosition();

                            int lastVisibleItemPosition = ((LinearLayoutManager)
                              recyclerView.getLayoutManager())
                              .findLastVisibleItemPosition();

                       analyzeAndAddViewData(firstVisibleItemPosition,
                         lastVisibleItemPosition);

                       firstTrackFlag = true;
                }
              }
            });

            // Track the views when user scrolls through the recyclerview.
            recyclerView.addOnScrollListener(new RecyclerView.OnScrollListener() {
              @Override
              public void onScrollStateChanged(RecyclerView recyclerView,
                int newState) {
                  super.onScrollStateChanged(recyclerView, newState);

                    // User is scrolling, calculate and store the tracking
                    // data of the views that were being viewed
                    // before the scroll.
                    if (newState == RecyclerView.SCROLL_STATE_DRAGGING) {
                        endTime = System.currentTimeMillis();

                        for (int trackedViewsCount = 0;
                             trackedViewsCount < viewsViewed.size();
                             trackedViewsCount++ ) {

                            trackingData.add(prepareTrackingData(String
                                            .valueOf(viewsViewed
                                              .get(trackedViewsCount)),
                                    (endTime - startTime)/1000));
                        }

                        // We clear the list of current item positions.
                        // If we don't do this, the items will be tracked
                        // every time the new items are added.   
                        viewsViewed.clear();
                    }

                    // Scrolling has ended, start the tracking
                    // process by assigning a start time
                    // and maintaining a list of views being viewed.
                    if (newState == RecyclerView.SCROLL_STATE_IDLE) {

                        startTime = System.currentTimeMillis();

                        int firstVisibleItemPosition = ((LinearLayoutManager)
                                recyclerView.getLayoutManager())
                                .findFirstVisibleItemPosition();

                        int lastVisibleItemPosition = ((LinearLayoutManager)
                                recyclerView.getLayoutManager())
                                .findLastVisibleItemPosition();

                        analyzeAndAddViewData(firstVisibleItemPosition,
                          lastVisibleItemPosition);
                    }
                }
          });
    }

    // Track the items currently visible and then stop the tracking process.
    public void stopTracking() {

            endTime = System.currentTimeMillis();

            int firstVisibleItemPosition = ((LinearLayoutManager)
                    recyclerView.getLayoutManager()).findFirstVisibleItemPosition();

            int lastVisibleItemPosition = ((LinearLayoutManager)
                    recyclerView.getLayoutManager()).findLastVisibleItemPosition();

            analyzeAndAddViewData(firstVisibleItemPosition,
              lastVisibleItemPosition);

            for (int trackedViewsCount = 0; trackedViewsCount < viewsViewed.size();
                 trackedViewsCount++ ) {

                trackingData.add(prepareTrackingData(String.valueOf(viewsViewed
                                .get(trackedViewsCount)),
                                (endTime - startTime)/1000));

            viewsViewed.clear();
        }
    }

    private void analyzeAndAddViewData(int firstVisibleItemPosition,
      int lastVisibleItemPosition) {

        // Analyze all the views
        for (int viewPosition = firstVisibleItemPosition;
             viewPosition <= lastVisibleItemPosition; viewPosition++) {

            Log.i("View being considered", String.valueOf(viewPosition));

            // Get the view from its position.
            View itemView = recyclerView.getLayoutManager()
                    .findViewByPosition(viewPosition);

            // Check if the visibility of the view is more than or equal
            // to the threshold provided. If it falls under the desired limit,
            // add it to the tracking data.
            if (
              getVisibleHeightPercentage(itemView) >=
                minimumVisibleHeightThreshold) {
                viewsViewed.add(viewPosition);
            }
        }
      }

      // Method to calculate how much of the view is visible
      // (i.e. within the screen) wrt the view height.
      // @param view
      // @return Percentage of the height visible.
      private double getVisibleHeightPercentage(View view) {

          Rect itemRect = new Rect();
          view.getLocalVisibleRect(itemRect);

          // Find the height of the item.
          double visibleHeight = itemRect.height();
          double height = view.getMeasuredHeight();

          Log.i("Visible Height", String.valueOf(visibleHeight));
          Log.i("Measured Height", String.valueOf(height));

          double viewVisibleHeightPercentage = ((visibleHeight/height) * 100);

          Log.i("Percentage visible", String.valueOf(viewVisibleHeightPercentage));

          Log.i("___", "___");

          return viewVisibleHeightPercentage;
      }

      // Method to store the tracking data in an instance of "TrackingData" and
      // then returning that instance.
      // @param viewId
      // @param viewDuration in seconds.
      private TrackingData prepareTrackingData(String viewId, long viewDuration) {

          TrackingData trackingData = new TrackingData();

          trackingData.setViewId(viewId);
          trackingData.setViewDuration(viewDuration);

          return trackingData;
      }
    }    

**(a) Finding the items that are visible on the screen**

The way we go about finding the items that are visible to the user is by using `findFirstVisibleItemPosition()` and `findLastVisibleItemPosition()` methods provided by the Android SDK. These methods return the first and last items partially or completely visible. Now, by knowing the positions of these two items, we can find all the item between them and conclude that those items are visible too.

Do note, the method `findLastVisibleItemPosition()` can return the item right below the last visible item because technically, it fetches the last attached item in the RecyclerView.

To the above two methods, it does not matter whether the item is partially visible or completely. We could have substituted them with `findFirstCompletelyVisibleItemPosition` and `findLastCompletelyVisibleItemPosition` respectively had we wanted to ignore the partially visible views. But, in our case it does not make any difference, we can use any of those methods.

**(b) Figuring out the visible percentage of a view (w.r.t. height)**

To filter out the partially visible views we have `getVisibleHeightPercentage()` method that measures the percentage of a view actually on the screen (in terms of height). We can set a threshold (of, for example, 40%) and ignore any view whose visibility is below that point.

The final data is stored in an ArrayList of `TrackingData`.

When it comes to tracking user behaviour on a RecyclerView, we can lot more functionality - like tracking taps on views inside the list item or on the item itself. Though, all this is beyond the scope of this article.
