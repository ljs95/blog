---
title: SpringBoot上传文件到七牛云
date: 2019-12-16 20:09:22
tags:
 - Java
 - 工具类
categories:
 - Java
---

# 准备工作
## maven
pom.xml添加七牛云的sdk依赖
```xml
        <dependency>
            <groupId>com.qiniu</groupId>
            <artifactId>qiniu-java-sdk</artifactId>
            <version>7.2.27</version>
        </dependency>
```

## 配置项
七牛云上传必要的配置有：accessKey、secretKey、bucket
其中accessKey、secretKey在该网址可查看
> [https://portal.qiniu.com/user/key](https://portal.qiniu.com/user/key)

bucket为你的存储空间名，如下：
![image.png](http://qiniublog.colablog.cn/e8e42937-7465-42c7-bdd1-f465fc58a1e6.png)

---

# 实现
## application.yml配置
```yml
upload:
  qiniu:
    domain: 填你的域名
    access-key: 你的accesskey
    secret-key: 你的secretKey
    bucket: 你的存储空间名，我这里为colablog
```

可以看到我的七牛云上传配置中有`domain`这项配置，这个配置是七牛云buket的存储域名，在**内容管理**下，主要用于上传文件成功后把文件访问路径返还回去。
![image.png](http://qiniublog.colablog.cn/350b2429-074f-4487-993a-fb77b7b26c2c.png)
但是这个域名是限时30天使用的，所以你最好绑定一个新的域名。

## 上传配置类
使用SpringBoot的`@ConfigurationProperties`和`@Component`注解实现上传的配置类`UploadProperties`，
因为上传配置可能会有本地上传和云上传或者其他上传的，所以该配置类我留了扩展点。因为受到了rabbitmq的配置类启发，而且上传的配置不会很多，所以用内部类来分割这种配置类。上传配置类如下：
```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

/**
 * @author Johnson
 * @date 2019/12/16/ 09:35:36
 */
@Component
@ConfigurationProperties(prefix = "upload")
public class UploadProperties {

    private Local local = new Local();

    public Local getLocal() {
        return local;
    }

    /**
     * @author: Johnson
     * @Date: 2019/12/16
     * 本地上传配置
     */
    public class Local {
        ...
    }

    private Qiniu qiniu = new Qiniu();

    public Qiniu getQiniu() {
        return qiniu;
    }

    /**
     * @author: Johnson
     * @Date: 2019/12/16
     * 七牛云上传配置
     */
    public class Qiniu {
        /**
         * 域名
         */
        private String domain;

        /**
         * 从下面这个地址中获取accessKey和secretKey
         * https://portal.qiniu.com/user/key
         */
        private String accessKey;

        private String secretKey;

        /**
         * 存储空间名
         */
        private String bucket;

        public String getDomain() {
            return domain;
        }

        public void setDomain(String domain) {
            this.domain = domain;
        }

        public String getAccessKey() {
            return accessKey;
        }

        public void setAccessKey(String accessKey) {
            this.accessKey = accessKey;
        }

        public String getSecretKey() {
            return secretKey;
        }

        public void setSecretKey(String secretKey) {
            this.secretKey = secretKey;
        }

        public String getBucket() {
            return bucket;
        }

        public void setBucket(String bucket) {
            this.bucket = bucket;
        }
    }
}
```

## 七牛云上传接口和类
上传接口如下：
```java
public interface UploadFile {
    String uploadFile(MultipartFile file);
}

```
上传类
```java
import cn.colablog.blogserver.utils.properties.UploadProperties;
import com.qiniu.http.Response;
import com.qiniu.storage.Configuration;
import com.qiniu.storage.Region;
import com.qiniu.storage.UploadManager;
import com.qiniu.util.Auth;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.util.UUID;

/**
 * @author Johnson
 * @date 2019/12/14/ 17:20:16
 * 上传文件到七牛云
 */
public class UploadFileQiniu implements UploadFile {

    private UploadProperties.Qiniu properties;

    //构造一个带指定Region对象的配置类
    private Configuration cfg = new Configuration(Region.region2());

    private UploadManager uploadManager= new UploadManager(cfg);

    public UploadFileQiniu(UploadProperties.Qiniu properties) {
        this.properties = properties;
    }

    /**
    * @author: Johnson
    */
    @Override
    public String uploadFile(MultipartFile file) {
        Auth auth = Auth.create(properties.getAccessKey(), properties.getSecretKey());
        String token = auth.uploadToken(properties.getBucket());
        try {
            String originalFilename = file.getOriginalFilename();
            // 文件后缀
            String suffix = originalFilename.substring(originalFilename.lastIndexOf("."));
            String fileKey = UUID.randomUUID().toString() + suffix;
            Response response = uploadManager.put(file.getInputStream(), fileKey, token, null, null);
            return properties.getDomain() + fileKey;
        } catch (IOException e) {
            e.printStackTrace();
        }
        return "error";
    }
}
```
`Region`配置，这里代表空间的存取区域，因为我的存储空间的区域为华南。所以为`Region.region2()`,查看自己的存储区域可在**空间概览**的最下方查看到，这里就不截图了，图片占位太大。  
`Region`对应的设置：
![image.png](http://qiniublog.colablog.cn/f1717ab3-17de-4bf1-a237-8b841333563d.png)

好了，准备工作已完成，现在就到调用了，调用类如下：
```java
@RestController
@RequestMapping("/upload")
public class UploadFileController {
    @Autowired
    UploadProperties uploadProperties;

    @PostMapping("/img")
    public String uploadFileYun(MultipartFile file) {
        // 上传到七牛云
        UploadFile uploadFile = new UploadFileQiniu(uploadProperties.getQiniu());
        return uploadFile.uploadFile(file);
    }
}
```
是不是很简单呢？屁啊！简单个毛线，其实这个我是已经简化了，实际上在我的项目的结构是比这个复杂的。

---

# 总结
一：我的类名都是以`Upload`开头的，类名已经写死了只有上传功能，就限制了这个类的可扩展性了，假如添加文件删除功能，就不应该添加到这个类中。现在已经在重构**文件操作**（非文件上传了）的功能模块了。

二：一开始我觉得文件上传功能可能使用的比较少，所以使用到的时候才去实例化文件上传类，但是这需要根据开发场景来决定，因为我的项目是一个博客后台管理系统，会经常上传图片，所以上传文件类可以注入到Spring容器中，这样也可以减少实例化的开销（虽然很小）。注入的话就是用`@Component`类注解。

三：配置文件我为什么会想到使用内部类来分割配置项呢？其实大家在编写一些类似功能的时候，都可以去参照一下别人的源码，当然，这里指的是大神的源码。因为我在写配置项的时候就想看看大神的配置项是怎么写的，就点进了rabbitmq的配置项。所以啊，看到了大神的代码是真的有长进的。

如果你需要查看更详细的官方文档，请点击下方链接：
> [https://developer.qiniu.com/kodo/sdk/1239/java](https://developer.qiniu.com/kodo/sdk/1239/java)

---

最后：感谢大家的阅读，Thanks♪(･ω･)ﾉ