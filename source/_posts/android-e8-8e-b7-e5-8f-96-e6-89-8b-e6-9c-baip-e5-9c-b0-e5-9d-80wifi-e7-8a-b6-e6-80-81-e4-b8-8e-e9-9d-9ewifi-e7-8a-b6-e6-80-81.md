---
title: android获取手机ip地址wifi状态与非wifi状态
id: 54
categories:
  - 未分类
date: 2016-09-13 14:50:08
tags:
---

<span style="font-size: large;">方法一（只适合wifi状态）：</span>
<pre class="java">public static String getIp(Context contxext) {
		WifiManager wm = (WifiManager) contxext.getSystemService(Context.WIFI_SERVICE);
		// 检查Wifi状态
		if (!wm.isWifiEnabled())
			wm.setWifiEnabled(true);
		WifiInfo wi = wm.getConnectionInfo();
		int ipAdd = wi.getIpAddress();
		String ip = intToIp(ipAdd);
		return ip;
	}</pre>
<span style="font-size: large;"> </span>
<pre class="java">public static String intToIp(int i) {
		return (i &amp; 0xFF) + "." + ((i &gt;&gt; 8) &amp; 0xFF) + "." + ((i &gt;&gt; 16) &amp; 0xFF)
				+ "." + (i &gt;&gt; 24 &amp; 0xFF);
	}</pre>
<span style="font-size: large;">
方法二（通用ipv4地址）：</span>
<pre class="java">private String getIp() {
        try {
            for (Enumeration&lt;NetworkInterface&gt; en = NetworkInterface.getNetworkInterfaces(); en.hasMoreElements(); ) {
                NetworkInterface intf = en.nextElement();
                for (Enumeration&lt;InetAddress&gt; enumIpAddr = intf.getInetAddresses(); enumIpAddr.hasMoreElements(); ) {
                    InetAddress inetAddress = enumIpAddr.nextElement();
                    if (!inetAddress.isLoopbackAddress() &amp;&amp; inetAddress instanceof Inet4Address) {
                        return inetAddress.getHostAddress();
                    }
                }
            }
        } catch (SocketException ex) {
            ex.printStackTrace();
        }
        return null;
    }</pre>
<span style="font-size: large;">
方法三（通用ipv4与ipv6）
</span>
<pre class="java">/**
     * Get IP address from first non-localhost interface
     *
     * @param useIPv4 true=return ipv4, false=return ipv6
     * @return address or empty string
     */
    public static String getIPAddress(boolean useIPv4) {
        try {
            List&lt;NetworkInterface&gt; interfaces = Collections.list(NetworkInterface.getNetworkInterfaces());
            for (NetworkInterface intf : interfaces) {
                List&lt;InetAddress&gt; addrs = Collections.list(intf.getInetAddresses());
                for (InetAddress addr : addrs) {
                    if (!addr.isLoopbackAddress()) {
                        String sAddr = addr.getHostAddress().toUpperCase();
                        boolean isIPv4 = InetAddressUtils.isIPv4Address(sAddr);
                        if (useIPv4) {
                            if (isIPv4)
                                return sAddr;
                        } else {
                            if (!isIPv4) {
                                int delim = sAddr.indexOf('%'); // drop ip6 port suffix
                                return delim &lt; 0 ? sAddr : sAddr.substring(0, delim);
                            }
                        }
                    }
                }
            }
        } catch (Exception ex) {
        }
        return "";
    }</pre>
&nbsp;