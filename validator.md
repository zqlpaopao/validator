目前大都是使用[validator](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fgo-playground%2Fvalidator)
 下面有些例子可能还会用到以下的包，请自行安装：
 [https://github.com/go-playground/locales](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fgo-playground%2Flocales)
 [https://github.com/go-playground/universal-translator](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fgo-playground%2Funiversal-translator)

# 1.安装



```go
go get gopkg.in/go-playground/validator.v9
```

# 2. 原理

当然只能通过反射来实现了，之前写过一篇反射的文章[golang之反射和断言](https://www.jianshu.com/p/79257b59203d)，里面有写到怎么通过反射获取struct tag。
 读取struct tag之后就是对里面的标识符进行识别，然后进行验证了。具体可以去看源码。

# 3. demo

- 简单使用：



```go
package main

import (
    "fmt"
    "gopkg.in/go-playground/validator.v9"
)

// User contains user information
type UserInfo struct {
    FirstName      string     `validate:"required"`
    LastName       string     `validate:"required"`
    Age            uint8      `validate:"gte=0,lte=100"`
    Email          string     `validate:"required,email"`
}


func main() {
    validate := validator.New()
    user := &UserInfo{
        FirstName:      "Badger",
        LastName:       "Smith",
        Age:            105,
        Email:          "",
    }
    err := validate.Struct(user)
    if err != nil {
        for _, err := range err.(validator.ValidationErrors) {
            fmt.Println(err)
        }
        return
    }
    fmt.Println("success")
}
```

输出：



```csharp
Key: 'UserInfo.Age' Error:Field validation for 'Age' failed on the 'lte' tag
Key: 'UserInfo.Email' Error:Field validation for 'Email' failed on the 'required' tag
```

其它类型可以参照文档[https://godoc.org/gopkg.in/go-playground/validator.v9](https://links.jianshu.com/go?to=https%3A%2F%2Fgodoc.org%2Fgopkg.in%2Fgo-playground%2Fvalidator.v9)
 几个例子：



```rust
1.IP
type UserInfo struct {
    Ip             string     `validate:"ip"`
}
2.数字
type UserInfo struct {
    Number float32 `validate:"numeric"`
}
3.最大值
type UserInfo struct {
    Number float32 `validate:"max=10"`
}
4.最小值
type UserInfo struct {
    Number float32 `validate:"min=10"`
}
```

- 自定义验证函数



```go
package main

import (
    "fmt"
    "gopkg.in/go-playground/validator.v9"
    "unicode/utf8"
)

// User contains user information
type UserInfo struct {
    Name           string     `validate:"checkName"`
    Number float32 `validate:"numeric"`
}
// 自定义验证函数
func checkName(fl validator.FieldLevel) bool {
    count := utf8.RuneCountInString(fl.Field().String())
    fmt.Printf("length: %v \n", count)
    if  count > 5 {
        return false
    }
    return true
}

func main() {
    validate := validator.New()
        //注册自定义函数，与struct tag关联起来
    err := validate.RegisterValidation("checkName", checkName)
    user := &UserInfo{
        Name:            "我是中国人，我爱自己的祖国",
        Number:         23,
    }
    err = validate.Struct(user)
    if err != nil {
        for _, err := range err.(validator.ValidationErrors) {
            fmt.Println(err)
        }
        return
    }
    fmt.Println("success")
}
```

- 设置错误提示为中文



```go
package main

import (
    "fmt"
    "github.com/go-playground/locales/zh"
    "github.com/go-playground/universal-translator"
    "gopkg.in/go-playground/validator.v9"
    zh_translations "gopkg.in/go-playground/validator.v9/translations/zh"
)

type UserInfo struct {
    FirstName      string     `validate:"required"`
    LastName       string     `validate:"required"`
    Age            uint8      `validate:"gte=0,lte=100"`
    Email          string     `validate:"required,email"`
}


func main() {
    //中文翻译器
    zh_ch := zh.New()
    uni := ut.New(zh_ch)
    trans, _ := uni.GetTranslator("zh")
    //验证器
    validate := validator.New()
    //验证器注册翻译器
    zh_translations.RegisterDefaultTranslations(validate, trans)
    user := &UserInfo{
        FirstName:      "Badger",
        LastName:       "Smith",
        Age:            105,
        Email:          "",
    }
    err := validate.Struct(user)
    if err != nil {
        for _, err := range err.(validator.ValidationErrors) {
                        //翻译错误信息
            fmt.Println(err.Translate(trans))
        }
        return
    }
    fmt.Println("success")
}
```

输出：



```undefined
Age必须小于或等于100
Email为必填字段
```

