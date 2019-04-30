　　在开发阶段，常常会使用本地参数覆盖掉远程参数，就需要进行一下配置：

```
spring:
  cloud:
    config:
      # 当 allow-override 为 true 时，
      # override-none 设置为 true，外部的配置优先级更低，
      # 而且不能覆盖任何存在的属性源，默认：false
      override-none: true
      # 标识 override-system-properties 属性是否启用。默认：true
      # 设置为 false 意为禁止用户的设置
      allow-override: true
      # 用来标识外部配置是否能够覆盖系统属性，默认：true
      override-system-properties: false
```

* **注意：上面的配置项要写在 远程配置文件中**
