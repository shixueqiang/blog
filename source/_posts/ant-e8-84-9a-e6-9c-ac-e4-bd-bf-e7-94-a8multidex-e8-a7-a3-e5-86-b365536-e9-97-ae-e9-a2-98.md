---
title: ant脚本使用multidex解决65536问题
id: 94
categories:
  - android
  - ant
date: 2017-08-06 17:46:31
tags:
---

<span style="font-size: 18px;">现在的android项目应该大多都用gradle构建了吧，但是仍然有很多老项目使用的ant工具，这里并不推荐使用ant构建，因为最新的android sdk tools里边已经去掉了ant相关的lib包。不管gradle也好，ant也好，其实编译打包apk的过程基本都是一样的。</span>

<span style="font-size: 18px;">我遇到的这个项目，经历了两次方案的调整。</span>

<span style="font-size: 18px;">方案一：</span>

<span style="font-size: 18px;">最开始并没用dx的multidex参数，而是将所有的第三方jar包（应用启动时用的除外）打到从包中去，从包可以是多个，剩下的源码打到主包中，也就是分别调用dx打出dex包。</span>
<pre class="html"><span style="font-size: 14px;">&lt;macrodef name="dex-helper"&gt;
        &lt;element name="external-libs" optional="yes" /&gt;
        &lt;attribute name="nolocals" default="false" /&gt;
        &lt;sequential&gt;
            &lt;!-- sets the primary input for dex. If a pre-dex task sets it to
                 something else this has no effect --&gt;
            &lt;property name="out.dex.input.absolute.dir" value="${out.classes.absolute.dir}" /&gt;
            &nbsp;
            &lt;!-- set the secondary dx input: the project (and library) jar files
                 If a pre-dex task sets it to something else this has no effect --&gt;
            &lt;if&gt;
                &lt;condition&gt;
                    &lt;isreference refid="out.dex.jar.input.ref" /&gt;
                &lt;/condition&gt;
                &lt;else&gt;
                    &lt;path id="out.dex.jar.input.ref"&gt;
                        &lt;path refid="project.all.jars.path" /&gt;
                    &lt;/path&gt;
                &lt;/else&gt;
            &lt;/if&gt;
            &lt;echo&gt;Converting external libraries into ${assets}/${dex}...&lt;/echo&gt;
	    &lt;delete file="${asset.absolute.dir}\plug.jar"/&gt;
	    &lt;delete file="${asset.absolute.dir}\plug2.jar"/&gt;
	    &lt;if condition="${proguard.enabled}"&gt;
            	&lt;then&gt;
            	    &lt;separete 
            	        inputjar="${out.dex.input.absolute.dir}"
            	        mapping="${obfuscate.absolute.dir}/mapping.txt"
            	        alljarpath="project.all.jars.path"
			plugjarconfig="${plug.jar.config}"
			plugjarconfig2="${plug.jar.config2}"
			plugjarpath="project.plug.path"
			plugjarpath2="project.plug.path2"
			projectjarpath="project.core.path"/&gt;
            	    &lt;path id="project.dex.core.path"&gt;
                        &lt;path refid="project.core.path" /&gt;
                    &lt;/path&gt;
                &lt;/then&gt;
            	&lt;else&gt;
            		&lt;filterlib </span>				
                                alljarpath="project.all.jars.path"
			   	plugjarconfig="${plug.jar.config}"
			   	plugjarconfig2="${plug.jar.config2}"
			    	plugjarpath="project.plug.path"
			    	plugjarpath2="project.plug.path2"
			    	projectjarpath="project.core.path"/&gt;
                     	&lt;path id="project.dex.core.path"&gt;
                         &lt;path path="${out.dex.input.absolute.dir}"/&gt;
	              		 &lt;path refid="project.core.path" /&gt;
                   	 &lt;/path&gt;
                &lt;/else&gt;
	   &lt;/if&gt;
            &nbsp;
            &lt;dex executable="${dx}"
                    output="${asset.absolute.dir}\plug.jar"
                  	dexedlibs="${out.dexed.absolute.dir}"
                    nolocals="@{nolocals}"
                    forceJumbo="${dex.force.jumbo}"
                    disableDexMerger="${dex.disable.merger}"
                    verbose="${verbose}"&gt;
                &lt;path refid="project.plug.path" /&gt;
            &lt;/dex&gt;
            &lt;dex executable="${dx}"
                    output="${asset.absolute.dir}\plug2.jar"
                  	dexedlibs="${out.dexed.absolute.dir}"
                    nolocals="@{nolocals}"
                    forceJumbo="${dex.force.jumbo}"
                    disableDexMerger="${dex.disable.merger}"
                    verbose="${verbose}"&gt;
                &lt;path refid="project.plug.path2" /&gt;
            &lt;/dex&gt;
            &lt;delete file="${asset.absolute.dir}\plug.jar.d"/&gt;
            &lt;delete file="${asset.absolute.dir}\plug2.jar.d"/&gt;
	    &lt;checksum file="${asset.absolute.dir}\plug.jar" forceOverwrite="yes"/&gt;
	    &lt;checksum file="${asset.absolute.dir}\plug2.jar" forceOverwrite="yes"/&gt;
	    &lt;dex executable="${dx}"
                    output="${intermediate.dex.file}"
                    dexedlibs="${out.dexed.absolute.dir}"
                    nolocals="@{nolocals}"
                    forceJumbo="${dex.force.jumbo}"
                    disableDexMerger="${dex.disable.merger}"
                    verbose="${verbose}"&gt;
	              &lt;path refid="project.dex.core.path" /&gt;
                &lt;external-libs /&gt;
            &lt;/dex&gt;
        &nbsp;
        &lt;/sequential&gt;
    &lt;/macrodef&gt;</span></pre>
<span style="font-size: 18px;">其中separete和filterlib是两个自定义的task，作用都是从project.all.jars.path中过滤出事先定义好的第三方jar包的path，区别是separete是在代码经过混淆后执行，将混淆后的代码生成plug.jar，可以看到调用了三次dex。</span>

<span style="font-size: 18px;">代码中加载从包的方法：</span>
<pre class="html"><span style="font-size: 14px;">	PathClassLoader pathClassLoader = (PathClassLoader) app.getClassLoader();
	DexClassLoader dexClassLoader = new DexClassLoader(libPath, app.getDir("dex", 0).getAbsolutePath(), libPath, app.getClassLoader());
	InjectResult result = null;
	try {
		Object dexElements = combineArray(getDexElements(getPathList(pathClassLoader)), getDexElements(getPathList(dexClassLoader)));
		Object pathList = getPathList(pathClassLoader);
		setField(pathList, pathList.getClass(), "dexElements", dexElements);
	} catch (IllegalArgumentException e) {
		result = makeInjectResult(false, e);
		e.printStackTrace();
	}</span></pre>
<span style="font-size: 18px;">
DexClassLoader加载从包dex，然后将pathClassLoader与dexClassLoader中的<span style="color: #660066; font-family: 'Source Code Pro',monospace; white-space: pre;">DexPathList的</span>dexElements属性值合并，再放到pathClassLoader中的<span style="color: #660066; font-family: 'Source Code Pro',monospace; white-space: pre;">DexPathList</span>中。最开始方案一是满足需求的，但随着主包越来越大，终于还是65536了，这种只分出去第三方jar包的做法还是不太灵活，于是有了方案二。</span>

<span style="font-size: 18px;">方案二：</span>

<span style="font-size: 18px;">使用dx的multidex参数，指定maindexlist清单文件，在这个清单文件中的类会打到主包中，清单文件的生成可以使用build-tools下的mainDexClasses脚本生成。</span>

&nbsp;
<pre class="html"><span style="font-size: 14px;"> &lt;macrodef name="dex-helper" &gt;
		&lt;element name="external-libs" optional="yes" /&gt;
        &lt;attribute name="nolocals" default="false" /&gt;
        &lt;sequential&gt;
            &lt;!-- sets the primary input for dex. If a pre-dex task sets it to
                 something else this has no effect --&gt;
            &lt;property name="out.dex.input.absolute.dir" value="${out.classes.absolute.dir}" /&gt;
			 &lt;if&gt;
                &lt;condition&gt;
                    &lt;isreference refid="out.dex.jar.input.ref" /&gt;
                &lt;/condition&gt;
                &lt;else&gt;
                    &lt;path id="out.dex.jar.input.ref"&gt;
                        &lt;path refid="project.all.jars.path" /&gt;
                    &lt;/path&gt;
                &lt;/else&gt;
            &lt;/if&gt;
            &lt;echo message="start dx ${maindexlist.dir}"/&gt;
            &lt;multidex executable="${dx}"
                    output="${out.dir}"
                    dexedlibs="${out.dexed.absolute.dir}"
                    nolocals="@{nolocals}"
                    forceJumbo="${dex.force.jumbo}"
                    disableDexMerger="${dex.disable.merger}"
                    multidex="true"
                    mainDexList="${maindexlist.dir}"
                    verbose="${verbose}"&gt;
	            &lt;path path="${out.dex.input.absolute.dir}"/&gt;  
                &lt;path refid="out.dex.jar.input.ref" /&gt;  
                &lt;external-libs /&gt;
            &lt;/multidex&gt;
        &lt;/sequential&gt;
    &lt;/macrodef&gt;</span><span style="font-size: 14px;">&lt;target name="-maindexlist" depends="-set-release-mode, -compile, -post-compile, -obfuscate"&gt;
        &lt;property name="out.dex.input.absolute.dir" value="${out.classes.absolute.dir}" /&gt;
        &nbsp;
        &lt;!-- set the secondary dx input: the project (and library) jar files
             If a pre-dex task sets it to something else this has no effect --&gt;
        &lt;if&gt;
            &lt;condition&gt;
                &lt;isreference refid="out.dex.jar.input.ref" /&gt;
            &lt;/condition&gt;
            &lt;else&gt;
                &lt;path id="out.dex.jar.input.ref"&gt;
                    &lt;path refid="project.all.jars.path" /&gt;
                &lt;/path&gt;
            &lt;/else&gt;
        &lt;/if&gt;
        &lt;maindexlist output="build/maindexlist-proguard.txt"
                mainDexList="build/maindexlist.txt"
                mappingFile="${out.absolute.dir}/proguard/mapping.txt" &gt;
            &lt;path path="${out.dex.input.absolute.dir}"/&gt;
            &lt;path refid="out.dex.jar.input.ref" /&gt;
        &lt;/maindexlist&gt;
    &lt;/target&gt;</span></pre>
<span style="font-size: 18px;">m</span><span style="font-size: 18px;">ultidex和maindexlist也是两个自定义task，multidex其实就是调用的dx --dex --multi-dex --main-dex-list=maindexlist.txt  --minimal-main-dex --output bin --input-list,maindexlist task的作用是生成混淆后的主包清单文件。应用启动时代码原理和方案一差不多，也可以直接使用官方的android-surpport-multidex.jar包</span>