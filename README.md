# coolweather
Android 第一行代码（第二版）的实战项目
<br>
酷欧天气 coolweather
<br>

- #### 使用的开源库

1. okHttp
2. Glide
3. gson

- #### 在写的过程遇见过一些问题

1. okHttp3 开源框架下，实现的sendOkHttpRequest ，收到接口地址访问结果，status != ok ，总是onFailure情况。

2. 模拟器上可以运行，手机上加载失败（加载城市）。 在queryCounties中调用 queryFromServer（）目的是获得服务器返回的天气数据。通过sendOkHttpRequesst()方法。返回callback 进入了onFailure（）方法。

   模拟器能够跑起来，URL这些是没有问题的。

   手机使用的Android 9.0 应该需要在某处设置一些参数。(已解决)

   ```java
   private void queryCounties(){
       //访问服务器 获取数据
          String address = "http://guolin.tech/api/china/"+provinceCode + "/" + cityCode;
          queryFromServer(address,"county");
       }
   
   
   /**
        * 根据传入的地址 和 类型 从服务器上查询省市,,查询后存入数据库 ，再调用上面三个方法
        */
   
       private void queryFromServer(String address , final String type){
           showProgressDialog();
           HttpUtil.sendOkHttpRequest(address, new Callback() {
   
   
               @Override
               public void onFailure(Call call, IOException e) {
                   //通过runOnUiThread() 方法回到主线程
                   getActivity().runOnUiThread(new Runnable() {
                       @Override
                       public void run() {
                           closeProgressDialog();
                           Toast.makeText(getContext()
                                          ,"加载失败"
                                          ,Toast.LENGTH_SHORT).show();
                       }
                   });
               }
   
               @Override
               public void onResponse(Call call, Response response) throws IOException {
                   String responseText = response.body().string();
                   boolean result = false;
                   if ("province".equals(type)){
                       result = Utility.handleProvinceResponse(responseText);
                   }else if ("city".equals(type)){
                       result = Utility
                           .handleCityResponse(responseText , selectedProvince.getId());
                   }else if( "county".equals(type)){
                       result = Utility
                           .handleCountyResponse(responseText,selectedCity.getId());
                   }
                   
                   if(result){
                       getActivity().runOnUiThread(new Runnable() {
                           @Override
                           public void run() {
                               closeProgressDialog();
                               if ("province".equals(type)){
                                   queryProvinces();
                               }else if("city".equals(type)){
                                   queryCities();
                               }else if ("county".equals(type)){
                                   queryCounties();
                               }
                           }
                       });
                   }
               }
           });
       }
   
   ```

   - ##### 问题原因：

   应用官方的说明:在 Android 6.0 中，我们取消了对 Apache HTTP 客户端的支持。 从 Android 9 开始，默认情况下该内容库已从 bootclasspath 中移除且不可用于应用。且Android P 限制了明文流量的网络请求，非加密的流量请求都会被系统禁止掉。

   ##### 解决办法:

    在res目录下新建xml文件夹,文件夹中新建文件network_security_config.xml,文件内容如下 

   ```xml
   <network-security-config>
       <base-config cleartextTrafficPermitted="true" />
   </network-security-config>
   ```

    在AndroidManifest.xml文件中,Application标签下添加如下属性: 

   ```xml
   android:networkSecurityConfig="@xml/network_security_config"
   ```

   ————————————————

   方法来源：https://blog.csdn.net/code_hero666/article/details/89478122

3. 每次下拉刷新，数据变动（理应不变）。(未解决)

