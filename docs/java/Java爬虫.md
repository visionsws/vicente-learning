## Java爬虫

## 使用场景

先定义一个最简单的使用场景，给你一个url，把这个url中指定的内容爬下来，然后停止

- 一个待爬去的网址（有个地方指定爬的网址）
- 如何获取指定的内容（可以配置规则来获取指定的内容）

### 基本数据结构

1. 一个配置项，包含塞入的 url 和 获取规则

```
@ToString
public class CrawlMeta {

    /**
     * 待爬去的网址
     */
    @Getter
    @Setter
    private String url;

    /**
     * 获取指定内容的规则, 因为一个网页中，你可能获取多个不同的内容， 所以放在集合中
     */
    @Setter
    private Set<String> selectorRules;


    // 这么做的目的就是为了防止NPE, 也就是说支持不指定选择规则
    public Set<String> getSelectorRules() {
        return selectorRules != null ? selectorRules : new HashSet<>();
    }

}
```

2. 抓取的结果

   ```
   @Getter
   @Setter
   @ToString
   public class CrawlResult {
   
       /**
        * 爬取的网址
        */
       private String url;
   
   
       /**
        * 爬取的网址对应的 DOC 结构
        */
       private Document htmlDoc;
   
   
       /**
        * 选择的结果，key为选择规则，value为根据规则匹配的结果
        */
       private Map<String, List<String>> result;
   
   }
   ```

   ### 爬取任务

   #### `IJob.java`

   这里定义了两个方法，在job执行之前和之后的回调，加上主要某些逻辑可以放在这里来做（如打日志，耗时统计等），将辅助的代码从爬取的代码中抽取，使代码结构更整洁

   ```
   public interface IJob extends Runnable {
   
       /**
        * 在job执行之前回调的方法
        */
       void beforeRun();
   
   
       /**
        * 在job执行完毕之后回调的方法
        */
       void afterRun();
   }
   ```

   #### `AbstractJob`

   因为`IJob` 多了两个方法，所以就衍生了这个抽象类，不然每个具体的实现都得去实现这两个方法，有点蛋疼

   然后就是借用了一丝模板设计模式的思路，把run方法也实现了，单独拎了一个`doFetchPage`方法给子类来实现，具体的抓取网页的逻辑

   ```
   public abstract class AbstractJob implements IJob {
   
       public void beforeRun() {
       }
   
       public void afterRun() {
       }
   
   
       @Override
       public void run() {
           this.beforeRun();
   
   
           try {
               this.doFetchPage();
           } catch (Exception e) {
               e.printStackTrace();
           }
   
   
           this.afterRun();
       }
   
   
       /**
        * 具体的抓去网页的方法， 需要子类来补全实现逻辑
        *
        * @throws Exception
        */
       public abstract void doFetchPage() throws Exception;
   }
   ```

   #### `SimpleCrawlJob`

   一个最简单的实现类，直接利用了JDK的URL方法来抓去网页，然后利用jsoup进行html结构解析，这个实现中有较多的硬编码，先看着，下面就着手第一步优化

   ```java
   /**
    * 最简单的一个爬虫任务
    */
   @Getter
   @Setter
   public class SimpleCrawlJob extends AbstractJob {
   
       /**
        * 配置项信息
        */
       private CrawlMeta crawlMeta;
   
   
       /**
        * 存储爬取的结果
        */
       private CrawlResult crawlResult;
   
   /**
    * 执行抓取网页
    */
   public void doFetchPage() throws Exception {
       HttpClient httpClient = HttpClients.createDefault();
       HttpGet httpGet = new HttpGet(crawlMeta.getUrl());
       HttpResponse response = httpClient.execute(httpGet);
       String res = EntityUtils.toString(response.getEntity());
       if (response.getStatusLine().getStatusCode() == 200) { // 请求成功
           doParse(res);
       } else {
           this.crawlResult = new CrawlResult();
           this.crawlResult.setStatus(response.getStatusLine().getStatusCode(), response.getStatusLine().getReasonPhrase());
           this.crawlResult.setUrl(crawlMeta.getUrl());
       }
   }
   
   
       private void doParse(String html) {
           Document doc = Jsoup.parse(html);
   
           Map<String, List<String>> map = new HashMap<>(crawlMeta.getSelectorRules().size());
           for (String rule: crawlMeta.getSelectorRules()) {
               List<String> list = new ArrayList<>();
               for (Element element: doc.select(rule)) {
                   list.add(element.text());
               }
   
               map.put(rule, list);
           }
   
   
           this.crawlResult = new CrawlResult();
           this.crawlResult.setHtmlDoc(doc);
           this.crawlResult.setUrl(crawlMeta.getUrl());
           this.crawlResult.setResult(map);
       }
   }
   ```

   ## 4. 测试

   > 上面一个最简单的爬虫就完成了，就需要拉出来看看，是否可以正常的工作了

   就拿自己的博客作为测试网址，目标是获取 title + content，所以测试代码如下

   ```
   /**
    * 测试我们写的最简单的一个爬虫,
    *
    * 目标是爬取一篇博客
    */
   @Test
   public void testFetch() throws InterruptedException {
       String url = "https://my.oschina.net/u/566591/blog/1031575";
       Set<String> selectRule = new HashSet<>();
       selectRule.add("div[class=title]"); // 博客标题
       selectRule.add("div[class=blog-body]"); // 博客正文
   
       CrawlMeta crawlMeta = new CrawlMeta();
       crawlMeta.setUrl(url); // 设置爬取的网址
       crawlMeta.setSelectorRules(selectRule); // 设置抓去的内容
   
   
       SimpleCrawlJob job = new SimpleCrawlJob();
       job.setCrawlMeta(crawlMeta);
       Thread thread = new Thread(job, "crawler-test");
       thread.start();
   
       thread.join(); // 确保线程执行完毕
   
   
       CrawlResult result = job.getCrawlResult();
       System.out.println(result);
   }
   ```

   **代码演示示意图如下**

   从返回的结果可以看出，抓取到的title中包含了博客标题 + 作着，主要的解析是使用的 jsoup，所以这些抓去的规则可以参考jsoup的使用方式

