---
title: 浅谈 mvc 和 mvp 在安卓的应用
date: 2018-01-21 21:21:10
categories: android
tags: 框架原理
---

## MVC框架模式

MVC全称是Model - View - Controller，是模型(model)－视图(view)－控制器(controller)的缩写。
MVC是一种框架模式而非设计模式，GOF把MVC看作是3种设计模式：观察者模式、策略模式与组合模式的合体，而核心是观察者模式。
简而言之，框架是大智慧，用来对软件设计进行分工；设计模式是小技巧，对具体问题提出解决方案，以提高代码复用率，降低耦合度。

<!-- more -->

在Android项目中，业务逻辑，数据处理等担任了Model（模型）角色，XML界面显示等担任了View（视图）角色，Activity担任了Contronller（控制器）角色。
contronller（控制器）是一个中间桥梁的作用，通过接口通信来协同 View（视图）和Model（模型）工作，起到了两者之间的通信作用

``` bash
public class MainActivity extends Activity implements StrUIDataListener {  
    private StrVolleyInterface networkHelper;  
    private StrVolleyInterface expertNetworkHelper;  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        }  
    //加载网络数据  
    private void initDate() {  
        try {  
            networkHelper = new StrVolleyInterface(DemandDetailActivity.this);  
            networkHelper.setStrUIDataListener(DemandDetailActivity.this);  
            ApiClient.getDataDetail(DemandDetailActivity.this, id, networkHelper);  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
    }  
    @Override  
    public void onDataChanged(String data) {  
        //数据加载成功  
        //....更新UI  
    }  
    @Override  
    public void onErrorHappened(VolleyError error) {  
        Toast.makeText(DemandDetailActivity.this, "加载错误，请检查网络！", Toast.LENGTH_SHORT).show();  
    }  
}  
从上面代码可以看到，Activity持有了ApiClient网络请求模型的对象，当我们需要获取数据时，只需要调用initData（）方法。
比如我们点击Button，Activity作为Controller控制层会处理View视图层，并调用ApiClient.getDataDetail方法，当Model模型处理数据结束后，
通过接口onDataChanged通知View视图层数据处理完毕，View视图层该更新界面UI了。然后View视图层去更新界面。
看到这里，我们会发现整个MVC框架流程就在Activity中体现出来了。
```

## mvp框架模式

MVP是模型（Model）、视图（View）、控制者（Presenter）的缩写，分别代表项目中3个不同的模块。

	模型（Model）：负责处理数据的加载或者存储，比如从网络或本地数据库获取数据等；
	视图（View）：负责界面数据的展示，与用户进行交互；
	控制者（Presenter）：相当于协调者，是模型与视图之间的桥梁，将模型与视图分离开来

 在Andorid项目中，我们习惯将Activity作为MVC中的控制者来达到Model模型和View视图分离，但是在MVP框架模式中，通常将Activity作为View视图层，
 因为在MVC框架模式中Activity和View视图显示关联紧密，Activity中包含大量的View视图显示代码，如果哪天老板说需要修改View视图显示，
 这时候你是不是感觉需要修改Activity中的大量代码？这么一来会将Activity中控制逻辑破坏，也导致Activity中承担太多的职责。
 根据单一职责原则，Activity主要起到用户交互作用，也就是接收用户输入，显示请求结果。因此可以通过MVP框架模式来减轻Activity的职责
 
```bash
// model
/** 
 * 天气Model接口 
 */  
public interface WeatherModel {  
    void loadWeather(String cityNO, OnWeatherListener listener);  
}  
.........  
/** 
 * 天气Model实现 
 */  
public class WeatherModelImpl implements WeatherModel {  
    @Override  
    public void loadWeather(String cityNO, final OnWeatherListener listener) {  
        /*数据层操作*/  
        VolleyRequest.newInstance().newGsonRequest("http://www.weather.com.cn/data/sk/" + cityNO + ".html",  
                Weather.class, new Response.Listener<weather>() {  
                    @Override  
                    public void onResponse(Weather weather) {  
                        if (weather != null) {  
                            listener.onSuccess(weather);  
                        } else {  
                            listener.onError();  
                        }  
                    }  
                }, new Response.ErrorListener() {  
                    @Override  
                    public void onErrorResponse(VolleyError error) {  
                        listener.onError();  
                    }  
                });  
    }  
}  

/** 
 * 天气 Presenter接口 
 */  
public interface WeatherPresenter {  
    /** 
     * 获取天气的逻辑 
     */  
    void getWeather(String cityNO);  
   
}  
   
/** 
 * 在Presenter层实现，给Model层回调，更改View层的状态，确保Model层不直接操作View层 
 */  
public interface OnWeatherListener {  
    /** 
     * 成功时回调 
     * 
     * @param weather 
     */  
    void onSuccess(Weather weather);  
    /** 
     * 失败时回调，简单处理，没做什么 
     */  
    void onError();  
   
}  
  
/** 
 * 天气 Presenter实现 
 */  
public class WeatherPresenterImpl implements WeatherPresenter, OnWeatherListener {  
    /*Presenter作为中间层，持有View和Model的引用*/  
    private WeatherView weatherView;  
    private WeatherModel weatherModel;  
   
    public WeatherPresenterImpl(WeatherView weatherView) {  
        this.weatherView = weatherView;  
        weatherModel = new WeatherModelImpl();  
    }  
   
    @Override  
    public void getWeather(String cityNO) {  
        weatherView.showLoading();  
        weatherModel.loadWeather(cityNO, this);  
    }  
   
    @Override  
    public void onSuccess(Weather weather) {  
        weatherView.hideLoading();  
        weatherView.setWeatherInfo(weather);  
    }  
   
    @Override  
    public void onError() {  
        weatherView.hideLoading();  
        weatherView.showError();  
    }  
}  

public interface WeatherView {  
    void showLoading();  
   
    void hideLoading();  
   
    void showError();  
   
    void setWeatherInfo(Weather weather);  
}  

/** 
 * 天气界面 
 */  
public class WeatherActivity extends BaseActivity implements WeatherView, View.OnClickListener {  
    ..........................  
    private WeatherPresenter weatherPresenter;  
   
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
        init();  
   
    }  
   
    private void init() {  
 <span style="white-space:pre">   </span>....................  
  
        findView(R.id.btn_go).setOnClickListener(this);  
   
        weatherPresenter = new WeatherPresenterImpl(this); //传入WeatherView  
        loadingDialog = new ProgressDialog(this);  
        loadingDialog.setTitle("加载天气中...");  
    }  
   
    @Override  
    public void onClick(View v) {  
        switch (v.getId()) {  
            case R.id.btn_go:  
                weatherPresenter.getWeather(cityNOInput.getText().toString().trim());  
                break;  
        }  
    }  
   
   
    @Override  
    public void showLoading() {  
        loadingDialog.show();  
    }  
   
    @Override  
    public void hideLoading() {  
        loadingDialog.dismiss();  
    }  
   
    @Override  
    public void showError() {  
        //Do something  
        Toast.makeText(getApplicationContext(), "error", Toast.LENGTH_SHORT).show();  
    }  
   
    @Override  
    public void setWeatherInfo(Weather weather) {  
        WeatherInfo info = weather.getWeatherinfo();  
        //更新界面  
        .....................  
    }  
   
}  
```
## 对比
 
 ![](http://img.blog.csdn.net/20160120100717863)
 从上图可以看出：MVC的耦合性还是较高的，View可以直接访问Model，导致3者之间构成了回路。
 所以两者的主要区别是，MVP中View不能直接访问Model，需要通过Presenter发出请求，View与Model不能直接通信。
 
mvc存在的问题：
View强依赖于Model是MVC的主要问题。由此导致很多控件都是根据业务定制，从Android的角度来看，原本可以由一个通用的layout就能实现的控件，
由于要绑定实体模型，现在必须要自定义控件，这导致出现大量不必要的重复代码。因此有必要将View和Model进行解耦，而MVP的主要思想就是解耦View和Model。

 MVP存在的问题:
尽管已经有了大量的应用，但不可否认该模式的还是存在一些问题，这些问题在携程的使用过程中也得到了体现。比如，上下文丢失问题，生命周期问题，内存泄露问题以及大量的自定义接口，回调链变长等问题。可以归纳为：
1.业务复杂时，可能使得Activity变成更加复杂，比如要实现N个IView，然后写更多个模版方法。
2.业务复杂时，各个角色之间通信会变得很冗长和复杂，回调链过长。
3.Presenter处理业务，让业务变得很分散，不能全局掌握业务，很难去回答某个业务究竟是在哪里处理的。
4.用Presenter替代Controller是一个危险的做法，可能出现内存泄漏，生命周期不同步，上下文丢失等问题。
 