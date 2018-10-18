title: spring 流服务
date: 2018-07-03 19:44:58
tags: [spring, file, image, video]
categories: spring
---

## 主要内容
spring boot (Spring Mvc) 提供文件流服务，最简单方式。

*更新历史*
无

环境：spring 4.2 以上


```java
    @ResponseBody
    @GetMapping(value = "file/{id}")
    public ResponseEntity getFile(@PathVariable Long id) {

        UrlResource resource = null;
        try {
            resource = new UrlResource(Paths.get(${Path of file id}).toUri());
        } catch (Exception e) {

        }

        if(resource == null) {
            return ResponseEntity.badRequest().body("无对应资源");
        }

        return ResponseEntity.status(HttpStatus.PARTIAL_CONTENT)  //断点续传
                .contentType(MediaTypeFactory
                        .getMediaType(resource)
                        .orElse(MediaType.APPLICATION_OCTET_STREAM)) // MediaType
                .body(resource);
    }

```

