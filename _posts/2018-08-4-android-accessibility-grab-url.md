---
layout: post
comments: true
title: Android Accessibility - Grabbing URLs from a browser
tags: [Code, Android]
---
    
Android's accessibility service is one of the most powerful tools offered by the operating system in terms of user's data collection.

In this post I'll explain how to use the accessibility service to grab URLs off from a browser's URL bar. Or, for that matter almost any information from an app running in foreground.

Before I begin I should put some related links wrt to Accessibility service. These links contain information which I think one should be aware of before deploying an app, on the Play Store, which uses this service. 

1. [Security issues with Android Accessibility](https://android.jlelse.eu/android-accessibility-75fdc5810025)
2. [An article in response to the first one](https://www.androidpolice.com/2017/11/12/google-will-remove-play-store-apps-use-accessibility-services-anything-except-helping-disabled-users/)
3. [Reddit thread on potential app ban on using Accessibility in way not intended](https://www.reddit.com/r/Android/comments/7c4go5/is_google_play_really_going_to_suspend_all_apps/?st=jhrex6in&sh=e1a1f7fd)
4. [Reddit thread on Google (partially) reversing its decision about the ban](https://www.reddit.com/r/Android/comments/7i4mlm/google_pausing_the_30_day_notice_on_wrong/?st=jkfd2mas&sh=816a4410)

Now that we have that out of the way let's look at the code. 

      public class ASUrl extends AccessibilityService {

          @Override
          public void onAccessibilityEvent(AccessibilityEvent event) {

            AccessibilityNodeInfo source = event.getSource();

            if (source == null)
            return;

            final String packageName = String.valueOf(source.getPackageName());

            // Add browser package list here (comma seperated values)
            String BROWSER_LIST = "";

            List<String> browserList = Arrays.asList(BROWSER_LIST.split(",\\s*"));
            if (event.getEventType() 
              == AccessibilityEvent.TYPE_WINDOW_CONTENT_CHANGED) {
              if (!browserList.contains(packageName)) {
                return;
              }
            }

            if (browserList.contains(packageName)) {
              try {
                // App opened is a browser.
                // Parse urls in browser.
                if (AccessibilityEvent
                  .eventTypeToString(event.getEventType())
                  .contains("WINDOW")) {
                  AccessibilityNodeInfo nodeInfo = event.getSource();
                  getUrlsFromViews(nodeInfo);
                }
              } catch(StackOverflowError ex){
                ex.printStackTrace();
              } catch (Exception ex) {
                ex.printStackTrace();
              }
            } 
          }

          /**
          * Method to loop through all the views and try to find a URL.
          * @param info
          */
          public void getUrlsFromViews(AccessibilityNodeInfo info) {  

            try {
              if (info == null) 
                return;

              if (info.getText() != null && info.getText().length() > 0) {

                String capturedText = info.getText().toString();

                    if (capturedText.contains("https://") 
                      || capturedText.contains("http://")) {
                      if (!previousUrl.equals(capturedText)) {
                        // Do something with the url.
                        previousUrl = capturedText;
                      }
                    }
              }

              for (int i = 0; i < info.getChildCount(); i++) {
                AccessibilityNodeInfo child = info.getChild(i);
                getUrlsFromViews(child);
                if(child != null){
                  child.recycle();
                }
              }
            } catch(StackOverflowError ex){
              ex.printStackTrace();
            } catch (Exception ex) {
              ex.printStackTrace();
            }
          }

          @Override
          public void onInterrupt() {

          }
      }


In the above implementation the service `ASUrl` is called every time an app is opened (You can configure the service to start for certain apps only). Once the service is invoked the `onAccessibilityEvent()` method is called and this is where we check if the app opened is a browser of our choice or not. If it is a browser we call the `getUrlsFromViews()` method. This method loops through all the available views of the app, gets any string value in them and checks if that string is a URL or not. Also, if a certain URL is found more than once continuously, we ignore the second occurrence.


This is the basic of grabbing any kind of data from a app's window using Accessibility Service.


