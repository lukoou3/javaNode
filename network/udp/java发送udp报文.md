
一个发送udp报文的例子(功能的最后是发送报文)：
```java
package com.dp.dac.dao;

import java.io.ByteArrayOutputStream;
import java.io.DataOutputStream;
import java.io.IOException;
import java.net.DatagramSocket;
import java.net.InetAddress;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.text.NumberFormat;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import net.sf.json.JSONObject;
import net.sf.json.JsonConfig;

import org.apache.commons.lang.ArrayUtils;
import org.apache.http.HttpEntity;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

import com.dp.dac.param.DacHikDtServerConfParam;
import com.dp.plat.type.DateTime;
import com.dp.plat.type.IpAddr;
import com.dp.plat.util.CommonParam;
import com.dp.plat.util.DBService;

public class DacHikDtSyncDao
{
	public static class DacHikDtData
	{
		long sourceIp; //平台ip
		long termIp; //前端设备ip
		int type;//1:VehiclePass,2:Face,3:Person
		String termIdcode = ""; //前端设备编码
		String termType = ""; //前端设备类型
		String factory = ""; //前端设备厂商 
		String termModel = ""; //前端设备型号
		String location = ""; //位置
		int messageNum;//数据消息条数
		long flow;//流量	
		long time;//时间
		
		public long getSourceIp()
		{
			return sourceIp;
		}
		public void setSourceIp(long sourceIp)
		{
			this.sourceIp = sourceIp;
		}
		public long getTermIp()
		{
			return termIp;
		}
		public void setTermIp(long termIp)
		{
			this.termIp = termIp;
		}		
		public int getType()
		{
			return type;
		}
		public void setType(int type)
		{
			this.type = type;
		}
		public String getTermIdcode()
		{
			return termIdcode;
		}
		public void setTermIdcode(String termIdcode)
		{
			this.termIdcode = termIdcode;
		}
		public String getTermType()
		{
			return termType;
		}
		public void setTermType(String termType)
		{
			this.termType = termType;
		}
		public String getFactory()
		{
			return factory;
		}
		public void setFactory(String factory)
		{
			this.factory = factory;
		}
		public String getTermModel()
		{
			return termModel;
		}
		public void setTermModel(String termModel)
		{
			this.termModel = termModel;
		}
		public String getLocation()
		{
			return location;
		}
		public void setLocation(String location)
		{
			this.location = location;
		}
		public int getMessageNum()
		{
			return messageNum;
		}
		public void setMessageNum(int messageNum)
		{
			this.messageNum = messageNum;
		}
		public long getFlow()
		{
			return flow;
		}
		public void setFlow(long flow)
		{
			this.flow = flow;
		}
		public long getTime()
		{
			return time;
		}
		public void setTime(long time)
		{
			this.time = time;
		}				
	}
	
	public static class DacHikDtVehiclePassReplyData
	{
		public static class Data
		{
			String id;
			String subResourceId;
			String mainResourceId;
			int[] hourDetail;
			int[][] minuteDetail;
			int total;		
			String belongTime;
			
			public String getId()
			{
				return id;
			}
			public void setId(String id)
			{
				this.id = id;
			}
			public String getSubResourceId()
			{
				return subResourceId;
			}
			public void setSubResourceId(String subResourceId)
			{
				this.subResourceId = subResourceId;
			}
			public String getMainResourceId()
			{
				return mainResourceId;
			}
			public void setMainResourceId(String mainResourceId)
			{
				this.mainResourceId = mainResourceId;
			}
			public int[] getHourDetail()
			{
				return hourDetail;
			}
			public void setHourDetail(int[] hourDetail)
			{
				this.hourDetail = hourDetail;
			}
			public int[][] getMinuteDetail()
			{
				return minuteDetail;
			}
			public void setMinuteDetail(int[][] minuteDetail)
			{
				this.minuteDetail = minuteDetail;
			}
			public int getTotal()
			{
				return total;
			}
			public void setTotal(int total)
			{
				this.total = total;
			}
			public String getBelongTime()
			{
				return belongTime;
			}
			public void setBelongTime(String belongTime)
			{
				this.belongTime = belongTime;
			}
			@Override
			public String toString()
			{
				return "Data [id=" + id + ", subResourceId=" + subResourceId
						+ ", mainResourceId=" + mainResourceId + ", hourDetail="
						+ Arrays.toString(hourDetail) + ", minuteDetail="
						+ Arrays.toString(minuteDetail) + ", total=" + total
						+ ", belongTime=" + belongTime + "]";
			}
									
		}
		
		private String code;
		private String msg;	
		private Data[] data;

		public String getCode()
		{
			return code;
		}

		public void setCode(String code)
		{
			this.code = code;
		}

		public String getMsg()
		{
			return msg;
		}

		public void setMsg(String msg)
		{
			this.msg = msg;
		}

		public Data[] getData()
		{
			return data;
		}

		public void setData(Data[] data)
		{
			this.data = data;
		}

		@Override
		public String toString()
		{
			return "DacHikDtVehiclePassReplyData [code=" + code + ", msg=" + msg
					+ ", data=" + Arrays.toString(data) + "]";
		}		
	}
	
	public static class DacHikDtFaceReplyData
	{
		public static class Data
		{
			String id;
			String deviceID;
			String type;
			int[] hourDetail;
			int[][] minuteDetail;
			int total;
			String other;		
			String time;
			
			public String getId()
			{
				return id;
			}

			public void setId(String id)
			{
				this.id = id;
			}

			public String getDeviceID()
			{
				return deviceID;
			}

			public void setDeviceID(String deviceID)
			{
				this.deviceID = deviceID;
			}

			public String getType()
			{
				return type;
			}

			public void setType(String type)
			{
				this.type = type;
			}

			public int[] getHourDetail()
			{
				return hourDetail;
			}

			public void setHourDetail(int[] hourDetail)
			{
				this.hourDetail = hourDetail;
			}

			public int[][] getMinuteDetail()
			{
				return minuteDetail;
			}

			public void setMinuteDetail(int[][] minuteDetail)
			{
				this.minuteDetail = minuteDetail;
			}

			public int getTotal()
			{
				return total;
			}

			public void setTotal(int total)
			{
				this.total = total;
			}

			public String getOther()
			{
				return other;
			}

			public void setOther(String other)
			{
				this.other = other;
			}

			public String getTime()
			{
				return time;
			}

			public void setTime(String time)
			{
				this.time = time;
			}

			@Override
			public String toString()
			{
				return "Data [id=" + id + ", deviceID=" + deviceID + ", type="
						+ type + ", hourDetail=" + Arrays.toString(hourDetail)
						+ ", minuteDetail=" + Arrays.toString(minuteDetail)
						+ ", total=" + total + ", other=" + other + ", time="
						+ time + "]";
			}					
		}
		
		private String code;
		private String msg;	
		private Data[] data;

		public String getCode()
		{
			return code;
		}

		public void setCode(String code)
		{
			this.code = code;
		}

		public String getMsg()
		{
			return msg;
		}

		public void setMsg(String msg)
		{
			this.msg = msg;
		}

		public Data[] getData()
		{
			return data;
		}

		public void setData(Data[] data)
		{
			this.data = data;
		}

		@Override
		public String toString()
		{
			return "DacHikDtVehiclePassReplyData [code=" + code + ", msg=" + msg
					+ ", data=" + Arrays.toString(data) + "]";
		}		
	}
	
	private static long lastFetchTime = CommonParam.getInstance().getLong("dac.hikdt.last.fetch.time", 0L);
	private static DacHikDtServerConfParam serverConf = null;
	
	public static void serverConfUpdate(DacHikDtServerConfParam param)
	{
		serverConf = param;
	}
	
	public static void syncHikDtData()
	{
		if(serverConf == null)
		{
			serverConf = new DacHikDtServerConfDao().getHikDtServerConfParam();
		}		
		if(!serverConf.isEnable())
		{
			return;
		}
		
		long fetchTime = DateTime.timeToday().longValue();
		if(new DateTime().longValue() - fetchTime < 60*60)
		{
			return;
		}		
		
		if(lastFetchTime >= fetchTime)
		{
			return;
		}
				
		List<DacHikDtData> list = null;
		try
		{
			list = fetchHikDtData();						
		}
		catch (Exception e)
		{
			e.printStackTrace();
		}
		
		if(list == null)
		{
			return;
		}
		
		try
		{
			saveHikDtData(list);			
		}
		catch (Exception e)
		{
			e.printStackTrace();
			return;
		}
		
		lastFetchTime = fetchTime;
	}
		
	private static List<DacHikDtData> fetchHikDtData() throws Exception
	{				
		String ip = serverConf.getWebServerIp();
		int port = serverConf.getWebServerPort();
		SimpleDateFormat dateFormatter = new SimpleDateFormat("yyyyMMdd");
		String daybegin = dateFormatter.format(DateTime.timeYesterday());
		String dayend = dateFormatter.format(DateTime.timeToday());
		
		DacHikDtVehiclePassReplyData vehiclePassReply = fetchHikDtVehiclePass(ip,port,daybegin,dayend);
		//if(vehiclePassReply == null || !"0".equals(vehiclePassReply.getCode()))
		if(vehiclePassReply == null)
		{
			throw new Exception("获取过车数据量失败");
		}
		
		DacHikDtFaceReplyData faceReply = fetchHikDtFace(ip,port,daybegin,dayend);
		//if(faceReply == null || !"0".equals(faceReply.getCode()))
		if(faceReply == null)
		{
			throw new Exception("获取人脸抓拍数据量失败");
		}
		
		DacHikDtVehiclePassReplyData.Data[] vehiclePassArray = vehiclePassReply.getData();
		DacHikDtFaceReplyData.Data[] faceArray = faceReply.getData();
		
		if(vehiclePassArray == null)
		{
			vehiclePassArray = new DacHikDtVehiclePassReplyData.Data[0];
		}
		if(faceArray == null)
		{
			faceArray = new DacHikDtFaceReplyData.Data[0];
		}
		
		
		List<DacHikDtData> list = new ArrayList<DacHikDtData>(vehiclePassArray.length + faceArray.length);
		Map<String, DacHikDtData> vehiclePassMap = new HashMap<String,DacHikDtData>((int) ((float) vehiclePassArray.length / 0.75F + 1.0F));
		Map<String, DacHikDtData> faceMap = new HashMap<String,DacHikDtData>((int) ((float) faceArray.length / 0.75F + 1.0F));
				
		IpAddr ipAddr = new IpAddr(serverConf.getWebServerIp());
		long serverIp = ipAddr.getIp();	
		long time = DateTime.timeYesterday().longValue();//取昨天的时间
		
		DacHikDtData dtData;
		DacHikDtVehiclePassReplyData.Data vehiclePassData;		
		for (int i = 0; i < vehiclePassArray.length; i++)
		{
			vehiclePassData = vehiclePassArray[i];
			dtData = new DacHikDtData();
			dtData.termIdcode = vehiclePassData.mainResourceId;
			dtData.messageNum = vehiclePassData.total;
			dtData.type = 1;
			dtData.termType = "卡口";
			dtData.factory = "海康";
			dtData.sourceIp = serverIp;
			dtData.time = time;
			
			list.add(dtData);
			vehiclePassMap.put(dtData.termIdcode, dtData);
		}
		joinDbVehiclePassInfo(vehiclePassMap);		
		
		DacHikDtFaceReplyData.Data faceData;		
		for (int i = 0; i < faceArray.length; i++)
		{
			faceData = faceArray[i];
			dtData = new DacHikDtData();
			dtData.termIdcode = faceData.deviceID;
			dtData.messageNum = faceData.total;
			dtData.type = 2;
			dtData.termType = "人脸抓拍";
			dtData.factory = "海康";
			dtData.sourceIp = serverIp;
			dtData.time = time;
			
			list.add(dtData);
			faceMap.put(dtData.termIdcode, dtData);
		}
		joinDbFaceInfo(faceMap);				
				
		return list;		
	}
		
	private static void joinDbVehiclePassInfo(Map<String, DacHikDtData> map)
	{
		String driver = "org.postgresql.Driver";
	    String url = "jdbc:postgresql://"+serverConf.getDbIp()+":"+serverConf.getDbPort()+"/bms";
	    String username = serverConf.getDbName();
	    String password = serverConf.getDbPasswd();
	    Connection conn = null;
	    Statement stmt = null;
		ResultSet rst = null;
	    try {
	        Class.forName(driver); //classLoader,加载对应驱动
	        conn = (Connection) DriverManager.getConnection(url, username, password);
	        
	        stmt =  conn.createStatement();
	        rst = stmt.executeQuery("select CROSSING_NAME,GATCODE from bms_crossing_info");
	        DacHikDtData data;
	        while(rst.next())
			{
	        	data = map.get(rst.getString("GATCODE"));
	        	if(data != null)
	        	{
	        		data.setLocation(rst.getString("CROSSING_NAME"));
	        	}
			}	        
	    } 
	    catch (Exception e) 
	    {
	        e.printStackTrace();
	    }
	    finally
	    {
	    	DBService.closeRst(rst);
	    	if(stmt != null)
	    	{
	    		try
				{
					stmt.close();
				}
				catch (SQLException e)
				{
					e.printStackTrace();
				}
	    	}
	    	if(conn != null)
	    	{
	    		try
				{
	    			conn.close();
				}
				catch (SQLException e)
				{
					e.printStackTrace();
				}
	    	}
	    }
	}
	
	private static void joinDbFaceInfo(Map<String, DacHikDtData> map)
	{
		String driver = "org.postgresql.Driver";
	    String url = "jdbc:postgresql://"+serverConf.getDbIp()+":"+serverConf.getDbPort()+"/fas";
	    String username = serverConf.getDbName();
	    String password = serverConf.getDbPasswd();
	    Connection conn = null;
	    Statement stmt = null;
		ResultSet rst = null;
	    try {
	        Class.forName(driver); //classLoader,加载对应驱动
	        conn = (Connection) DriverManager.getConnection(url, username, password);
	        
	        stmt =  conn.createStatement();
	        rst = stmt.executeQuery("select CAMERA_IP,CAMERA_NAME,INDEX_CODE from fms_res_camera");
	        DacHikDtData data;
	        IpAddr ipAddr = new IpAddr();
	        while(rst.next())
			{
	        	data = map.get(rst.getString("INDEX_CODE"));
	        	if(data != null)
	        	{	        		
	        		data.setLocation(rst.getString("CAMERA_NAME"));
	        		if( ipAddr.parseFrom(rst.getString("CAMERA_IP")) )
	        		{
	        			data.setTermIp(ipAddr.getIp());
	        		}	        		        		
	        	}
			}	        
	    } 
	    catch (Exception e) 
	    {
	        e.printStackTrace();
	    }
	    finally
	    {
	    	DBService.closeRst(rst);
	    	if(stmt != null)
	    	{
	    		try
				{
					stmt.close();
				}
				catch (SQLException e)
				{
					e.printStackTrace();
				}
	    	}
	    	if(conn != null)
	    	{
	    		try
				{
	    			conn.close();
				}
				catch (SQLException e)
				{
					e.printStackTrace();
				}
	    	}
	    }
	}
	
	private static DacHikDtVehiclePassReplyData fetchHikDtVehiclePass(String ip,int port,String daybegin,String dayend)
	{				
		String url = "http://"+ip+":"+port+"/bms-ops/rest/getVehiclePassCount";
		
		JSONObject jsonObject = new JSONObject();
	    jsonObject.put("startTime", daybegin);
	    jsonObject.put("endTime", dayend);
	    jsonObject.put("subResourceId ", "");
	    jsonObject.put("mainResourceId", "");
		
		String requestText = postRawRequest(url,jsonObject.toString());
		
		DacHikDtVehiclePassReplyData data = null;
		
		try
		{
			JsonConfig cfg = new JsonConfig();
			String[] excludes = new String[]{"hourDetail","minuteDetail"};
			cfg.setExcludes(excludes);		
			data = (DacHikDtVehiclePassReplyData) JSONObject.toBean(JSONObject.fromObject(requestText,cfg),DacHikDtVehiclePassReplyData.class);
		}
		catch (Exception e)
		{
			e.printStackTrace();
		}
		
		return data;
	}
	
	private static DacHikDtFaceReplyData fetchHikDtFace(String ip,int port,String daybegin,String dayend)
	{	
		String url = "http://"+ip+":"+port+"/bms-ops/rest/getFaceCount?startTime="+daybegin+"&endTime="+dayend;
		
		JSONObject jsonObject = new JSONObject();
	    jsonObject.put("startTime", daybegin);
	    jsonObject.put("endTime", dayend);
	    jsonObject.put("devCodes ", "");
		
		String requestText = postRawRequest(url,jsonObject.toString());
		
		DacHikDtFaceReplyData data = null;
		
		try
		{
			JsonConfig cfg = new JsonConfig();
			String[] excludes = new String[]{"hourDetail","minuteDetail"};
			cfg.setExcludes(excludes);		
			data = (DacHikDtFaceReplyData) JSONObject.toBean(JSONObject.fromObject(requestText,cfg),DacHikDtFaceReplyData.class);
		}
		catch (Exception e)
		{
			e.printStackTrace();
		}
		
		return data;
	}
	
	private static String postRawRequest(String url, String stringJson)
	{
		CloseableHttpClient httpClient = HttpClients.createDefault();
		HttpPost httpPost = new HttpPost(url);
		
		httpPost.setHeader("Content-Type", "application/json");
		
		RequestConfig requestConfig = RequestConfig.custom().setSocketTimeout(3600000).setConnectTimeout(3000).build();
		httpPost.setConfig(requestConfig);
		
		CloseableHttpResponse response = null;		
		try
		{
			if(stringJson != null && !stringJson.equals("")){
				StringEntity stringEntity = new StringEntity(stringJson, "utf-8");
				httpPost.setEntity(stringEntity);
			}
			response = httpClient.execute(httpPost);
			HttpEntity responseEntity = response.getEntity();

			return EntityUtils.toString(responseEntity,"utf-8");
		}
		catch (Exception e)
		{
			e.printStackTrace();
		}
		finally
		{
			try
			{
				// 释放资源
				if (httpClient != null)
				{
					httpClient.close();
				}
				if (response != null)
				{
					response.close();
				}
			}
			catch (Exception e)
			{
				e.printStackTrace();
			}
		}
		
		return null;		
	}
	
	private static boolean saveHikDtData(List<DacHikDtData> list) throws Exception
	{				
		DatagramSocket socket = null;
		try
		{
			socket = new DatagramSocket();			
			java.net.DatagramPacket packet = new java.net.DatagramPacket(new byte[436+16], 436+16, InetAddress.getByName("127.0.0.1"), 9502);
			
			long time = DateTime.timeToday().longValue();	
			DacHikDtData data;
			
			for (int i = 0; i < list.size(); i++)
			{						
				data = list.get(i);
				byte[] buf = ArrayUtils.addAll(int2bytes(2, 2), int2bytes(436+16, 2));
				buf = ArrayUtils.addAll(buf, long2bytes(time, 4));
				buf = ArrayUtils.addAll(buf, intToBytes(0));
				buf = ArrayUtils.addAll(buf, intToBytes(0));
				
				buf = ArrayUtils.addAll(buf, int2bytes(211, 2));
				buf = ArrayUtils.addAll(buf, int2bytes(436, 2));//4*4 + 64*6 + 32 + 4
		        
				buf = ArrayUtils.addAll(buf, long2bytes(data.sourceIp, 4));
				buf = ArrayUtils.addAll(buf, long2bytes(data.time, 4));
				buf = ArrayUtils.addAll(buf, long2bytes(data.termIp, 4));
				buf = ArrayUtils.addAll(buf, StringToBytes(data.factory, 64));
				buf = ArrayUtils.addAll(buf, StringToBytes(data.termIdcode, 64));
				buf = ArrayUtils.addAll(buf, StringToBytes(data.termType, 64));
				buf = ArrayUtils.addAll(buf, StringToBytes(data.termModel, 64));
				buf = ArrayUtils.addAll(buf, StringToBytes(data.location, 128));
				buf = ArrayUtils.addAll(buf, StringToBytes("0", 32));
				buf = ArrayUtils.addAll(buf, intToBytes(data.messageNum));
				packet.setData(buf);
				socket.send(packet);
			}
			
		}
		catch (Exception e)
		{
			throw e;
		}
		finally
		{
			if (socket != null) 
			{
				socket.close();
			}
		}	
				
		return true;
	}
	
	public static void sendMessage(byte[] buf, String ip, int port) throws Exception
	{
		DatagramSocket socket = null;
		try
		{
			socket = new DatagramSocket();
			java.net.DatagramPacket packet = new java.net.DatagramPacket(buf, buf.length, InetAddress.getByName(ip), port);
			socket.send(packet);
		}
		catch (IOException e)
		{
			throw e;
		}
		finally
		{
			if (socket != null) 
			{
				socket.close();
			}
		}
	}
	
	public static byte[] int2bytes(int n, int length) {
		byte[] b = new byte[length];
		for( int i=0; i<4 && i<length; i++) {
			b[i] = (byte) ((n >> 8*(length-1-i)) & 0xFF);
		}		
		return b;
	}
	
	public static byte[] long2bytes(long n, int length) {
		byte[] b = new byte[length];
		for( int i=0; i<8 && i<length; i++) {
			b[i] = (byte) (n >>> 8*(length-1-i));
		}
		return b;
	}
	
	public static byte[] intToBytes(int v) {
		byte writeBuffer[] = new byte[4];		
		writeBuffer[0] = (byte)((v >>> 24) & 0xFF);
		writeBuffer[1] = (byte)((v >>> 16) & 0xFF);
		writeBuffer[2] = (byte)((v >>>  8) & 0xFF);
		writeBuffer[3] = (byte)((v >>>  0) & 0xFF);		
		return writeBuffer;
	}
	
	public static byte[] longToBytes(long v) {
		byte writeBuffer[] = new byte[8];
		writeBuffer[0] = (byte)(v >>> 56);
        writeBuffer[1] = (byte)(v >>> 48);
        writeBuffer[2] = (byte)(v >>> 40);
        writeBuffer[3] = (byte)(v >>> 32);
        writeBuffer[4] = (byte)(v >>> 24);
        writeBuffer[5] = (byte)(v >>> 16);
        writeBuffer[6] = (byte)(v >>>  8);
        writeBuffer[7] = (byte)(v >>>  0); 
        return writeBuffer;
	}
	
	public static byte[] StringToBytes(String str,int length) {
		byte writeBuffer[] = new byte[length];		
		try
		{
			byte[] bytes = str.getBytes("utf-8");			
			System.arraycopy(bytes, 0, writeBuffer, 0, bytes.length);
		}
		catch (Exception e)
		{
			e.printStackTrace();
		}	
		return writeBuffer;
	}
	
	public static void main(String[] args) throws Exception
	{
		//test1();
		test2();
		
	}
	
	public static void test2() throws Exception
	{
		System.out.println(System.currentTimeMillis());
		
		long time = DateTime.timeToday().longValue();
		DatagramSocket socket = null;
		try
		{
			socket = new DatagramSocket();
			
			java.net.DatagramPacket packet = new java.net.DatagramPacket(new byte[436+16], 436+16, InetAddress.getByName("127.0.0.1"), 9502);
			
			byte[] buf = new byte[436+16];
			byte[] zerobody = new byte[432];
			
			int destPos = 0;
			System.arraycopy(int2bytes(2, 2), 0, buf, destPos, 2);
			destPos += 2;
			System.arraycopy(int2bytes(436+16, 2), 0, buf, destPos, 2);
			destPos += 2;
			System.arraycopy(long2bytes(DateTime.timeToday().longValue(), 4), 0, buf, destPos, 4);
			destPos += 4;
			destPos += 4;
			destPos += 4;
			
			System.arraycopy(int2bytes(211, 2), 0, buf, destPos, 2);
			destPos += 2;
			System.arraycopy(int2bytes(436, 2), 0, buf, destPos, 2);
			destPos += 2;
												
			for (int i = 0; i < 1000; i++)
			{	
				destPos = 20;
				System.arraycopy(zerobody, 0, buf, destPos, zerobody.length);
								
				System.arraycopy(intToBytes(1), 0, buf, destPos, 4);
				destPos += 4;			
				System.arraycopy(long2bytes(time, 4), 0, buf, destPos, 4);
				destPos += 4;
				System.arraycopy(intToBytes(1), 0, buf, destPos, 4);
				destPos += 4;
				System.arraycopy(StringToBytes("a", 64), 0, buf, destPos, 64);
				destPos += 64;
				System.arraycopy(StringToBytes("a", 64), 0, buf, destPos, 64);
				destPos += 64;
				System.arraycopy(StringToBytes("a", 64), 0, buf, destPos, 64);
				destPos += 64;
				System.arraycopy(StringToBytes("a", 64), 0, buf, destPos, 64);
				destPos += 64;
				System.arraycopy(StringToBytes("a", 128), 0, buf, destPos, 128);
				destPos += 128;
				System.arraycopy(StringToBytes("500000000", 32), 0, buf, destPos, 32);
				destPos += 32;
				System.arraycopy(intToBytes(9), 0, buf, destPos, 4);
				
				packet.setData(buf);
				socket.send(packet);		
				//sendMessage(buf,"127.0.0.1",9502);
			}
			
		}
		catch (IOException e)
		{
			throw e;
		}
		finally
		{
			if (socket != null) 
			{
				socket.close();
			}
		}	
		
		System.out.println(System.currentTimeMillis());
	}
	
	public static void test1() throws Exception
	{
		System.out.println(System.currentTimeMillis());
		
		
		DatagramSocket socket = null;
		try
		{
			socket = new DatagramSocket();
			
			java.net.DatagramPacket packet = new java.net.DatagramPacket(new byte[436+16], 436+16, InetAddress.getByName("127.0.0.1"), 9502);
			
			for (int i = 0; i < 10000; i++)
			{						
				byte[] buf = ArrayUtils.addAll(int2bytes(2, 2), int2bytes(436+16, 2));
				//buf = ArrayUtils.addAll(buf, intToBytes(0x));
				buf = ArrayUtils.addAll(buf, long2bytes(0, 4));
				buf = ArrayUtils.addAll(buf, intToBytes(0));
				buf = ArrayUtils.addAll(buf, intToBytes(0));
				
				buf = ArrayUtils.addAll(buf, int2bytes(211, 2));
				buf = ArrayUtils.addAll(buf, int2bytes(436, 2));//4*4 + 64*6 + 32 + 4
		        
				buf = ArrayUtils.addAll(buf, intToBytes(1));
				buf = ArrayUtils.addAll(buf, long2bytes(0, 4));
				//buf = ArrayUtils.addAll(buf, intToBytes((int) new DateTime().lastValue()));
				buf = ArrayUtils.addAll(buf, intToBytes(1));
				buf = ArrayUtils.addAll(buf, StringToBytes("a", 64));
				buf = ArrayUtils.addAll(buf, StringToBytes("a", 64));
				buf = ArrayUtils.addAll(buf, StringToBytes("a", 64));
				buf = ArrayUtils.addAll(buf, StringToBytes("a", 64));
				buf = ArrayUtils.addAll(buf, StringToBytes("a", 128));
				buf = ArrayUtils.addAll(buf, StringToBytes("10", 32));
				buf = ArrayUtils.addAll(buf, intToBytes(3));
				packet.setData(buf);
				socket.send(packet);		
				//sendMessage(buf,"127.0.0.1",9502);
			}
			
		}
		catch (IOException e)
		{
			throw e;
		}
		finally
		{
			if (socket != null) 
			{
				socket.close();
			}
		}	
		
		System.out.println(System.currentTimeMillis());
	}
	
	
	public static void test() throws Exception
	{
		NumberFormat numberFormat = NumberFormat.getNumberInstance();
        numberFormat.setMinimumIntegerDigits(8);
        numberFormat.setGroupingUsed(false);
        byte [] num = numberFormat.format(0).getBytes();
        for (int i = 0; i < num.length; i++)
		{
        	System.out.println(num[i]);
		}
        System.out.println("#####################");

        num = long2bytes(100000, 8);
        for (int i = 0; i < num.length; i++)
		{
        	System.out.println(num[i]);
		}
        System.out.println("#####################");

        num = longToBytes(100000);
        for (int i = 0; i < num.length; i++)
		{
        	System.out.println(num[i]);
		}
        System.out.println("#####################");
        
        byte[] buf = null; 
        long l = 100000; 
        ByteArrayOutputStream baos = new ByteArrayOutputStream(); 
        DataOutputStream dos = new DataOutputStream(baos); 
        dos.writeLong(l); 
        dos.writeInt(1);
        buf = baos.toByteArray();
        
        for (int i = 0; i < buf.length; i++)
		{
        	System.out.println(buf[i]);
		}
        System.out.println("#####################");

        num = StringToBytes("aaaa", 3);
        for (int i = 0; i < num.length; i++)
		{
        	System.out.println(num[i]);
		}
        
	}
}
```

