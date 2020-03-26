---
title: Golang Tips
date: '2017-11-17 13:06:17'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- golang
---

#### How to write customized error with optional error struct passed

        
        type Y2HError struct {
            errMsg string
            errDetail error
        }

        func (e *Y2HError) Error() string {
            if e.errDetail != nil {
                return e.errMsg + "\n-------------------\n" + e.errDetail.Error()
            }
            return e.errMsg
        }

        func NewY2HError(errMsg string, errSlice ...error) *Y2HError {
            var errDetail error = nil
            if len(errSlice) == 1 {
                errDetail = errSlice[0]
            }
            return  &Y2HError{errMsg, errDetail}
        }

#### How to do generic type checking

Use reflect package to get TypeOf(object)

        func parseHtml(htmlElements []interface{}) []byte {
            for _, elem := range(htmlElements) {
                switch reflect.TypeOf(elem).Kind(){
                    case reflect.String:
                        fmt.Println("it is string")
                    case reflect.Map:
                        fmt.Println("it is map")
                }
            }
            ...
        }

#### How to append []byte to another []byte

Please follow: http://golang.org/doc/go_spec.html#Appending_and_copying_slices, it need pay attention that you need to use "[]T..." for the final argument. append() takes a slice of type []T, and then a variable number of values of the type of the slice member T. In other words, if you pass a []uint8 as the slice to append() then it wants every subsequent argument to be a uint8. The solution to this is to use the slice... syntax for passing a slice in place of a varargs argument. Your code should look like:

        content := make([]byte, 0)
        for i :=0; i<10; i++ {
            fileContent := ioutil.ReadFile("test.file")
            content = append(content, fileContent...)
        }


#### How to convert interface{} to string

Use fmt.Sprintf("%v", variable) to cast interface{} to string if you know the object is a string type