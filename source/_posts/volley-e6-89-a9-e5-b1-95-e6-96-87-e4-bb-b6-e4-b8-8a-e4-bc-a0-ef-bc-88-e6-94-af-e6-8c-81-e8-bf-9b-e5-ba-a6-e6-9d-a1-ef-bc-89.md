---
title: Volley扩展——文件上传（支持进度条）
id: 60
categories:
  - 未分类
date: 2016-09-13 14:51:32
tags:
---

volley是一个轻量级的开源网络通信框架，开源的好处就是可以自由定制自己需要的jar包。volley里网络通信时android2.3以上用的 HttpUrlConnection,2.3以下用的HttpClient，我做的改动只考虑了2.3以上，不支持2.3版本以下。 HttpUrlConnection默认传输数据是将数据全部写到内存中再发送到服务端，Volley就是采用默认的方式，这样在上传大文件时很容易就 out of memory，有一种解决办法是设置每次传输流的大小：

已知文件大小：connection .setFixedLengthStreamingMode（long l）;

不知道文件大小：connection.setChunkedStreamingMode(1024); //建议使用

android的文件上传一般都是模拟表单，也可以直接socket传，我这里是集成了表单上传，下面是关键类：
<pre class="java">public class MultipartRequest extends Request&lt;String&gt; {
    private final Listener&lt;String&gt; mListener;
    private Map&lt;String, String&gt; headerMap;
    private Map&lt;String, String&gt; mParams;
    private FormFile[] files;
    private String BOUNDARY = "---------7dc05dba8f3e19";

    public MultipartRequest(String url, Listener&lt;String&gt; listener, Map&lt;String, String&gt; params, FormFile[] files) {
        this(Method.POST, url, listener, params, files);
    }

    public MultipartRequest(int method, String url, Listener&lt;String&gt; listener, Map&lt;String, String&gt; params, FormFile[] files) {
        super(method, url, listener);
        mListener = listener;
        mParams = params;
        this.files = files;
    }

    @Override
    public Map&lt;String, String&gt; getHeaders() throws AuthFailureError {
        headerMap = new HashMap&lt;String, String&gt;();
        headerMap.put("Charset", "UTF-8");
        //Keep-Alive
        headerMap.put("Connection", "Keep-Alive");
        headerMap.put("Content-Type", "multipart/form-data; boundary=" + BOUNDARY);
        return headerMap;
    }

    @Override
    public byte[] getBody() throws AuthFailureError {
        //传参数
        StringBuilder sb = new StringBuilder();
        for (Map.Entry&lt;String, String&gt; entry : mParams.entrySet()) {
            // 构建表单字段内容
            sb.append("--");
            sb.append(BOUNDARY);
            sb.append("\r\n");
            sb.append("Content-Disposition: form-data; name=\"" + entry.getKey() + "\"\r\n\r\n");
            sb.append(entry.getValue());
            sb.append("\r\n");
        }
        return sb.toString().getBytes();
    }

    @Override
    public void handRequest(OutputStream out) {
        DataOutputStream dos = (DataOutputStream) out;
        try {
            //发送文件数据
            if (files != null) {
                for (FormFile file : files) {
                    // 发送文件数据
                    StringBuilder split = new StringBuilder();
                    split.append("--");
                    split.append(BOUNDARY);
                    split.append("\r\n");
                    split.append("Content-Disposition: form-data;name=\"" + file.getParameterName() + "\";filename=\"" + file.getFilname() + "\"\r\n");
                    split.append("Content-Type: " + file.getContentType() + "\r\n\r\n");
                    dos.write(split.toString().getBytes());
                    if (file.getInStream() != null) {
                        byte[] buffer = new byte[1024];
                        int len = -1;
                        int count = 0;
                        while ((len = file.getInStream().read(buffer)) != -1) {
                            dos.write(buffer, 0, len);
                            count += len;
                            if (mListener != null) {
                                mListener.onProgressChange(file.getFileSize(), count);
                            }
                        }
                        count = 0;
                        file.getInStream().close();
                    } else {
                        dos.write(file.getData(), 0, file.getData().length);
                    }
                    dos.write("\r\n".getBytes());
                }
            }
            dos.writeBytes("--" + BOUNDARY + "--\r\n");
            dos.flush();
        } catch (IOException e) {
            mListener.onError(new VolleyError(e.toString()));
            try {
                dos.close();
            } catch (IOException e1) {
                e1.printStackTrace();
            }
        }
    }

    @Override
    protected Response&lt;String&gt; parseNetworkResponse(NetworkResponse response) {
        String parsed;
        try {
            parsed = new String(response.data, HttpHeaderParser.parseCharset(response.headers));
        } catch (UnsupportedEncodingException e) {
            parsed = new String(response.data);
        }
        return Response.success(parsed, HttpHeaderParser.parseCacheHeaders(response));
    }

    @Override
    protected void deliverResponse(String response) {
        mListener.onSuccess(response);
    }

    @Override
    public void deliverError(VolleyError error) {
        mListener.onError(error);
    }
}</pre>
附上demo连接：[http://download.csdn.net/detail/s569646547/9098291](http://download.csdn.net/detail/s569646547/9098291)