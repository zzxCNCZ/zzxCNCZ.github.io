---
title: MultipartFile上传——使用详解
date: 2017-11-22 14:59:54
tags:
---
##上传文件
```javascript
  @RequestMapping("/uploadfile")
      @ResponseBody
      private message uploadfile(@RequestParam("attachment") MultipartFile[] multipartfiles, HttpServletRequest request) throws IOException {
          MultipartHttpServletRequest mulReq = (MultipartHttpServletRequest) request;
          /*MultipartFile file = mulReq.getFile("attachment");*/
          String savePath = "E:\\ludy\\mdm\\attachment";//本地地址
          String filename = "";
          message retmsg = new message();//返回对象
  
          try {
              if (null != multipartfiles && multipartfiles.length > 0) {
                  //遍历并保存文件
                  for (MultipartFile file : multipartfiles) {
                      String originname = file.getOriginalFilename();
                      //String [] suffix=originname.split(".");
                      String prefix = originname.substring(originname.lastIndexOf(".") + 1);
                      filename = mD5(file);
                      file.transferTo(new File(savePath + "\\" + filename + "." + prefix));
                      retmsg.setRet("200");
                      retmsg.setMsg(filename + "." + prefix);//后缀
                  }
              }
          } catch (Exception e) {
              e.printStackTrace();
          }
          return retmsg;
  
      }
      
     
```
##上传文件到服务器（尚有问题，但可行）
```javascript
  public static message uploadFile(MultipartFile file, String url) {
  
          message ret = new message();
          JSONObject result = new JSONObject();
  
          HttpClient httpclient = new DefaultHttpClient();
          String smsUrl = "http://192.168.1.1/upload";//服务器地址
          HttpPost httppost = new HttpPost(smsUrl);
          try {
              FileBody bin = new FileBody(multipartToFile(file));
              HttpEntity reqEntity= MultipartEntityBuilder.create().addPart("attachment",bin ).build();
              httppost.setEntity(reqEntity);
              // 发起请求 并返回请求的响应
              HttpResponse response = httpclient.execute(httppost);
              HttpEntity resEntity = response.getEntity();
              String conResult = EntityUtils.toString(response.getEntity());
              result = fromObject(conResult);
              ret.setRet(result.getString("ret"));
              ret.setMsg(result.getString("msg"));
  
          } catch (ClientProtocolException e) {
              e.printStackTrace();
          } catch (IOException e) {
              e.printStackTrace();
          } finally {
              httpclient.getConnectionManager().shutdown();
          }
  
          return ret;
  
      }
      //将MultipartFile转换为file类型文件
      public static File multipartToFile(MultipartFile multipartFile) throws IOException {
          CommonsMultipartFile cf = (CommonsMultipartFile)multipartFile;
          DiskFileItem fi = (DiskFileItem) cf.getFileItem();
          File file = fi.getStoreLocation();
          //手动创建临时文件
          System.out.print(file.length());
          if(file.length() < 2048){
              File tmpFile = new File(System.getProperty("java.io.tmpdir") + System.getProperty("file.separator") +
                      file.getName());
              multipartFile.transferTo(tmpFile);
              return tmpFile;
          }
          return file;
      }
```
