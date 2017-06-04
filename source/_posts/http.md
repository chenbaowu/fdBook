---
title: android网络编程(httpurlconnect接口,httpclient接口,与socket接口)
date: 2017-05-10 17:20:11
categories: android
tags: 网络编程
---

## 前言
Android中提供的HttpURLConnection和HttpClient接口可以用来开发HTTP程序。
HttpClient实际上是对Java提供方法的一些封装，在HttpURLConnection中的输入输出流操作，在这个接口中被统一封装成了HttpPost(HttpGet)和HttpResponse，这样，就减少了操作的繁琐性。
另外，在使用POST方式进行传输时，需要进行字符编码。
Manifest文件中权限的设定：Xml代码 
< uses-permission android:name="android.permission.INTERNET" /> 

<!-- more -->

## 1.HttpURLConnection接口
首先需要明确的是，Http通信中的POST和GET请求方式的不同。GET可以获得静态页面，也可以把参数放在URL字符串后面，传递给服务器。而POST方法的参数是放在Http请求中。因此，在编程之前，应当首先明确使用的请求方法，然后再根据所使用的方式选择相应的编程方式。
HttpURLConnection是继承于URLConnection类，二者都是抽象类。其对象主要通过URL的openConnection方法获得。创建方法如下代码所示：
``` bash
URL url = new URL("http://www.51cto.com/index.jsp?par=123456");    
HttpURLConnection urlConn=(HttpURLConnection)url.openConnection();

//设置输入和输出流    
urlConn.setDoOutput(true);    
urlConn.setDoInput(true);    
//设置请求方式为POST    
urlConn.setRequestMethod("POST");    
//POST请求不能使用缓存    
urlConn.setUseCaches(false);   
//关闭连接    
urlConn.disConnection();  
```
具体调用
``` bash
//以Get方式上传参数  
public class Activity03 extends Activity  
{  
    private final String DEBUG_TAG = "Activity03";   
    /** Called when the activity is first created. */  
    @Override  
    public void onCreate(Bundle savedInstanceState)  
    {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.http);    
        TextView mTextView = (TextView)this.findViewById(R.id.TextView_HTTP);  
        //http地址"?par=abcdefg"是我们上传的参数  
        String httpUrl = "http://192.168.1.110:8080/httpget.jsp?par=abcdefg";  
        //获得的数据  
        String resultData = "";  
        URL url = null;  
        try  
        {  
            //构造一个URL对象  
            url = new URL(httpUrl);   
        }  
        catch (MalformedURLException e)  
        {  
            Log.e(DEBUG_TAG, "MalformedURLException");  
        }  
        if (url != null)  
        {  
            try  
            {  
                // 使用HttpURLConnection打开连接  
                HttpURLConnection urlConn = (HttpURLConnection) url.openConnection();  
                //得到读取的内容(流)  
                InputStreamReader in = new InputStreamReader(urlConn.getInputStream());  
                // 为输出创建BufferedReader  
                BufferedReader buffer = new BufferedReader(in);  
                String inputLine = null;  
                //使用循环来读取获得的数据  
                while (((inputLine = buffer.readLine()) != null))  
                {  
                    //我们在每一行后面加上一个"\n"来换行  
                    resultData += inputLine + "\n";  
                }           
                //关闭InputStreamReader  
                in.close();  
                //关闭http连接  
                urlConn.disconnect();  
                //设置显示取得的内容  
                if ( resultData != null )  
                {  
                    mTextView.setText(resultData);  
                }  
                else   
                {  
                    mTextView.setText("读取的内容为NULL");  
                }  
            }  
            catch (IOException e)  
            {  
                Log.e(DEBUG_TAG, "IOException");  
            }  
        }  
        else  
        {  
            Log.e(DEBUG_TAG, "Url NULL");  
        }  
}

//以post方式上传参数  
public class Activity04  extends Activity  
{  
    private final String DEBUG_TAG = "Activity04";   
    /** Called when the activity is first created. */  
    @Override  
    public void onCreate(Bundle savedInstanceState)  
    {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.http);  
          
        TextView mTextView = (TextView)this.findViewById(R.id.TextView_HTTP);  
        //http地址"?par=abcdefg"是我们上传的参数  
        String httpUrl = "http://192.168.1.110:8080/httpget.jsp";  
        //获得的数据  
        String resultData = "";  
        URL url = null;  
        try  
        {  
            //构造一个URL对象  
            url = new URL(httpUrl);   
        }  
        catch (MalformedURLException e)  
        {  
            Log.e(DEBUG_TAG, "MalformedURLException");  
        }  
        if (url != null)  
        {  
            try  
            {  
                // 使用HttpURLConnection打开连接  
                HttpURLConnection urlConn = (HttpURLConnection) url.openConnection();  
                //因为这个是post请求,设立需要设置为true  
                urlConn.setDoOutput(true);  
                urlConn.setDoInput(true);  
                // 设置以POST方式  
                urlConn.setRequestMethod("POST");  
                // Post 请求不能使用缓存  
                urlConn.setUseCaches(false);  
                urlConn.setInstanceFollowRedirects(true);  
                // 配置本次连接的Content-type，配置为application/x-www-form-urlencoded的  
                urlConn.setRequestProperty("Content-Type","application/x-www-form-urlencoded");  
                // 连接，从postUrl.openConnection()至此的配置必须要在connect之前完成，  
                // 要注意的是connection.getOutputStream会隐含的进行connect。  
                urlConn.connect();  
                //DataOutputStream流  
                DataOutputStream out = new DataOutputStream(urlConn.getOutputStream());  
                //要上传的参数  
                String content = "par=" + URLEncoder.encode("ABCDEFG", "gb2312");  
                //将要上传的内容写入流中  
                out.writeBytes(content);   
                //刷新、关闭  
                out.flush();  
                out.close();   
                //获取数据  
                BufferedReader reader = new BufferedReader(new InputStreamReader(urlConn.getInputStream()));  
                String inputLine = null;  
                //使用循环来读取获得的数据  
                while (((inputLine = reader.readLine()) != null))  
                {  
                    //我们在每一行后面加上一个"\n"来换行  
                    resultData += inputLine + "\n";  
                }           
                reader.close();  
                //关闭http连接  
                urlConn.disconnect();  
                //设置显示取得的内容  
                if ( resultData != null )  
                {  
                    mTextView.setText(resultData);  
                }  
                else   
                {  
                    mTextView.setText("读取的内容为NULL");  
                }  
            }  
            catch (IOException e)  
            {  
                Log.e(DEBUG_TAG, "IOException");  
            }  
        }  
        else  
        {  
            Log.e(DEBUG_TAG, "Url NULL");  
        }  
    }  
}  
```
##  2.HttpClient接口
使用Apache提供的HttpClient接口同样可以进行HTTP操作。
对于GET和POST请求方法的操作有所不同。GET方法的操作代码示例如下：
``` bash
public class Activity02 extends Activity  
{  
    /** Called when the activity is first created. */  
    @Override  
    public void onCreate(Bundle savedInstanceState)  
    {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.http);  
        TextView mTextView = (TextView) this.findViewById(R.id.TextView_HTTP);  
        // http地址  
        String httpUrl = "http://192.168.1.110:8080/httpget.jsp?par=HttpClient_android_Get";  
        //HttpGet连接对象  
        HttpGet httpRequest = new HttpGet(httpUrl);  
        try  
        {  
            //取得HttpClient对象  
            HttpClient httpclient = new DefaultHttpClient();  
            //请求HttpClient，取得HttpResponse  
            HttpResponse httpResponse = httpclient.execute(httpRequest);  
            //请求成功  
            if (httpResponse.getStatusLine().getStatusCode() == HttpStatus.SC_OK)  
            {  
                //取得返回的字符串  
                String strResult = EntityUtils.toString(httpResponse.getEntity());  
                mTextView.setText(strResult);  
            }  
            else  
            {  
                mTextView.setText("请求错误!");  
            }  
        }  
        catch (ClientProtocolException e)  
        {  
            mTextView.setText(e.getMessage().toString());  
        }  
        catch (IOException e)  
        {  
            mTextView.setText(e.getMessage().toString());  
        }  
        catch (Exception e)  
        {  
            mTextView.setText(e.getMessage().toString());  
        }    
      
            }  
}  
```
使用POST方法进行参数传递时，需要使用NameValuePair来保存要传递的参数，另外，还需要设置所使用的字符集。代码如下所示：
``` bash
public class Activity03 extends Activity  
{  
    /** Called when the activity is first created. */  
    @Override  
    public void onCreate(Bundle savedInstanceState)  
    {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.http);  
        TextView mTextView = (TextView) this.findViewById(R.id.TextView_HTTP);  
        // http地址  
        String httpUrl = "http://192.168.1.110:8080/httpget.jsp";  
        //HttpPost连接对象  
        HttpPost httpRequest = new HttpPost(httpUrl);  
        //使用NameValuePair来保存要传递的Post参数  
        List<NameValuePair> params = new ArrayList<NameValuePair>();  
        //添加要传递的参数  
        params.add(new BasicNameValuePair("par", "HttpClient_android_Post"));  
        try  
        {  
            //设置字符集  
            HttpEntity httpentity = new UrlEncodedFormEntity(params, "gb2312");  
            //请求httpRequest  
            httpRequest.setEntity(httpentity);  
            //取得默认的HttpClient  
            HttpClient httpclient = new DefaultHttpClient();  
            //取得HttpResponse  
            HttpResponse httpResponse = httpclient.execute(httpRequest);  
            //HttpStatus.SC_OK表示连接成功  
            if (httpResponse.getStatusLine().getStatusCode() == HttpStatus.SC_OK)  
            {  
                //取得返回的字符串  
                String strResult = EntityUtils.toString(httpResponse.getEntity());  
                mTextView.setText(strResult);  
            }  
            else  
            {  
                mTextView.setText("请求错误!");  
            }  
        }  
        catch (ClientProtocolException e)  
        {  
            mTextView.setText(e.getMessage().toString());  
        }  
        catch (IOException e)  
        {  
            mTextView.setText(e.getMessage().toString());  
        }  
        catch (Exception e)  
        {  
            mTextView.setText(e.getMessage().toString());  
        }    
    }  
}  
````
## 3 Socket编程实例：
创建一个java 类作为服务器，android 应用程序作为客户端
服务器端代码：
``` bash
package com.server;  
  
import java.io.IOException;  
import java.io.OutputStream;  
import java.net.ServerSocket;  
import java.net.Socket;  
import java.text.SimpleDateFormat;  
  
public class MyServer {  
      
    private  static int count=0;  
    public static void main(String[]args){  
          
        try {  
            //实例化服务器套接字 设置端口号8888  
            ServerSocket server=new ServerSocket(8888);  
            while(true){  
                //连接编号设置  
                count=count+1;  
                //时间格式  
                SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS");  
                //实例化客户端  
                Socket client=server.accept();  
                //实例化时间  以及 id  
                System.out.println(count+":"+sdf.format(System.currentTimeMillis()));  
                //获取输出流  
                OutputStream out=client.getOutputStream();  
                //输出字符串  
                String msg="Hello,Android!";  
                //写字符串  
                out.write(msg.getBytes());  
            }  
        } catch (IOException e) {  
            // TODO Auto-generated catch block  
            e.printStackTrace();  
        }  
          
          
    }  
}  
```
android 客户端代码：
``` bash
package com.client;  
  
import java.io.IOException;  
import java.io.InputStream;  
import java.net.Socket;  
import java.net.UnknownHostException;  
  
import android.app.Activity;  
import android.os.Bundle;  
import android.view.View;  
import android.view.View.OnClickListener;  
import android.widget.Button;  
import android.widget.TextView;  
  
public class MyClientActivity extends Activity {  
    /** Called when the activity is first created. */  
    private Button rev=null;  
    private TextView revtext=null;  
    @Override  
    public void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.main);  
        rev=(Button)findViewById(R.id.rev);      
        revtext=(TextView)findViewById(R.id.receiver);  
        rev.setOnClickListener(new receiverlistenr());  
    }  
    class receiverlistenr implements OnClickListener{  
        public void onClick(View v) {  
            // TODO Auto-generated method stub  
            try {  
                //实例化Socket  
                Socket socket=new Socket("169.254.202.149",8888);  
                //获得输入流  
                InputStream in=socket.getInputStream();  
                //缓冲区  
                byte[] buffer=new byte[in.available()];  
                //读取缓冲区  
                in.read(buffer);  
                //转换字符串  
                String msg=new String(buffer);  
                //设置文本框的字符串  
                revtext.setText(msg);  
            } catch (UnknownHostException e) {  
                // TODO Auto-generated catch block  
                e.printStackTrace();  
            } catch (IOException e) {  
                // TODO Auto-generated catch block  
                e.printStackTrace();  
            }  
              
        }  
    }  
}  
```