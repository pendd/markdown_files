## 1. RESTful

> RESTful 是面向资源的  URI中都是名词

![avatar](https://raw.githubusercontent.com/pendd/picture/master/jpa/restful_uri.png)

> http 对应请求

![avatar](https://raw.githubusercontent.com/pendd/picture/master/jpa/restful_http.png)

前端请求过滤、排序、选择、分页
![avatar](https://raw.githubusercontent.com/pendd/picture/master/jpa/restful_request_method.png)

> 复杂资源关系的表达

**GET /cars/711/drivers/ 返回 使用过编号711汽车的所有司机**

**GET /cars/711/drivers/4 返回 使用过编号711汽车的4号司机**

> 版本化你的API

强制性增加API版本声明，不要发布无版本的API。如：/api/v1/blog