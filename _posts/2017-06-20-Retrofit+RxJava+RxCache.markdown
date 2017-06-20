---
layout:     post
title:      "RxJava2+Retrofit2+RxCache实现缓存"
subtitle:   " \"简洁的博客带给人好心情\""
date:       2017-06-20 18:33:00
author:     "Daisy"
header-img: "img/post-bg-rxcache.jpg"
catalog: true
tags:
    - Rx
---

> “have a good start. ”

## RxJava2+Retrofit2+RxCache实现缓存  ##
> 一个原生的Android应用如果使用缓存可以一定程度上提升用户体验，但并不是每个应用都应用了缓存，我们今天整合一个Retrofit2+RxJava2+RxCache的Demo.

### 写在前面 ###
Retrofit算是现在Android网络请求的头号选择，RxJava更是因为可以与Retrofit无缝链接以及他自身的异步操作的特点被人们推崇，所以如果你还没有学习过，推荐尽快入手.<br>

### 导入库  版本号请注意 ###
    // rxandroid
    compile 'io.reactivex.rxjava2:rxandroid:2.0.1'
    // rxjava2
    compile 'io.reactivex.rxjava2:rxjava:2.1.0'
    // retrofit
    compile 'com.squareup.retrofit2:retrofit:2.2.0'
    // gson-converter
    compile 'com.squareup.retrofit2:converter-gson:2.2.0'
    // rxjava-adapter
    compile 'com.squareup.retrofit2:adapter-rxjava2:2.2.0'
    // rxcache
    compile 'com.github.VictorAlbertos.RxCache:runtime:1.8.0-2.x'
    compile 'com.github.VictorAlbertos.Jolyglot:gson:0.0.3'
    testCompile 'junit:junit:4.12'
    // okhttp 日志拦截器
    compile 'com.squareup.okhttp3:logging-interceptor:3.6.0'

### Retrofit接口 ###
    public interface RxJavaService {
	    /**
	     * 获取所有储卡/球券信息 
	     *
	     * @return
	     */
	    @FormUrlEncoded
	    @POST("rest/cardAndCoupons/getCardAndCoupons")
	    Observable<ParentBean<BallCardItemBean>> getCardAndCoupons(@Field("data") String data);
	}

### RxCache接口 ###
    public interface CacheProviders {
	    @LifeCache(duration = 2, timeUnit = TimeUnit.MINUTES)
	    Observable<Reply<List<User>>> getUsers(Observable<List<User>> oUsers, DynamicKey idLastUserQueried, EvictProvider evictProvider);
	
	    /**
	     * @param data          retrofit 返回的Observable
	     * @param fieldData     参数  组合方法名为key
	     * @param evictProvider 是否强制刷新
	     * @return
	     */
	    @LifeCache(duration = 5, timeUnit = TimeUnit.MINUTES)
	    Observable<ParentBean<BallCardItemBean>> getCardAndCoupons(Observable<ParentBean<BallCardItemBean>> data, DynamicKey fieldData, EvictProvider evictProvider);
	}

### 自定义Application提供 Repository 单例 ###
    public class SampleAndroidApp extends Application {

	    private Repository repository;
	
	    @Override
	    public void onCreate() {
	        super.onCreate();
	        repository = Repository.init(getCacheDir());
	    }
	
	    public Repository getRepository() {
	        return repository;
	    }
	}

### Repository代码 ###
    public class Repository {

	    private final String TAG = "zhuhu";
	
	    public static Repository init(File cacheDir) {
	        return new Repository(cacheDir);
	    }
	
	    private final CacheProviders cacheProviders;
	    private final RxJavaService rxjavaApi;
	
	    private OkHttpClient okHttpClient; // 配置自己的Okhttp，可配置过期时间，错误是否重试，日志拦截输出
	
	    private HttpLoggingInterceptor.Level level = HttpLoggingInterceptor.Level.BODY;
	    private HttpLoggingInterceptor loggingInterceptor = new HttpLoggingInterceptor(new HttpLoggingInterceptor.Logger() {
	        @Override
	        public void log(String message) {
	            LogUtils.logI(TAG, "message:" + message);
	        }
	    });
	
	    public Repository(File cacheDir) {
	
	        loggingInterceptor.setLevel(level);
	
	        okHttpClient = new OkHttpClient()
	                .newBuilder()
	                .addInterceptor(new MyInterceptor()) // 自定义拦截器 添加参数
	                .addInterceptor(loggingInterceptor) //  日志拦截器
	                .connectTimeout(5, TimeUnit.SECONDS) // 如果超时，会调用Retrofit 的onFailure函数，并且Throable的报错信息为TimeOut
	                .build();
	
	        cacheProviders = new RxCache.Builder()
	                .persistence(cacheDir, new GsonSpeaker())
	                .using(CacheProviders.class);
	
	        rxjavaApi = new Retrofit.Builder().baseUrl(Api.HTTP_BALL_SERVER_PREFIX)
	                .addConverterFactory(GsonConverterFactory.create())
	                .client(okHttpClient)
	                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
	                .build()
	                .create(RxJavaService.class);
	    }
	
	    public Observable<ParentBean<BallCardItemBean>> getCardAndCoupons(String fieldData, final boolean update) {
	        return cacheProviders.getCardAndCoupons(rxjavaApi.getCardAndCoupons(fieldData), new DynamicKey(fieldData), new EvictDynamicKey(update));
	
	    }
	}

### CacheActivity具体调用方法 ###
     private void btn_rxjava_getCardAndCoupons() {
	        tvInfo.setText("");
	        Map<String, String> map = new HashMap<>();
	        map.put("page", "1");
	        map.put("rows", "10");
	        map.put("type", "1");
	        Gson mGson = new Gson();
	        Observable<ParentBean<BallCardItemBean>> cardAndCoupons = ((SampleAndroidApp) getApplicationContext()).getRepository().getCardAndCoupons(mGson.toJson(map), isPullToRefresh);
	        cardAndCoupons.subscribeOn(Schedulers.io())
	                .observeOn(AndroidSchedulers.mainThread())
	                .subscribe(new Observer<ParentBean<BallCardItemBean>>() {
	                    @Override
	                    public void onSubscribe(@NonNull Disposable d) {
	                        LogUtils.logI("zhuhu", "onSubscribe");
	                    }
	
	                    @Override
	                    public void onNext(@NonNull ParentBean<BallCardItemBean> ballCardItemBeanParentBean) {
	                        LogUtils.logI("zhuhu", "ballCardItemBeanParentBean:" + ballCardItemBeanParentBean.toString());
	                        tvInfo.append(ballCardItemBeanParentBean.toString() + "\n");
	                    }
	
	                    @Override
	                    public void onError(@NonNull Throwable e) {
	                        LogUtils.logI("zhuhu", "onError");
	                        e.printStackTrace();
	                        tvInfo.append(e.getMessage() + "\n");
	                    }
	
	                    @Override
	                    public void onComplete() {
	                        LogUtils.logI("zhuhu", "onComplete");
	                    }
	                });
	    }

> 这里使用map封装参数是因为服务器需要一个json对象字符串，可自行改变


----------

**说一下实验结果。<br>**
1.如果你的代码编写正确，可以在手机的/Android/data/package-name/cache/文件夹下找到缓存的文件，命名方式为你的方法名+dynamicKey,我的缓存文件的名字为"getCardAndCoupons$d$d$d$1$g$g$g$"，后面乱码应该是因为转换了编码<br>
2.当你缓存一次之后再次调用不会再去网络请求，如何判断是不是从网络请求？Android Studio的Monitors控制面板可以看到<br>
![](http://i.imgur.com/4QqrfoM.png)
并且，你可以看到当你请求一次之后再次点击网络请求是不会再去请求网络的，如果你发现还是会再次请求，请检查你的代码<br>
3.缓存过期时间，RxCache的使用其实很简单，他的过人之处在于可配置过期时间，也就是@LifeCache注解，我测试的Demo代码过期时间为5分钟，测验结果为到达5分钟立刻缓存没有了，显示网络错误，后期我们可以看一下RxCache的源码。











