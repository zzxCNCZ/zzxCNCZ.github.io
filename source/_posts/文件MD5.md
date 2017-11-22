---
title: 文件MD5
date: 2017-11-22 15:15:28
tags:
---

*根据文件获取MD5
```javascript
  //获取单个文件的MD5的值
      public static String mD5(MultipartFile file)
      {
          if (file == null && StringUtil.isNull(file.getOriginalFilename()))
          {
              return null;
          }
          MessageDigest digest   = null;
          InputStream fis      = null;
          byte            buffer[] = new byte[1024];
          int             len;
          try
          {
              digest = MessageDigest.getInstance("MD5");
               fis = file.getInputStream();
              while ((len = fis.read(buffer, 0, 1024)) != -1)
              {
                  digest.update(buffer, 0, len);
              }
          }
          catch (Exception e)
          {
              e.printStackTrace();
              return "";
          }
          finally
          {
              if (fis != null)
              {
                  try
                  {
                      fis.close();
                  }
                  catch (IOException e)
                  {
                      e.printStackTrace();
                  }
              }
          }
          return bytesToHexString(digest.digest());
      }
  
      private static String bytesToHexString(byte[] src)
      {
          if (src == null || src.length <= 0)
          {
              return null;
          }
          StringBuilder stringBuilder = new StringBuilder();
          for (byte aSrc : src)
          {
              int    v  = aSrc & 0xFF;
              String hv = Integer.toHexString(v);
              if (hv.length() < 2)
              {
                  stringBuilder.append(0);
              }
              stringBuilder.append(hv);
          }
          return stringBuilder.toString();
      }
```
