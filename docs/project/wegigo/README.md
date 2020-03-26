---
title: Wegigo开发笔记
date: 2018-01-12
draft: false
categories:
- 项目
---

#### 2018-01-12

- Issue1: sdk/server/pongo2_mem_loader.go:18: undefined: pongo2.TemplateLoader

    * Solution:

        > The dependency pongo need use the last version in master branch, change Gopkg.toml for pongo2 from `version = "3.0.0"` to `branch = "master"` and run `dep ensure`again.

- Issue2: vendor/github.com/coreos/etcd/client/keys.generated.go:71: r.EncodeArrayStart undefined

    * Symptom:

            vendor/github.com/coreos/etcd/client/keys.generated.go:71: r.EncodeArrayStart undefined (type codec.genHelperEncDriver has no field or method EncodeArrayStart)
            vendor/github.com/coreos/etcd/client/keys.generated.go:79: r.EncodeMapStart undefined (type codec.genHelperEncDriver has no field or method EncodeMapStart)
            vendor/github.com/coreos/etcd/client/keys.generated.go:83: z.EncSendContainerState undefined (type codec.genHelperEncoder has no field or method EncSendContainerState)
            vendor/github.com/coreos/etcd/client/keys.generated.go:91: z.EncSendContainerState undefined (type codec.genHelperEncoder has no field or method EncSendContainerState)
            vendor/github.com/coreos/etcd/client/keys.generated.go:93: z.EncSendContainerState undefined (type codec.genHelperEncoder has no field or method EncSendContainerState)
            vendor/github.com/coreos/etcd/client/keys.generated.go:102: z.EncSendContainerState undefined (type codec.genHelperEncoder has no field or method EncSendContainerState)
            vendor/github.com/coreos/etcd/client/keys.generated.go:109: z.EncSendContainerState undefined (type codec.genHelperEncoder has no field or method EncSendContainerState)
            vendor/github.com/coreos/etcd/client/keys.generated.go:111: z.EncSendContainerState undefined (type codec.genHelperEncoder has no field or method EncSendContainerState)
            vendor/github.com/coreos/etcd/client/keys.generated.go:119: z.EncSendContainerState undefined (type codec.genHelperEncoder has no field or method EncSendContainerState)
            vendor/github.com/coreos/etcd/client/keys.generated.go:126: z.EncSendContainerState undefined (type codec.genHelperEncoder has no field or method EncSendContainerState)
            vendor/github.com/coreos/etcd/client/keys.generated.go:126: too many errors
  
    * Solution:

          > The genreated keys is obselote, need delete it manually, then it will generate the latest one automatically while building
          > `rm vendor/github.com/coreos/etcd/client/keys.generated.go`

- Issue3: "plugin: not implemented" error repoted when open plugin by statically linked binary

    * Symptom:

            When using statically linked binary to try open plugin, it reports error: "plugin: not implemented"

    * Solution:

        > This is a known issue: 
        > - [plugin: requires CGO_ENABLED=1](https://github.com/golang/go/issues/19569)
        > - [go/build: fail to build "plugin" if CGO_ENABLED=0](https://go-review.googlesource.com/c/go/+/43158)
        > Basically, it need CGO_ENABLED=1 when compiling the binary which need invoke plugin


##### 2018-01-29
- Issue1: runtime: goroutine stack exceeds 1000000000-byte limit when do pluing.Open()

    * Symptom:

            runtime: goroutine stack exceeds 1000000000-byte limit
            fatal error: stack overflow

            runtime stack:
            runtime.throw(0xe8d6b2, 0xe)
                    /usr/local/go/src/runtime/panic.go:596 +0x95
            runtime.newstack(0x0)
                    /usr/local/go/src/runtime/stack.go:1089 +0x3f2
            runtime.morestack()
                    /usr/local/go/src/runtime/asm_amd64.s:398 +0x86

            goroutine 1 [running]:
            runtime.resolveNameOff(0x7f8e17ca1ce0, 0x79aa0, 0x0)
                    /usr/local/go/src/runtime/type.go:168 +0x326 fp=0xc440300360 sp=0xc440300358
            runtime.(*_type).nameOff(0x7f8e17ca1ce0, 0x79aa0, 0x0)
                    /usr/local/go/src/runtime/type.go:199 +0x33 fp=0xc440300388 sp=0xc440300360
            runtime.(*_type).string(0x7f8e17ca1ce0, 0x0, 0x0)
                    /usr/local/go/src/runtime/type.go:46 +0x36 fp=0xc4403003b0 sp=0xc440300388

    * Solution:

        > This seems to be a golang issue, when the second time to Open() plugin file, it will panic
        > The problem resolved in golang 1.9

##### 2018-02-03

- Issue1: Plugin build failed with "runtime.main_main·f: relocation target main.main not defined"

    * Symptom:

            runtime.main_main·f: relocation target main.main not defined
            runtime.main_main·f: undefined: "main.main"
            make: *** [plugin] Error 2


    * Solution:

        > The plugin which we try to build import a my own library which also refer to go's standard "plugin" library
        > It need avoid refer to standard "plugin" when building plugin file
