---
layout: w
title: Httpclient使用详解
date: 2017-11-22 14:35:08
categories:
- Java
tags:
- httpclient
---

##post请求
<!--more-->
```javascript
  public static JSONObject sendSms(String imei, String app_id, JSONObject payload, String ttl, String priority, String publisher_id) {
          HttpClient httpclient = new DefaultHttpClient();
          String smsUrl = ""; //接口地址
          if (payload.getString("action") == "screenShot") {
              smsUrl = "http://192.168.1.1/user/" + imei + "/asyn";
          } else {
              smsUrl = "http://192.168.1.1/user/" + imei + "/syn";
          }
          HttpPost httppost = new HttpPost(smsUrl);
          JSONObject result = new JSONObject();
          String strResult = "";
          try {
              List<NameValuePair> formatdata = new ArrayList<NameValuePair>();
              formatdata.add(new BasicNameValuePair("app_id", app_id));
              formatdata.add(new BasicNameValuePair("payload", getStringFromJson(payload)));
              formatdata.add(new BasicNameValuePair("ttl", ttl));
              formatdata.add(new BasicNameValuePair("priority", priority));
              formatdata.add(new BasicNameValuePair("publisher_id", publisher_id));
  
              httppost.addHeader("Content-type", "application/x-www-form-urlencoded");
              httppost.setEntity(new UrlEncodedFormEntity(formatdata, "UTF-8"));
  
              HttpResponse response = httpclient.execute(httppost);
              String conResult = EntityUtils.toString(response.getEntity());
              result = fromObject(conResult);
          } catch (ClientProtocolException e) {
              e.printStackTrace();
          } catch (IOException e) {
              e.printStackTrace();
          } finally {
              httpclient.getConnectionManager().shutdown();
          }
  
          return result;
      }
      
```

##get请求

```javascript
  public static JSONObject sendSms(String onlineurl){ //获取
          HttpClient httpclient = new DefaultHttpClient();
          HttpGet httpget=new HttpGet(onlineurl);
          //HttpPost httppost = new HttpPost(onlineurl);
          JSONObject result = new JSONObject();
          String strResult = "";
          try {
              List<NameValuePair> formatdata = new ArrayList<NameValuePair>();
  
              // 设置请求和传输超时时间
              RequestConfig requestConfig = RequestConfig.custom()
                      .setSocketTimeout(2000).setConnectTimeout(2000).build();
              httpget.setConfig(requestConfig);
              HttpResponse response = httpclient.execute(httpget);
              String conResult = EntityUtils.toString(response.getEntity());
              result = fromObject(conResult);
          } catch (ClientProtocolException e) {
              e.printStackTrace();
          } catch (IOException e) {
              e.printStackTrace();
          } finally {
              httpclient.getConnectionManager().shutdown();
          }
  
          return result;
      }
      
      
       //json对象转为json字符串
          private static String getStringFromJson(JSONObject adata) {
              StringBuffer sb = new StringBuffer();
              sb.append("{");
              for (Object key : adata.keySet()) {
                  sb.append("\"" + key + "\":\"" + adata.get(key) + "\",");
              }
              String rtn = sb.toString().substring(0, sb.toString().length() - 1) + "}";
              return rtn;
          }
```



