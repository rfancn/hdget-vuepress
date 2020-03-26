---
title: Pass additional argument to httprouter handle func
draft: false
date: 2018-01-12
categories:
- 博客
tags:
- program
- golang
---

As we known, standard httprouter func has 3 parameters like below:

	func ViewIndex(w http.ResponseWriter, r *http.Request, ps httprouter.Params){}

Sometimes, we need pass additional arguments to httprouter handle func, like:

	func ViewIndex(w http.ResponseWriter, r *http.Request, ps httprouter.Params, settings map[string]string){}
	
There are two ways to achive this target:

1. Use a wrapper

	func getIndexWithSettings2(s Settings) httprouter.Handle {
		return func(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
			//
	
			// Settings is in the scope and ps httprouter.params
			fmt.Println(s.Path)
	
			//
		}
	}
	
2. Use a middleware

	func settingsMiddleware(h http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			// Take the context out from the request
			ctx := r.Context()
	
			// Get the settings
			s := somewhere()
	
			// Get new context with key-value "settings"
			ctx = context.WithValue(ctx, "params", s)
	
			// Get new http.Request with the new context
			r = r.WithContext(ctx)
	
			// Call your original http.Handler
			h.ServeHTTP(w, r)
		})
	}
	
#### Full example

	package main
	
	import (
		"context"
		"fmt"
		"net/http"
	
		"github.com/julienschmidt/httprouter"
	)
	
	func main() {
		r := httprouter.New()
	
		index := settingsMiddleware(http.HandlerFunc(getIndex))
	
		r.GET("/index", wrapHandler(index))
	
		http.ListenAndServe(":8080", r)
	}
	
	func wrapHandler(h http.Handler) httprouter.Handle {
		return func(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
			// Take the context out from the request
			ctx := r.Context()
			ctx = context.WithValue(ctx, "params", ps)
			r = r.WithContext(ctx)
			h.ServeHTTP(w, r)
		}
	}
	
	func settingsMiddleware(h http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			var s = "/var/bin"
	
			ctx := r.Context()
			ctx = context.WithValue(ctx, "settings", s)
			r = r.WithContext(ctx)
			h.ServeHTTP(w, r)
		})
	}
	
	func getIndex(w http.ResponseWriter, r *http.Request) {
		s, ok := r.Context().Value("settings").(string)
	
		if !ok {
			fmt.Println("s is not type string")
		}
	
		fmt.Println(s) // "/var/bin"
	}