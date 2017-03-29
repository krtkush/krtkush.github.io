---
layout: post
comments: true
title: Response caching using OkHttp (Part 2)
tags: [Code, Caching, OkHttp, Android]
---

**This post was featured in [AndroidDev Digest's #136 issue](https://www.androiddevdigest.com/digest136/).**

In the [previous post](https://krtkush.github.io/2016/06/01/caching-using-okhttp-part-1.html) I explained how we can use Retrofit and OkHttp to cache responses for various use cases.

The apparent problem with the linked approach is that *every* API request will have its Cache-Headers modified by the interceptors. There is no way to control API requests individually. To solve this problem I have come with a, dare I say, elegant hack - We add some custom headers while sending the request, whose value the interceptors will check and act accordingly.

Continuing with the interceptors we created in the last post, we'll modify them to check for boolean flag which specifies the developer's choice to use the caching functionality or not.

**Accepting headers dynamically in the API request**



    ```Java
    @GET("{someName}/getSomething")
    Observable<PhotoSearchResultDM> getPhotosBySol(
        @Header("ApplyOfflineCache") boolean offlineCache,
        @Header("ApplyResponseCache") boolean responseCache,
        @Path("arg1") String arg1,
        @Query("arg2") String arg2);
    ```

  Here, the two headers, `ApplyOfflineCache` and `ApplyResponseCache` accept boolean flags which convey to the interceptors whether to apply their respective caching policy or not.

**Reading the custom headers and their respective values**



    ```Java
    /**
     * Interceptor to cache data and maintain it for a minute.
     *
     * If the same network request is sent within a minute,
     * the response is retrieved from cache.
     */
    private static class ResponseCacheInterceptor implements Interceptor {
        @Override
        public Response intercept(Chain chain) throws IOException {

            Request request = chain.request();
            if(Boolean.valueOf(request.header("ApplyResponseCache"))) {
                Timber.i("Response cache applied");
                Response originalResponse = chain.proceed(chain.request());
                return originalResponse.newBuilder()
                        .removeHeader("ApplyResponseCache")
                        .header("Cache-Control", "public, max-age=" + 60)
                        .build();
            } else {
                Timber.i("Response cache not applied");
                return chain.proceed(chain.request());
            }
        }
    }
    ```


Similarly,


    ```Java
    /**
     * Interceptor to cache data and maintain it for four weeks.
     *
     * If the device is offline, stale (at most four weeks old)
     * response is fetched from the cache.
     */
    private static class OfflineResponseCacheInterceptor implements Interceptor {
        @Override
        public Response intercept(Chain chain) throws IOException {

            Request request = chain.request();
            if(Boolean.valueOf(request.header("ApplyOfflineCache"))) {
                Timber.i("Offline cache applied");
                if(!UtilityMethods.isNetworkAvailable()) {
                    request = request.newBuilder()
                            .removeHeader("ApplyOfflineCache")
                            .header("Cache-Control",
                                    "public, only-if-cached,
                                    max-stale=" + 2419200)
                            .build();
                }
            } else
                Timber.i("Offline cache not applied");

            return chain.proceed(request);
        }
    }
    ```

We remove the headers from the request after using them. It is important that we do not use any predefined headers for our own custom use cases.
