---
title: Volley扩展——文件断点下载（支持下载进度）
id: 62
categories:
  - 未分类
date: 2016-09-13 14:52:01
tags:
---

volley同样不适合大文件的下载，只能自己扩展了，我这里参考了Netroid,有兴趣的可以去分析下,文件下载模仿了图片的加载方式，添加了两个类 分别是FileDownloadRequest和FileDownloader。断点续传是根据http头里的Range 和Content-Range实现.

Range用于请求头中，指定第一个字节的位置和最后一个字节的位置，一般格式：

Range:(unit=first byte pos)-[last byte pos]

Content-Range用于响应头，指定整个实体中的一部分的插入位置，他也指示了整个实体的长度。在服务器向客户返回一个部分响应，它必须描述响应覆盖的范围和整个实体长度。一般格式：

Content-Range: bytes (unit first byte pos) - [last byte pos]/[entity legth]

例如：

Range: bytes=0-801       一般下载整个文件是 0-，没有后边的801

Content-Range: bytes 2753525-41526127/41526128        斜杠后边的是文件总大小

关键代码：
<pre class="html">public Map&lt;String, String&gt; getHeaders() throws AuthFailureError {
		Map&lt;String,String&gt; map = new HashMap&lt;String,String&gt;();
		map.put("Range", "bytes=" + mTemporaryFile.length() + "-");
		return map;
	}</pre>
&nbsp;
<pre class="html">public byte[] handleResponse(HttpResponse response, ResponseDelivery delivery) throws IOException, ServerError {
		// Content-Length might be negative when use HttpURLConnection because it default header Accept-Encoding is gzip,
		// we can force set the Accept-Encoding as identity in prepare() method to slove this problem but also disable gzip response.
		HttpEntity entity = response.getEntity();
		long fileSize = entity.getContentLength();
		if (fileSize &lt;= 0) {
			VolleyLog.d("Response doesn't present Content-Length!");
		}

		long downloadedSize = mTemporaryFile.length();
		boolean isSupportRange = HttpUtils.isSupportRange(response);
		if (isSupportRange) {
			fileSize += downloadedSize;

			// Verify the Content-Range Header, to ensure temporary file is part of the whole file.
			// Sometime, temporary file length add response content-length might greater than actual file length,
			// in this situation, we consider the temporary file is invalid, then throw an exception.
			String realRangeValue = HttpUtils.getHeader(response, "Content-Range");
			Log.e("FileDownloadRequest","response header Content-Range:" + realRangeValue);
			// response Content-Range may be null when "Range=bytes=0-"
			if (!TextUtils.isEmpty(realRangeValue)) {
				String assumeRangeValue = "bytes " + downloadedSize + "-" + (fileSize - 1);
				Log.e("FileDownloadRequest","assumeRangeValue:" + assumeRangeValue);
				if (TextUtils.indexOf(realRangeValue, assumeRangeValue) == -1) {
					throw new IllegalStateException(
							"The Content-Range Header is invalid Assume[" + assumeRangeValue + "] vs Real[" + realRangeValue + "], " +
									"please remove the temporary file [" + mTemporaryFile + "].");
				}
			}
		}

		// Compare the store file size(after download successes have) to server-side Content-Length.
		// temporary file will rename to store file after download success, so we compare the
		// Content-Length to ensure this request already download or not.
		if (fileSize &gt; 0 &amp;&amp; mStoreFile.length() == fileSize) {
			// Rename the store file to temporary file, mock the download success. ^_^
			mStoreFile.renameTo(mTemporaryFile);

			// Deliver download progress.
			delivery.postProgress(this, fileSize, fileSize);
			return null;
		}

		RandomAccessFile tmpFileRaf = new RandomAccessFile(mTemporaryFile, "rw");

		// If server-side support range download, we seek to last point of the temporary file.
		if (isSupportRange) {
			tmpFileRaf.seek(downloadedSize);
		} else {
			// If not, truncate the temporary file then start download from beginning.
			tmpFileRaf.setLength(0);
			downloadedSize = 0;
		}

		try {
			InputStream in = entity.getContent();
			// Determine the response gzip encoding, support for HttpClientStack download.
			if (HttpUtils.isGzipContent(response) &amp;&amp; !(in instanceof GZIPInputStream)) {
				in = new GZIPInputStream(in);
			}
			byte[] buffer = new byte[6 * 1024]; // 6K buffer
			int offset;

			while ((offset = in.read(buffer)) != -1) {
				tmpFileRaf.write(buffer, 0, offset);

				downloadedSize += offset;
				delivery.postProgress(this, fileSize, downloadedSize);

				if (isCanceled()) {
					delivery.postCancel(this);
					break;
				}
			}
		} finally {
			try {
				// Close the InputStream and release the resources by "consuming the content".
				if (entity != null) entity.consumeContent();
			} catch (Exception e) {
				// This can happen if there was an exception above that left the entity in
				// an invalid state.
				VolleyLog.v("Error occured when calling consumingContent");
			}
			tmpFileRaf.close();
		}

		return null;
	}</pre>
demo连接：[http://download.csdn.net/detail/s569646547/9098291](http://download.csdn.net/detail/s569646547/9098291)