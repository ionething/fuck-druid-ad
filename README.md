# fuck-druid-ad
去掉druid监控页面的广告banner

> druid 1.1.4版本之后加了直接到阿里云的footer广告banner，在 druid [issues2731](https://github.com/alibaba/druid/issues/2731#issuecomment-428277842)中提过一些想法去掉烦人的广告，这里做实践

## FuckDruidAdConfiguration（推荐）

Spring Boot项目中添加下面的configuration

```java
@Configuration
@ConditionalOnWebApplication
@AutoConfigureAfter(DruidDataSourceAutoConfigure.class)
@ConditionalOnProperty(name = "spring.datasource.druid.stat-view-servlet.enabled", havingValue = "true", matchIfMissing = true)
public class FuckDruidAdConfiguration {

    @Bean
    public FilterRegistrationBean fuckDruidAdFilterRegistrationBean(DruidStatProperties properties) {
        // 获取web监控页面的参数
        DruidStatProperties.StatViewServlet config = properties.getStatViewServlet();
        // 提取common.js的配置路径
        String pattern = config.getUrlPattern() != null ? config.getUrlPattern() : "/druid/*";
        String commonJsPattern = pattern.replaceAll("\\*", "js/common.js");

        final String filePath = "support/http/resources/js/common.js";

        Filter filter = new Filter() {
            @Override
            public void init(FilterConfig filterConfig) throws ServletException {
            }

            @Override
            public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
                chain.doFilter(request, response);
                // 重置缓冲区，响应头不会被重置
                response.resetBuffer();
                // 获取common.js
                String text = Utils.readFromResource(filePath);
                // 正则替换banner
                text = text.replaceAll("<a.*?banner\"></a><br/>", "");
                response.getWriter().write(text);
            }

            @Override
            public void destroy() {
            }
        };

        FilterRegistrationBean registrationBean = new FilterRegistrationBean();
        registrationBean.setFilter(filter);
        registrationBean.addUrlPatterns(commonJsPattern);
        return registrationBean;
    }
}
```

- 原理很简单，就是使用过滤器过滤common.js的请求，重新处理后用正则替换相关的广告代码片段
- 非 Spring Boot项目可以参照自己实现一个filter
- 可以封装成一个starter，但不建议这么做

## 使用干净的js

1. 下载干净的[common.js](https://raw.githubusercontent.com/alibaba/druid/35ff7bafad6b5fdad6ed174e6bfbde8fa6396f46/src/main/resources/support/http/resources/js/common.js)
1. 把这个文件放在自己的项目里，路径：src/main/resource/support/http/resources/js/common.js

## 说明

**利益无关，纯属学习**

Mail: vincent7xin@gmail.com