---
title: 使用glide填过的各种坑
id: 76
categories:
  - 未分类
date: 2017-01-22 13:43:49
tags:
---

使用版本glide-3.7.0

坑一：

无法手动刷新缓存，只能改变key添加自定义签名，我是将头像版本号放到key中了，因为取头像是根据手机号取的，头像url路径并不会变，而且首次加载时并不能拿到版本号。

builder.signature(new StringSignature(etag));

坑二：

从磁盘缓存加载头像时太慢，导致能看到由默认图变到头像的过程，给人感觉头像闪了一下，体验不好。因为glide的磁盘缓存加载逻辑放到了工作线程中，提高了滑动时的流畅度，但是也降低了加载速度，没想到好办法解决;

坑三：

服务器返回的是xml数据，需要解析完xml才能获取头像byte[],好在glide有现成的api

builder.asBitmap().imageDecoder(decoder)
<pre class="html">public class XmlBitmapDecoder implements ResourceDecoder&lt;InputStream, Bitmap&gt; {
	private static final String ID = "XmlBitmapDecoder.com.chinamobile.rcs.contacts.glide";
	private String id;
	private final BitmapPool bitmapPool;
	&nbsp;
	public XmlBitmapDecoder(BitmapPool bitmapPool) {
		this.bitmapPool = bitmapPool;
	}
	&nbsp;
	@Override
	public Resource&lt;Bitmap&gt; decode(InputStream is, int width, int height) throws IOException {
		AvatarModel avatarModel = Utils.stream2Avatar(is);
		byte[] datas = avatarModel.getData();
		Bitmap bitmap = BitmapFactory.decodeByteArray(datas, 0, datas.length);
		return BitmapResource.obtain(bitmap,bitmapPool);
	}
	&nbsp;
	@Override
	public String getId() {
		if (this.id == null) {
			this.id = ID;
		}
		return this.id;
	}
	&nbsp;
}</pre>
坑四：

消息列表中因为有不同的消息类型，默认头像也不一样，glide占位图不设置时，会设置null，所以onLoadStarted方法里需要区分开；

坑五：

token验证，切圆角图，这个相对容易实现

坑六：

某些场景需要跳过缓存强制从网络拉取，但是需要先用缓存的头像当占位图，这点上glide没有开放现成的api，实现方式有点迂回：
<pre class="html">DrawableTypeRequest&lt;MyGlideUrl&gt; builder = Glide.with(mContext).using(new StreamModelLoader&lt;MyGlideUrl&gt;() {
			&nbsp;
			@Override
			public DataFetcher&lt;InputStream&gt; getResourceFetcher(final MyGlideUrl model, int arg1, int arg2) {
				return new DataFetcher&lt;InputStream&gt;() {
					@Override
					public InputStream loadData(Priority priority) throws Exception {
						throw new IOException();
						// do nothing
					}
					&nbsp;
					@Override
					public void cleanup() {
						&nbsp;
					}
					&nbsp;
					@Override
					public String getId() {
						return model.getCacheKey();
					}
					&nbsp;
					@Override
					public void cancel() {
						&nbsp;
					}
				};
			}
		}).load(glideUrl);</pre>
重点是loadData里边不做任何事，正常情况下这里本该写从网络拉取头像获取输入流的逻辑。这样就能变相实现从缓存拿到bitmap。

坑七：

还是加载慢的问题，但是有时候从memory缓存里加载竟然也需要50，60ms，打开glide调试，发现从缓存加载只需要零点几毫秒，但是从我代码调用到真正发起请求竟然用了大部分时间，有人说是得到view宽高后才会发起请求，于是添加.override(w, h)，效果不明显。

自己手动改了改，增加了在缓存中删除单个key的方法