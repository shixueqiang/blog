---
title: rename
id: 23
categories:
  - 未分类
date: 2016-07-22 17:10:07
tags:
---

Dos/Windows下，对文件改名用rename。而书上说，Linux下对文件或目录改名该用mv。我一直也是这样做的，却忽略了Linux下也有 个叫rename的命令。都是rename，但功能上就有点差异了。Linux下的rename更像批量改名的工具，是util-linux套件中提供 的。

**一、基本功能**
从mv和rename命令的man文档中，可以看到如下信息：
<div class="quote">
<div class="quote-title">引用</div>
<div class="quote-content">mv - move (rename) files
rename - Rename files</div>
</div>
也就是说，mv也能用于改名，但不能实现批量处理（改名时，不支持*等符号的），而rename可以。
rename使用的格式：
<div class="code">$ rename foo foo0 foo?</div>
rename需要提供三个参数，然后才能决定最终结果。
模拟一下man文档的例子，原文件：
<div class="quote">
<div class="quote-title">引用</div>
<div class="quote-content">$ for i in `seq 100`;do touch foo$i;done
$ ls
foo1    foo18  foo27  foo36  foo45  foo54  foo63  foo72  foo81  foo90
foo10   foo19  foo28  foo37  foo46  foo55  foo64  foo73  foo82  foo91
foo100  foo2   foo29  foo38  foo47  foo56  foo65  foo74  foo83  foo92
foo11   foo20  foo3   foo39  foo48  foo57  foo66  foo75  foo84  foo93
foo12   foo21  foo30  foo4   foo49  foo58  foo67  foo76  foo85  foo94
foo13   foo22  foo31  foo40  foo5   foo59  foo68  foo77  foo86  foo95
foo14   foo23  foo32  foo41  foo50  foo6   foo69  foo78  foo87  foo96
foo15   foo24  foo33  foo42  foo51  foo60  foo7   foo79  foo88  foo97
foo16   foo25  foo34  foo43  foo52  foo61  foo70  foo8   foo89  foo98
foo17   foo26  foo35  foo44  foo53  foo62  foo71  foo80  foo9   foo99</div>
</div>
改名结果：
（红色是没有改动的，蓝色是有改动的一部分）
<div class="quote">
<div class="quote-title">引用</div>
<div class="quote-content">$ rename foo foo0 foo?
$ ls
foo01foo100foo20  foo30  foo40  foo50  foo60  foo70  foo80  foo90
foo02  foo11   foo21  foo31  foo41  foo51  foo61  foo71  foo81  foo91
foo03  foo12   foo22  foo32  foo42  foo52  foo62  foo72  foo82  foo92
foo04  foo13   foo23  foo33  foo43  foo53  foo63  foo73  foo83  foo93
foo05  foo14   foo24  foo34  foo44  foo54  foo64  foo74  foo84  foo94
foo06  foo15   foo25  foo35  foo45  foo55  foo65  foo75  foo85  foo95
foo07  foo16   foo26  foo36  foo46  foo56  foo66  foo76  foo86  foo96
foo08  foo17   foo27  foo37  foo47  foo57  foo67  foo77  foo87  foo97
foo09  foo18   foo28  foo38  foo48  foo58  foo68  foo78  foo88  foo98
foo10  foo19   foo29  foo39  foo49  foo59  foo69  foo79  foo89  foo99
$ rename foo foo0 foo??
$ ls
foo001  foo011  foo021  foo031  foo041  foo051  foo061  foo071  foo081  foo091
foo002  foo012  foo022  foo032  foo042  foo052  foo062  foo072  foo082  foo092
foo003  foo013  foo023  foo033  foo043  foo053  foo063  foo073  foo083  foo093
foo004  foo014  foo024  foo034  foo044  foo054  foo064  foo074  foo084  foo094
foo005  foo015  foo025  foo035  foo045  foo055  foo065  foo075  foo085  foo095
foo006  foo016  foo026  foo036  foo046  foo056  foo066  foo076  foo086  foo096
foo007  foo017  foo027  foo037  foo047  foo057  foo067  foo077  foo087  foo097
foo008  foo018  foo028  foo038  foo048  foo058  foo068  foo078  foo088  foo098
foo009  foo019  foo029  foo039  foo049  foo059  foo069  foo079  foo089  foo099
foo010  foo020  foo030  foo040  foo050  foo060  foo070  foo080  foo090  foo100</div>
</div>
该例子给出了两种文件批量重命名的用法：
<div class="quote">
<div class="quote-title">引用</div>
<div class="quote-content">第一个参数：被替换掉的字符串
第二个参数：替换成的字符串
第三个参数：匹配要替换的文件模式</div>
</div>
rename支持通配符，基本的通配符有以下几个：
<div class="quote">
<div class="quote-title">引用</div>
<div class="quote-content">?    可替代单个字符
*    可替代多个字符
[charset]    可替代charset集中的任意单个字符</div>
</div>
**二、其他例子**
看看*的作用：
<div class="quote">
<div class="quote-title">引用</div>
<div class="quote-content">$ rm -f *
$ for i in `seq 100`;do touch foo$i;done
$ rename foo foo0 foo*
$ ls
foo01    foo018  foo027  foo036  foo045  foo054  foo063  foo072  foo081  foo090
foo010   foo019  foo028  foo037  foo046  foo055  foo064  foo073  foo082  foo091
foo0100  foo02   foo029  foo038  foo047  foo056  foo065  foo074  foo083  foo092
foo011   foo020  foo03   foo039  foo048  foo057  foo066  foo075  foo084  foo093
foo012   foo021  foo030  foo04   foo049  foo058  foo067  foo076  foo085  foo094
foo013   foo022  foo031  foo040  foo05   foo059  foo068  foo077  foo086  foo095
foo014   foo023  foo032  foo041  foo050  foo06   foo069  foo078  foo087  foo096
foo015   foo024  foo033  foo042  foo051  foo060  foo07   foo079  foo088  foo097
foo016   foo025  foo034  foo043  foo052  foo061  foo070  foo08   foo089  foo098
foo017   foo026  foo035  foo044  foo053  foo062  foo071  foo080  foo09   foo099</div>
</div>
再看看[charset]的作用：
<div class="quote">
<div class="quote-title">引用</div>
<div class="quote-content">$ rm -f *
$ for i in `seq 100`;do touch foo$i;done
$ rename foo foo0 foo[9]*
$ ls
foo09   foo099  foo17  foo26  foo35  foo44  foo53  foo62  foo71  foo80
foo090  foo1    foo18  foo27  foo36  foo45  foo54  foo63  foo72  foo81
foo091  foo10   foo19  foo28  foo37  foo46  foo55  foo64  foo73  foo82
foo092  foo100  foo2   foo29  foo38  foo47  foo56  foo65  foo74  foo83
foo093  foo11   foo20  foo3   foo39  foo48  foo57  foo66  foo75  foo84
foo094  foo12   foo21  foo30  foo4   foo49  foo58  foo67  foo76  foo85
foo095  foo13   foo22  foo31  foo40  foo5   foo59  foo68  foo77  foo86
foo096  foo14   foo23  foo32  foo41  foo50  foo6   foo69  foo78  foo87
foo097  foo15   foo24  foo33  foo42  foo51  foo60  foo7   foo79  foo88
foo098  foo16   foo25  foo34  foo43  foo52  foo61  foo70  foo8   foo89</div>
</div>