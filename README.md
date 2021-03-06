<h1 align="center">spring-file-storage</h1>

<p align="center">
	<a target="_blank" href="https://search.maven.org/artifact/cn.xuyanwu/spring-file-storage">
		<img src="https://img.shields.io/maven-central/v/cn.xuyanwu/spring-file-storage.svg?label=Maven%20Central" />
	</a>
	<a target="_blank" href="https://www.apache.org/licenses/LICENSE-2.0">
		<img src="https://img.shields.io/badge/license-Apache%202-green.svg" />
	</a>
	<a target="_blank" href="https://www.oracle.com/technetwork/java/javase/downloads/index.html">
		<img src="https://img.shields.io/badge/JDK-8+-blue.svg" />
	</a>
	<a target="_blank" href='https://github.com/1171736840/spring-file-storage'>
		<img src="https://img.shields.io/github/stars/1171736840/spring-file-storage.svg?style=social" alt="github star"/>
	</a>

</p>

### 简介
在SpringBoot中通过简单的方式将文件存储到本地、阿里云OSS、华为云OBS、七牛云Kodo

`spring-file-storage` 模块是本体。

`spring-file-storage-test` 模块是测试+使用演示，不需要的情况下可以直接删除。

### 使用说明

#### 配置

`pom.xml`引入依赖

```xml

<dependencies>
    <!-- spring-file-storage 必须要引入 -->
    <dependency>
        <groupId>cn.xuyanwu</groupId>
        <artifactId>spring-file-storage</artifactId>
        <version>0.1.3</version>
    </dependency>

    <!-- 华为云 OBS 不使用的情况下可以不引入-->
    <dependency>
        <groupId>com.huaweicloud</groupId>
        <artifactId>esdk-obs-java</artifactId>
        <version>3.20.6.1</version>
    </dependency>

    <!-- 阿里云 OSS 不使用的情况下可以不引入-->
    <dependency>
        <groupId>com.aliyun.oss</groupId>
        <artifactId>aliyun-sdk-oss</artifactId>
        <version>3.6.0</version>
    </dependency>

    <!-- 七牛云 Kodo 不使用的情况下可以不引入-->
    <dependency>
        <groupId>com.qiniu</groupId>
        <artifactId>qiniu-java-sdk</artifactId>
        <version>7.4.0</version>
    </dependency>
</dependencies>
```

`application.yml`配置文件中添加以下相关配置（不使用的平台可以不配置）

```yaml
spring:
  file-storage: #文件存储配置
    default-platform: local-1 #默认使用的存储平台
    thumbnail-suffix: ".min.jpg" #缩略图后缀，例如【.min.jpg】【.png】
    local: # 本地存储，不使用的情况下可以不写
      - platform: local-1 # 存储平台标识
        enable-storage: true  #启用存储
        enable-access: true #启用访问（线上请使用 Nginx 配置，效率更高）
        domain: "" # 访问域名，例如：“http://127.0.0.1:8030/test/file/”，注意后面要和 path-patterns 保持一致，“/”结尾，本地存储建议使用相对路径，方便后期更换域名
        base-path: D:/Temp/test/ # 存储地址
        path-patterns: /test/file/** # 访问路径，开启 enable-access 后，通过此路径可以访问到上传的文件
    huawei-obs: # 华为云 OBS ，不使用的情况下可以不写
      - platform: huawei-obs-1 # 存储平台标识
        enable-storage: false  # 启用存储
        access-key: ??
        secret-key: ??
        end-point: ??
        bucket-name: ??
        domain: ?? # 访问域名，注意“/”结尾，例如：http://abc.obs.com/
        base-path: hy/ # 基础路径
    aliyun-oss: # 阿里云 OSS ，不使用的情况下可以不写
      - platform: aliyun-oss-1 # 存储平台标识
        enable-storage: false  # 启用存储
        access-key: ??
        secret-key: ??
        end-point: ??
        bucket-name: ??
        domain: ?? # 访问域名，注意“/”结尾，例如：https://abc.oss-cn-shanghai.aliyuncs.com/
        base-path: hy/ # 基础路径
    qiniu-kodo: # 七牛云 kodo ，不使用的情况下可以不写
      - platform: qiniu-kodo-1 # 存储平台标识
        enable-storage: false  # 启用存储
        access-key: ??
        secret-key: ??
        bucket-name: ??
        domain: ?? # 访问域名，注意“/”结尾，例如：http://abc.hn-bkt.clouddn.com/
        base-path: base/ # 基础路径
```

注意配置每个平台前面都有个`-`号，通过以下方式可以配置多个

```yaml
local:
  - platform: local-1 # 存储平台标识
    enable-storage: true
    enable-access: true
    domain: ""
    base-path: D:/Temp/test/
    path-patterns: /test/file/**
  - platform: local-2 # 存储平台标识，注意这里不能重复
    enable-storage: true
    enable-access: true
    domain: ""
    base-path: D:/Temp/test2/
    path-patterns: /test2/file/**
```

#### 编码

在启动类上加上`@EnableFileStorage`注解

```java
@EnableFileStorage
@SpringBootApplication
public class SpringFileStorageTestApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringFileStorageTestApplication.class, args);
	}
	
}
```

#### 开始使用

```java

@RestController
public class FileDetailController {

    @Autowired
    private FileStorageService fileStorageService;//注入实列

    /**
     * 上传文件，成功返回文件 url
     */
    @PostMapping("/upload")
    public String upload(MultipartFile file) {
        FileInfo fileInfo = fileStorageService.of(file)
                .setPath("upload/") //保存到相对路径下，为了方便管理，不需要可以不写
                .setObjectId("0")   //关联对象id，为了方便管理，不需要可以不写
                .setObjectType("0") //关联对象类型，为了方便管理，不需要可以不写
                .upload();  //将文件上传到对应地方
        return fileInfo == null ? "上传失败！" : fileInfo.getUrl();
    }

    /**
     * 上传图片，成功返回文件信息
     * 图片处理使用的是 https://github.com/coobird/thumbnailator
     */
    @PostMapping("/upload-image")
    public FileInfo uploadImage(MultipartFile file) {
        return fileStorageService.of(file)
                .image(img -> img.size(1000,1000))  //将图片大小调整到 1000*1000
                .thumbnail(th -> th.size(200,200))  //再生成一张 200*200 的缩略图
                .upload();
    }

    /**
     * 上传文件到指定存储平台，成功返回文件信息
     */
    @PostMapping("/upload-platform")
    public FileInfo uploadPlatform(MultipartFile file) {
        return fileStorageService.of(file)
                .setPlatform("aliyun-oss-1")    //使用指定的存储平台
                .upload();
    }
}
```

如果还想使用除了保存文件之前的其它功能，例如删除文件，还需要实现 `FileRecorder` 这个接口，把文件信息保存到数据库中

<details>
<summary>点击查看详情</summary>

```java
/**
 * 用来将文件上传记录保存到数据库，这里使用了 MyBatis-Plus 和 Hutool 工具类
 */
@Service
public class FileDetailService extends ServiceImpl<FileDetailMapper, FileDetail> implements FileRecorder {

    /**
     * 保存文件信息到数据库
     */
    @Override
    public boolean record(FileInfo info) {
        FileDetail detail = BeanUtil.copyProperties(info,FileDetail.class);
        boolean b = save(detail);
        if (b) {
            info.setId(detail.getId());
        }
        return b;
    }

    /**
     * 根据 url 查询文件信息
     */
    @Override
    public FileInfo getByUrl(String url) {
        return BeanUtil.copyProperties(getOne(new QueryWrapper<FileDetail>().eq(FileDetail.COL_URL,url)),FileInfo.class);
    }

    /**
     * 根据 url 删除文件信息
     */
    @Override
    public boolean delete(String url) {
        return remove(new QueryWrapper<FileDetail>().eq(FileDetail.COL_URL,url));
    }
}
```

数据库表结构推荐如下，你也可以根据自己喜好在这里自己扩展

```sql
-- 这里使用的是 mysql
CREATE TABLE `file_detail`
(
    `id`                int(10) UNSIGNED                                        NOT NULL AUTO_INCREMENT COMMENT '文件id',
    `url`               varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '文件访问地址',
    `size`              bigint(20)                                              NULL DEFAULT NULL COMMENT '文件大小，单位字节',
    `filename`          varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '文件名称',
    `original_filename` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '原始文件名',
    `base_path`         varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '基础存储路径',
    `path`              varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '存储路径',
    `ext`               varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci  NULL DEFAULT NULL COMMENT '文件扩展名',
    `platform`          varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci  NULL DEFAULT NULL COMMENT '存储平台',
    `th_url`            varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '缩略图访问路径',
    `th_filename`       varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '缩略图名称',
    `th_size`           bigint(20)                                              NULL DEFAULT NULL COMMENT '缩略图大小，单位字节',
    `object_id`         varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci  NULL DEFAULT NULL COMMENT '文件所属对象id',
    `object_type`       varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci  NULL DEFAULT NULL COMMENT '文件所属对象类型，例如用户头像，评价图片',
    `create_time`       datetime(0)                                             NULL DEFAULT NULL COMMENT '创建时间',
    PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB
  AUTO_INCREMENT = 1
  CHARACTER SET = utf8
  COLLATE = utf8_general_ci COMMENT = '文件记录表'
  ROW_FORMAT = Dynamic;
```
</details>

#### 自定义存储平台

<details>
<summary>点击查看详情</summary>

想要自定义存储平台就要实现 `FileStorage` 这个接口，并进行实例化，注意返回的 bean 是个 list

这里拿 LocalFileStorage 举例
```java

/**
 * 实现 FileStorage 接口，这里使用了 Lombok 和 Hutool 工具类
 */
@Getter
@Setter
public class LocalFileStorage implements FileStorage {

    /* 本地存储路径*/
    private String basePath;
    /* 存储平台 */
    private String platform;
    /* 访问域名 */
    private String domain;

    /**
     * 保存文件
     */
    @Override
    public boolean save(FileInfo fileInfo,UploadPretreatment pre) {
        String path = fileInfo.getPath();

        File newFile = FileUtil.touch(basePath + path,fileInfo.getFilename());
        fileInfo.setBasePath(basePath);
        fileInfo.setUrl(domain + path + fileInfo.getFilename());

        try {
            pre.getFileWrapper().transferTo(newFile);

            byte[] thumbnailBytes = pre.getThumbnailBytes();
            if (thumbnailBytes != null) { //上传缩略图
                fileInfo.setThUrl(fileInfo.getUrl() + pre.getThumbnailSuffix());
                FileUtil.writeBytes(thumbnailBytes,newFile.getPath() + pre.getThumbnailSuffix());
            }
            return true;
        } catch (IOException e) {
            FileUtil.del(newFile);
            throw new FileStorageRuntimeException("文件上传失败！platform：" + platform + "，filename：" + fileInfo.getOriginalFilename(),e);
        }
    }

    /**
     * 删除文件
     */
    @Override
    public boolean delete(FileInfo fileInfo) {
        if (fileInfo.getThFilename() != null) {   //删除缩略图
            FileUtil.del(new File(fileInfo.getBasePath() + fileInfo.getPath(),fileInfo.getThFilename()));
        }
        return FileUtil.del(new File(fileInfo.getBasePath() + fileInfo.getPath(),fileInfo.getFilename()));
    }

    /**
     * 文件是否存在
     */
    @Override
    public boolean exists(FileInfo fileInfo) {
        return new File(fileInfo.getBasePath() + fileInfo.getPath(),fileInfo.getFilename()).exists();
    }
}

/**
 * 初始化
 */
@Configuration
public class LocalFileStorageAutoConfiguration {

    /**
     * 这里拿本地存储做个演示，注意返回的是个List
     */
    @Bean
    public List<LocalFileStorage> localFileStorageList() {
        ArrayList<LocalFileStorage> list = new ArrayList<>();
        LocalFileStorage localFileStorage = new LocalFileStorage();
        localFileStorage.setPlatform("my-local-1");//平台名称
        localFileStorage.setBasePath("");
        localFileStorage.setDomain("");
        list.add(localFileStorage);
        return list;
    }
}
```
</details>

#### 自定义上传和删除等切面

<details>
<summary>点击查看详情</summary>

只需要实现`FileStorageAspect`接口即可对文件上传和删除等进行干预。

不需要的方法可以不用实现，此接口里的方法全部都有默认实现

```java
/**
 * 使用切面打印文件上传和删除的日志
 */
@Slf4j
@Component
public class LogFileStorageAspect implements FileStorageAspect {

    /**
     * 上传，成功返回文件信息，失败返回 null
     */
    @Override
    public FileInfo uploadAround(UploadAspectChain chain,FileInfo fileInfo,UploadPretreatment pre,FileStorage fileStorage,FileRecorder fileRecorder) {
        log.info("上传文件 before -> {}",fileInfo);
        fileInfo = chain.next(fileInfo,pre,fileStorage,fileRecorder);
        log.info("上传文件 after -> {}",fileInfo);
        return fileInfo;
    }

    /**
     * 删除文件，成功返回 true
     */
    @Override
    public boolean deleteAround(DeleteAspectChain chain,FileInfo fileInfo,FileStorage fileStorage,FileRecorder fileRecorder) {
        log.info("删除文件 before -> {}",fileInfo);
        boolean res = chain.next(fileInfo,fileStorage,fileRecorder);
        log.info("删除文件 after -> {}",res);
        return res;
    }
}
```
</details>
