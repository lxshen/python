<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-generate-toc again -->
**Table of Contents**
         
* [Nginx服务器](#nginx服务器)
	* [什么是Nginx?](#什么是nginx)
	* [负载均衡](#负载均衡)
* [算法](#算法)
	* [布隆去重](#1-布隆去重)
     
<!-- markdown-toc end -->
      
# Nginx服务器
## 什么是nginx

nginx是一个高性能的WEB和方向代理服务器，正向代理：用户去访问一个代理服务器，这个代理服务器去帮我们访问我们访问不了的网站内容
    负载均衡：负载均衡是多台服务器以对称的方式组成一个服务器集合，每台服务器都具有等价的地位，都可以单独对外提供服务而无须其它服务器的辅助。
    通过某种负载分担技术，将外部发送来的请求按照事先设定分配算法分配到对称结构中的某一台服务器上，而接收到请求的服务器独立地回应客户的请求
   
   作用：均衡负载能够平均分配客户请求到服务器列阵，藉此提供快速获取重要数据，解决大量并发访问服务器问题

## 负载均衡

> 算法有四种：
>
> * 1、weight轮询（默认）每个请求按时间循序逐一分配到不同的后端服务器，如果后端某台服务器宕机，故障系统被自动删除，使用户访问不受影响weight,指定轮询权值，weigt越大，分配到的访问几率越高，主要用于后端每个服务器性能不均的情况下
> * 2、ip_hash：每个请求按访问ip的hash结果分配，这样用于同一个ip的访问固定访问一个后端服务器，有效解决了动态网页存在的session共享问题
> * 3、fair:比上面两个更加智能的负载均衡算法，此种算法可以依据页面大小和加载时间长短智能地运行进行负载均衡，也就是根据后端服务器的响应时间来分配请求，响应时间短的优化分配。以nginx本身不支持fair的，如果选哟使用这个调度算法，必须下载nginx的upstream_fair模块
> * 4、url_hash:按照访问url(统一资源定位器)的hash结果来分配，使每个url定向到同一个后端服务器，可以进一步提高后端缓存服务器的效率。

nginx本身不支持url_hash的，如果需要使用这种调度算法，必须安装nginx的hash软件包


# 算法
## 1-布隆去重
<h3 id="articleHeader3"><a name="t4"></a>原理</h3>
<p>如果想判断一个元素是不是在一个集合里，一般想到的是将集合中所有元素保存起来，然后通过比较确定。链表、树、散列表（又叫哈希表，Hash table）等等数据结构都是这种思路。但是随着集合中元素的增加，我们需要的存储空间越来越大。同时检索速度也越来越慢。</p>
<p>Bloom Filter 是一种空间效率很高的随机数据结构，Bloom filter 可以看做是对 bit-map 的扩展, 它的原理是：</p>
<p>当一个元素被加入集合时，通过 <code>K</code> 个 <code>Hash 函数</code>将这个元素映射成一个<code>位阵列（Bit array）中的 K 个点</code>，把它们置为<code>1</code>。检索时，我们只要看看这些点是不是都是 1 就（大约）知道集合中有没有它了：</p>
<ul><li>如果这些点有任何一个 0，则被检索元素<strong>一定不在</strong>；</li><li>如果都是 1，则被检索元素<strong>很可能</strong>在。</li></ul><h3 id="articleHeader4"><a name="t5"></a>优点</h3>
<blockquote>
<p>It tells us that the element either definitely is not in the set or may be in the set.</p>
</blockquote>
<p>它的优点是<code>空间效率</code>和<code>查询时间</code>都远远超过一般的算法，布隆过滤器存储空间和插入 / 查询时间都是常数<code>O(k)</code>。另外, 散列函数相互之间没有关系，方便由硬件并行实现。布隆过滤器不需要存储元素本身，在某些对保密要求非常严格的场合有优势。</p>
<h3 id="articleHeader5"><a name="t6"></a>缺点</h3>
<p>但是布隆过滤器的缺点和优点一样明显。误算率是其中之一。随着存入的元素数量增加，<code>误算率</code>随之增加。但是如果元素数量太少，则使用散列表足矣。</p>
<p>(误判补救方法是：再建立一个小的白名单，存储那些可能被误判的信息。)</p>
<p>另外，一般情况下不能从布隆过滤器中<code>删除</code>元素. 我们很容易想到把位数组变成整数数组，每插入一个元素相应的计数器加 1, 这样删除元素时将计数器减掉就可以了。然而要保证安全地删除元素并非如此简单。首先我们必须保证删除的元素的确在布隆过滤器里面. 这一点单凭这个过滤器是无法保证的。另外计数器回绕也会造成问题。</p>
<h3 id="articleHeader6"><a name="t7"></a>Example</h3>
<p>可以快速且空间效率高的判断一个元素是否属于一个集合；用来实现数据字典，或者集合求交集。</p>
<p>如： Google chrome 浏览器使用bloom filter识别恶意链接（能够用较少的存储空间表示较大的数据集合，简单的想就是把每一个URL都可以映射成为一个bit）<br>
得多，并且误判率在万分之一以下。<br>
又如： 检测垃圾邮件</p>
<div class="widget-codetool">
<div class="widget-codetool--inner"><span title="" class="selectCode code-tool"></span><span title="" class="copyCode code-tool"></span><span title="" class="saveToNote code-tool"></span></div>
</div>
<pre class="hljs" name="code" onclick="hljs.copyCode(event)"><code class="hljs">假定我们存储一亿个电子邮件地址，我们先建立一个十六亿二进制（比特），即两亿字节的向量，然后将这十六亿个二进制全部设置为零。对于每一个电子邮件地址 X，我们用八个不同的随机数产生器（F1,F2, ...,F8） 产生八个信息指纹（f1, f2, ..., f8）。再用一个随机数产生器 G 把这八个信息指纹映射到 1 到十六亿中的八个自然数 g1, g2, ...,g8。现在我们把这八个位置的二进制全部设置为一。当我们对这一亿个 email 地址都进行这样的处理后。一个针对这些 email 地址的布隆过滤器就建成了。
</code><div class="hljs-button" data-title="复制"></div></pre>
<p>再如此题：</p>
<div class="widget-codetool" style="display:block;">
<div class="widget-codetool--inner"><span title="" class="selectCode code-tool"></span><span title="" class="copyCode code-tool"></span><span title="" class="saveToNote code-tool"></span></div>
</div>
<pre class="hljs" name="code" onclick="hljs.copyCode(event)"><code class="hljs vbscript"><ol class="hljs-ln" style="width:2057px"><li><div class="hljs-ln-numbers"><div class="hljs-ln-line hljs-ln-n" data-line-number="1"></div></div><div class="hljs-ln-code"><div class="hljs-ln-line">A,B 两个文件，各存放 <span class="hljs-number">50</span> 亿条 URL，每条 URL 占用 <span class="hljs-number">64</span> 字节，内存限制是 <span class="hljs-number">4</span>G，让你找出 A,B 文件共同的 URL。如果是三个乃至 n 个文件呢？</div></div></li><li><div class="hljs-ln-numbers"><div class="hljs-ln-line hljs-ln-n" data-line-number="2"></div></div><div class="hljs-ln-code"><div class="hljs-ln-line"> </div></div></li><li><div class="hljs-ln-numbers"><div class="hljs-ln-line hljs-ln-n" data-line-number="3"></div></div><div class="hljs-ln-code"><div class="hljs-ln-line">分析 ：如果允许有一定的错误率，可以使用 Bloom <span class="hljs-built_in">filter</span>，<span class="hljs-number">4</span>G 内存大概可以表示 <span class="hljs-number">340</span> 亿 bit。将其中一个文件中的 url 使用 Bloom <span class="hljs-built_in">filter</span> 映射为这 <span class="hljs-number">340</span> 亿 bit，然后挨个读取另外一个文件的 url，检查是否与 Bloom <span class="hljs-built_in">filter</span>，如果是，那么该 url 应该是共同的 url（注意会有一定的错误率）。”</div></div></li></ol></code><div class="hljs-button" data-title="复制"></div></pre>
            </div>
