---
layout: post
title: Response caching using OkHttp and Retrofit (Part 1)
---

Retrofit has an excellent feature which lets developers cache responses for future use. Caching is ideally controlled by HTTP headers which are sent by the server in its response. You can read more about HTTP caching over [here](https://tools.ietf.org/html/rfc7234).

The above approach to caching is what Retrofit uses and it works pretty well if the right headers are sent by the server. In that case, you just have to define the cache location and everything is pretty much handled by the library.

Then, there are cases where the cache related headers are either incorrect or missing and you can't do anything about it because the server is not under your control. In such cases OkHttp and it's powerful interceptors come to rescue.

Interceptors allow the developers to modify API response or request. In our case this implies that we can add the required headers which control/ enforce the caching policy.

**Caching Policy**

I need the responses cached for the following two cases -

1. Usually, when the device is offline, we get an UnknownHostException and no data is retrieved. Caching allows us to access the response that was last fetched and show it to the user while the device is offline.

2. When an identical request is made within a given time frame. There are cases when a device sends multiple identical API requests to the server in a short time period. Usually, in such cases, the data does not change - wasting time and resources to get identical data from the server. Hence, we can use cache to fetch the same response that was received in the first call. It is important to consider the time period under which the cached response is to be returned instead of making a new API request.

I'm using retrofit version 2 and OkHttp version 3. It should work for other versions of OkHttp that support Interceptors but I have not tested that.

Interceptor to cache data for a minute -

    ```Java
    /**
     * Interceptor to cache data and maintain it for a minute.
     *
     * If the same network request is sent within a minute,
     * the response is retrieved from cache.
     */
    private static class ResponseCacheInterceptor implements Interceptor {
        @Override
        public okhttp3.Response intercept(Chain chain) throws IOException {
            okhttp3.Response originalResponse = chain.proceed(chain.request());
                return originalResponse.newBuilder()
                        .header("Cache-Control", "public, max-age=" + 60)
                        .build();
        }
    }
    ```

Interceptor to cache data for offline access -

    ```Java
    /**
     * Interceptor to cache data and maintain it for four weeks.
     *
     * If the device is offline, stale (at most four weeks old)
     * response is fetched from the cache.
     */
    private static class OfflineResponseCacheInterceptor implements Interceptor {
        @Override
        public okhttp3.Response intercept(Chain chain) throws IOException {
            Request request = chain.request();
            if (!UtilityMethods.isNetworkAvailable()) {
                request = request.newBuilder()
                        .header("Cache-Control",
                          "public, only-if-cached, max-stale=" + 2419200)
                        .build();
            }
            return chain.proceed(request);
        }
    }
    ```

Adding the interceptors to the OkHttp Client. We add the `ResponseCacheInterceptor` as a network interceptor and the `OfflineResponseCacheInterceptor` just as an interceptor.

    ```Java
    OkHttpClient okHttpClient = new OkHttpClient.Builder()
                    // Enable response caching
                    .addNetworkInterceptor(new ResponseCacheInterceptor())
                    .addInterceptor(new OfflineResponseCacheInterceptor())
                    // Set the cache location and size (5 MB)
                    .cache(new Cache(new File(MarsExplorer
                            .getApplicationInstance().getCacheDir(),
                            "apiResponses"), 5 * 1024 * 1024))
                    .build();
    ```

**Acknowledgement:**

There are multiple solutions available on the internet and for some people they seem to work. But for me, [this answer](https://stackoverflow.com/questions/23429046/can-retrofit-with-okhttp-use-cache-data-when-offline/36795214#36795214) was what sent me in the correct direction.
