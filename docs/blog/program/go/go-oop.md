---
date: '2017-11-17 13:06:17'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- golang
title: Golang OOP
---

##### How to simiuate super.__init__() in Python

        type Baser interface {
            Init(name string)
            Do()
        }

        type BaseClass struct {
            name string
        }

        func (cls *BaseClass) Init(name string) {
            cls.name = name
        }


        type SepcificClass struct {
            BaseClass
        }

        func (cls *SpecificClass) Init(name string) {
            cls.BaseClass.Init(name)
        }

        func main() {
            var cls *specificClass
            cls = &SpecificClass{}
            cls.Init("ryan")
        }

##### How to automacially init base class's instance value
        type Baser interface {
            Do()
        }

        type BaseClass struct {
            name string
        }
        func (cls *BaseClass) Do() {
        }


        type SepcificClass struct {
            BaseClass
        }
        func (cls *SepcificClass) Do() {
        }

        // the explict way to initialize base class's value
        func (cls *SpecificClass) New1() Baser {
            return &SepcificClass{
                        BaseClass{"ryan"}
                    }
        }

        func (cls *SpecificClass) New2() Baser {
            instance := &SepcificClass{}
            instance.Baseclass.name = "ryan"
        }

        // the implicit way by using reflect
        func (cls *SpecificClass) New2() Baser {
            t := reflect.TypeOf(&SpecificClass{}).Elem()
            v := t.New().Elem()

            v.FieldByName("name").setString("ryan")
        }