+++
author = "Abser"
categories = ["acl","go"]
date = 2018-12-13T18:41:35+08:00
description = "go进行用户管理控制的源码阅读"
featured = ""
featuredalt = ""
featuredpath = ""
linktitle = ""
title = "Example ACL"
type = "post"

+++

# 项目简介

## TL;DR;

*acl* is a lightweight acl manager for *go*.

## Features

- Design simple & reusable roles to empower your application.
- Acquire the rights of other roles to build a powerful set of permissions.
- Resolve possible roles by examine them in an unified way.

## Example

```go
type User struct {
    isAdmin bool
}

func main() {
    // first of all: create a new manager instance to register all your roles in one place
    manager := acl.NewManager()

    // now you can use `Ensure` to guarantee that the role with the passed identifier is present
    user := manager.Ensure("user").Grant("profile.edit")
    // use `Grant`, `Revoke` and `AcquireFrom` to extend the right stack
    editor := manager.Ensure("editor").Grant("news.list", "news.create", "news.edit").AcquireFrom(user)

    // you can also use NewRole to create a Role manually
    admin := acl.NewRole("admin").Grant("news.delete").AcquireFrom(editor)
    // note, that you have to register the role by yourself
    manager.Register(admin)

    // to check if a right was granted to a role you can use:
    var hasAccess bool
    hasAccess = admin.Has("some.right")

    // to check if at least one of the expected rights is present:
    hasAccess = admin.HasOneOf("news.list", "news.create")

    // ... and finally, to check that all the expected rights are present, use:
    hasAccess = admin.HasAllOf("news.delete", "news.list")

    // a role can be extended with an examiner to determine whether a role can be added
    // to a `ResultSet`
    admin.SetExaminer(func (payload interface{}) bool {
        user := payload.(User)
        return user.isAdmin
    })

    // to get a result set you can use the managers `Examine` function
    rs := manager.Examine(User{isAdmin: true})

    // a result set contains "Has", "HasOneOf" and "HasAllOf" as described above and...
    // `GetRole` to grab specific roles from the result set
    expectedRole := rs.GetRole("admin")

    // you can also check if a role was added to a result set using:
    if rs.HasRole("admin") {
        // ...
    }
}
```



## Blueprint

![MrBoolean_acl](http://bmob-cdn-22876.b0.upaiyun.com/2018/12/13/ac72fe7e40e7a8658074a669bb56ab13.jpg)