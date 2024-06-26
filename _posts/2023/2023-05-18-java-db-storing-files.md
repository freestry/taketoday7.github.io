---
layout: post
title:  存储由数据库索引的文件
category: springdata
copyright: springdata
excerpt: Spring Data
---

## 1. 概述

当我们构建某种内容管理解决方案时，我们需要解决两个问题。我们需要一个地方来存储文件本身，我们需要某种数据库来索引它们。

可以将文件的内容存储在数据库本身中，或者我们可以将内容存储在其他地方并使用数据库对其进行索引。

在本文中，我们将通过一个基本的图像存档应用程序来说明这两种方法。我们还将实现用于上传和下载的REST API。

## 2. 用例

我们的图像存档应用程序将允许我们**上传和下载JPEG图像**。

当我们上传一张图片时，应用程序会为它创建一个唯一的标识符。然后我们可以使用这个标识符来下载它。

我们将使用带有[Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)和[Hibernate](https://www.baeldung.com/tag/hibernate/)的关系数据库。

## 3. 数据库存储

让我们从我们的数据库开始。

### 3.1 图像实体

首先，让我们创建Image实体：

```java
@Entity
class Image {

    @Id
    @GeneratedValue
    Long id;

    @Lob
    byte[] content;

    String name;
    // Getters and Setters
}
```

id字段用@GeneratedValue标注，这意味着数据库将为我们添加的每条记录创建一个唯一标识符(id)。通过使用这些值索引图像，我们无需担心同一图像的多次上传相互冲突。

其次，我们使用了[Hibernate @Lob注解](https://www.baeldung.com/hibernate-lob)，这就是我们告诉JPA我们打算**存储一个可能很大的二进制文件**的方式。

### 3.2 Repository

接下来，我们**需要一个Repository来连接数据库**。

我们将使用Spring [JpaRepository](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)：

```java
@Repository
interface ImageDbRepository extends JpaRepository<Image, Long> {}
```

现在我们准备保存我们的图像。我们只需要一种方法将它们上传到我们的应用程序。

## 4. RestController

我们使用[MultipartFile](https://www.baeldung.com/spring-file-upload)上传我们的图像。上传将返回imageId，我们稍后可以使用它来下载图像。

### 4.1 图片上传

下面创建我们的ImageController来支持上传：

```java
@RestController
class ImageController {

    @Autowired
    ImageDbRepository imageDbRepository;

    @PostMapping
    Long uploadImage(@RequestParam MultipartFile multipartImage) throws Exception {
        Image dbImage = new Image();
        dbImage.setName(multipartImage.getName());
        dbImage.setContent(multipartImage.getBytes());

        return imageDbRepository.save(dbImage).getId();
    }
}
```

MultipartFile对象包含文件的内容和原始名称。我们使用它来构造我们的Image对象以将其存储在数据库中。

该控制器返回生成的id作为其响应的主体。

### 4.2 图片下载

现在，让我们添加一个下载路径：

```java
@GetMapping(value = "/image/{imageId}", produces = MediaType.IMAGE_JPEG_VALUE)
Resource downloadImage(@PathVariable Long imageId) {
    byte[] image = imageRepository.findById(imageId)
        .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND))
        .getContent();

    return new ByteArrayResource(image);
}
```

imageId路径变量包含上传时生成的ID。如果提供的ID无效，那么我们将使用ResponseStatusException返回HTTP响应代码404(Not Found)。否则，我们将存储的文件字节包装在ByteArrayResource中，这样就可以下载它们。

## 5. 数据库图像存档测试

现在我们准备测试我们的图像存档。

首先，让我们构建我们的应用程序：

```shell
mvn package
```

其次，让我们启动它：

```shell
java -jar target/image-archive-1.0.0.jar
```

### 5.1 图片上传测试

应用程序运行后，我们**使用[curl命令行工具](https://www.baeldung.com/curl-rest)上传我们的图像**：

```shell
curl -H "Content-Type: multipart/form-data" \
  -F "image=@tuyucheng.jpeg" http://localhost:8080/image
```

由于上传服务的**响应是imageId**，这是我们的第一个请求，输出将是：

```shell
1
```

### 5.2 图片下载测试

然后我们可以下载我们的图像：

```shell
curl -v http://localhost:8080/image/1 -o image.jpeg
```

-o image.jpeg选项将创建一个名为image.jpeg的文件并将响应内容存储在其中：

```shell
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> GET /image/1 HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
> 
< HTTP/1.1 200 
< Accept-Ranges: bytes
< Content-Type: image/jpeg
< Content-Length: 9291
```

我们得到了一个HTTP/1.1 200，这意味着我们的下载成功了。

我们也可以尝试通过访问[http://localhost:8080/image/1](http://localhost:8080/image/1)在浏览器中下载图像。

## 6. 分离内容和位置

到目前为止，我们能够在数据库中上传和下载图像。

另一个不错的选择是将文件内容上传到不同的位置。然后我们**只在数据库中保存它的文件系统位置**。

为此，我们需要向Image实体添加一个新字段：

```java
String location;
```

这将包含某些外部存储中文件的逻辑路径。在我们的例子中，**它将是我们服务器文件系统上的路径**。 

但是，我们同样可以将这个想法应用到不同的存储中。例如，我们可以使用云存储[Google Cloud Storage](https://www.baeldung.com/java-google-cloud-storage)或[Amazon S3](https://www.baeldung.com/aws-s3-java)。该位置还可以使用URI格式，例如s3://somebucket/path/to/file。

我们的上传服务不是将文件的字节写入数据库，而是将文件存储在适当的服务(在本例中为文件系统)中，然后将文件的位置放入数据库中。

## 7. 文件系统存储

让我们将**在文件系统中存储图像**的功能添加到我们的解决方案中。

### 7.1 保存在文件系统中

首先，我们需要将图像保存到文件系统：

```java
@Repository
class FileSystemRepository {
    String RESOURCES_DIR = FileSystemRepository.class.getResource("/").getPath();

    String save(byte[] content, String imageName) throws Exception {
        Path newFile = Paths.get(RESOURCES_DIR + new Date().getTime() + "-" + imageName);
        Files.createDirectories(newFile.getParent());

        Files.write(newFile, content);

        return newFile.toAbsolutePath().toString();
    }
}
```

一个重要的注意事项-我们需要**确保我们的每张图片在上传时都有一个在服务器端定义的唯一位置**。否则，我们的上传可能会相互覆盖。

同样的规则适用于任何云存储，我们应该在其中创建唯一的标识。在此示例中，我们将以毫秒格式将当前日期添加到图像名称中：

```shell
/workspace/archive-achive/target/classes/1602949218879-tuyucheng.jpeg
```

### 7.2 从文件系统中检索

现在让我们实现从文件系统中获取图像的代码：

```java
FileSystemResource findInFileSystem(String location) {
    try {
        return new FileSystemResource(Paths.get(location));
    } catch (Exception e) {
        // Handle access or file not found problems.
        throw new RuntimeException();
    }
}
```

在这里，我们**通过location属性查找图像**。然后我们返回一个FileSystemResource。

此外，我们正在捕获读取文件时可能发生的任何异常。我们可能还希望抛出具有[特定HTTP状态](https://www.baeldung.com/spring-response-status-exception)的异常。

### 7.3 数据流和Spring的资源

我们的findInFileSystem方法返回一个[FileSystemResource](https://docs.spring.io/spring-framework/docs/3.0.x/javadoc-api/org/springframework/core/io/FileSystemResource.html)，它是Spring的[Resource](https://www.baeldung.com/spring-classpath-file-access)接口的一个实现。

**只有当我们使用它时，它才会开始读取我们的文件**。在我们的例子中，它将通过RestController将其发送到客户端时。此外，**它会将文件内容从文件系统流式传输给用户，从而避免我们将所有字节加载到内存中**。

这种方法是将文件流式传输到客户端的一个很好的通用解决方案。如果我们使用云存储而不是文件系统，我们可以将FileSystemResource替换为另一个资源的实现，如[InputStreamResource](https://docs.spring.io/spring-framework/docs/3.2.4.RELEASE_to_4.0.0.M3/Spring%20Framework%203.2.4.RELEASE/org/springframework/core/io/InputStreamResource.html)或[ByteArrayResource](https://docs.spring.io/spring-framework/docs/3.0.6.RELEASE_to_3.1.0.BUILD-SNAPSHOT/3.0.6.RELEASE/org/springframework/core/io/ByteArrayResource.html)。

## 8. 连接文件内容和位置

现在我们有了FileSystemRepository，我们需要将它与我们的ImageDbRepository链接起来。

### 8.1 保存在数据库和文件系统中

让我们创建一个FileLocationService，从我们的保存流程开始：

```java
@Service
class FileLocationService {

    @Autowired
    FileSystemRepository fileSystemRepository;
    @Autowired
    ImageDbRepository imageDbRepository;

    Long save(byte[] bytes, String imageName) throws Exception {
        String location = fileSystemRepository.save(bytes, imageName);

        return imageDbRepository.save(new Image(imageName, location)).getId();
    }
}
```

首先，我们**将图像保存在文件系统中**。然后我们**将包含其位置location的记录保存在数据库中**。

### 8.2 从数据库和文件系统中检索

现在，让我们创建一个方法来使用其id查找我们的图像：

```java
FileSystemResource find(Long imageId) {
    Image image = imageDbRepository.findById(imageId)
        .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND));

    return fileSystemRepository.findInFileSystem(image.getLocation());
}
```

首先，我们**在数据库中查找我们的图像**。然后我们获取它的位置并**从文件系统中获取它**。

如果我们在数据库中没有找到imageId，我们将使用ResponseStatusException返回一个HTTP Not Found响应。

## 9. 文件系统上传和下载

最后，让我们创建FileSystemImageController：

```java
@RestController
@RequestMapping("file-system")
class FileSystemImageController {

    @Autowired
    FileLocationService fileLocationService;

    @PostMapping("/image")
    Long uploadImage(@RequestParam MultipartFile image) throws Exception {
        return fileLocationService.save(image.getBytes(), image.getOriginalFilename());
    }

    @GetMapping(value = "/image/{imageId}", produces = MediaType.IMAGE_JPEG_VALUE)
    FileSystemResource downloadImage(@PathVariable Long imageId) throws Exception {
        return fileLocationService.find(imageId);
    }
}
```

首先，我们让新路径以“/file-system”开头。

然后我们创建了类似于ImageController中的上传路由，但没有dbImage对象。

最后，我们有我们的下载路由，它使用FileLocationService查找图像并返回FileSystemResource作为HTTP响应。

## 10. 文件系统镜像存档测试

现在，我们可以像测试数据库版本一样测试文件系统版本，路径现在以“file-system”开头：

```bash
curl -H "Content-Type: multipart/form-data" \
  -F "image=@tuyucheng.jpeg" http://localhost:8080/file-system/image

1
```

然后我们下载：

```shell
curl -v http://localhost:8080/file-system/image/1 -o image.jpeg
```

## 11. 总结

在本文中，我们学习了如何将文件信息保存在数据库中，文件内容可以保存在同一行中，也可以在外部位置。

我们还构建并测试了使用分段上传的REST API，并使用Resource提供了下载功能，以允许将文件流式传输给调用者。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-data-modules)上获得。