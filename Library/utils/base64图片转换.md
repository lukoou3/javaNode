## base64图片转换

相互转换：
```java
package com.dp.dac.util;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;

import sun.misc.BASE64Decoder;
import sun.misc.BASE64Encoder;

public class base64Change {
    /**
     * @param imgStr base64编码字符串
     * @param path   图片路径-具体到文件
     */
    public static boolean generateImage(String imgStr, String path) {
        if (imgStr == null)
            return false;
        BASE64Decoder decoder = new BASE64Decoder();
        try {
// 解密
            byte[] b = decoder.decodeBuffer(imgStr);
// 处理数据
            for (int i = 0; i < b.length; ++i) {
                if (b[i] < 0) {
                    b[i] += 256;
                }
            }
            OutputStream out = new FileOutputStream(path);
            out.write(b);
            out.flush();
            out.close();
            return true;
        } catch (Exception e) {
            return false;
        }
    }
    //图片转化成base64字符串  
    public static String GetImageStr()  
    {//将图片文件转化为字节数组字符串，并对其进行Base64编码处理  
        String imgFile = "F:\\tupian\\a.jpg";//待处理的图片
                         // 地址也有写成"F:/deskBG/86619-107.jpg"形式的
        InputStream in = null;  
        byte[] data = null;  
        //读取图片字节数组  
        try   
        {  
            in = new FileInputStream(imgFile);          
            data = new byte[in.available()];  
            in.read(data);  
            in.close();  
        }   
        catch (IOException e)   
        {  
            e.printStackTrace();  
        }  
        //对字节数组Base64编码  
        BASE64Encoder encoder = new BASE64Encoder();  
        return encoder.encode(data);//返回Base64编码过的字节数组字符串  
    }

    public static void main(String[] args)
	{
    	String imgStr= "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";
    	generateImage(imgStr, "D:\\aaa.png");
	}
}

```







