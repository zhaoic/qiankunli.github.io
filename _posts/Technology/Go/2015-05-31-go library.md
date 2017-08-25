---

layout: post
title: Go 常用的一些库
category: 技术
tags: Go
keywords: Go

---

## 一 前言

本文主要阐述一下golang中常用的库。

## 二 日志
golang中涉及到日志的库有很多，除了golang自带的log外，还有glog和log4go等，不过由于bug、更新缓慢和功能不强等原因，笔者推荐使用seelog。

### 2.1 安装

    $ go get github.com/cihub/seelog
    
### 2.1 使用

    package main
    import (
    	"fmt"
    	log "github.com/cihub/seelog"
    )
    func main() {
        // xml格式的字符串，xml配置了如何输出日志
    	testConfig := `
        <seelog type="sync">
            // 配置输出项，本例表示同时输出到控制台和日志文件中
        	<outputs formatid="main">
        		<console/>
        		<file path="log.log"/>
        	</outputs>
        	<formats>
        	    // 日志格式，outputs中的输出项可以通过id引用该格式
        		<format id="main" format="[%LEVEL] [%Time] [%FuncShort @ %File.%Line] %Msg%n"/>
        	</formats>
        </seelog>`   
    
        // 根据配置信息生成logger（应该是配置信息的对象表示）
    	logger, err := log.LoggerFromConfigAsBytes([]byte(testConfig))
    	if err != nil {
    		fmt.Println(err)
    	}
    	// 应该是配置日志组件加载配置信息，然后输出函数比如Info在输出时，会加载该配置
    	log.ReplaceLogger(logger)
    	log.Info("Hello from Seelog!")
    }
    
当然，也可以将配置项专门写入到一个文件中，使用`log.LoggerFromConfigAsFile("xx.xml")`加载。

## 三 json字符串与结构体的转换

Go中的json处理，跟结构体是密切相关的，一般要为json字符串建好相对应的struct。

如果只是想获取json串中某个key的值，可以使用`github.com/bitly/go-simplejson`

    js, err := simplejson.NewJson([]byte(inputStr))
	path := js.Get("path").MustString()
	
如果知道json字符串对应结构体（该结构体可能会嵌套），则可以使用golang自带的encoding/json包。

    type name struct {
    	FN string `json:"fn"`
    	LN string `json:"ln"`
    }
    type user struct {
    	Id       int64  `json:"id"`            // 指定转换为字符串后，该字段显示的key值
    	Username string `json:"username"`
    	Password string `json:"-"`            
    	N        name   `json:"name"`
    }
    func main() {
    	n := name{
    		FN: "li",
    		LN: "qk",
    	}
    	u := user{
    		Id:       1,
    		Username: "lqk",
    		Password: "123456",
    		N:        n,
    	}
    	// 根据结构体生成字符串
    	b, _ := json.Marshal(u)
    	
        jsonStr := string(b)
    	fmt.Println(jsonStr)
    	
        var u2 user
        // 根据字符串生成结构体
        err := json.Unmarshal([]byte(jsonStr), &u2)
    }

## 读写锁

    package main
    import (
    	"errors"
    	"fmt"
    	"sync"
    )
    var (
    	pcodes         = make(map[string]string)
    	mutex          sync.RWMutex
    	ErrKeyNotFound = errors.New("Key not found in cache")
    )
    func Add(address, postcode string) {
        // 写入的时候要完全上锁
    	mutex.Lock()
    	pcodes[address] = postcode
    	mutex.Unlock()
    }
    func Value(address string) (string, error) {
        // 读取的时候，只用读锁就可以
    	mutex.RLock()
    	pcode, ok := pcodes[address]
    	mutex.RUnlock()
    	if !ok {
    		return "", ErrKeyNotFound
    	}
    	return pcode, nil
    }
    func main() {
    	Add("henan", "453600")
    	v, err := Value("henan")
    	if err == nil {
    		fmt.Println(v)
    	}
    }
    
## command app

[urfave/cli](https://github.com/urfave/cli)

cli is a simple, fast, and fun package for building command line apps in Go. The goal is to enable developers to write fast and distributable command line applications in an expressive way.

Things like generating help text and parsing command flags/options should not hinder productivity when writing a command line app.This is where cli comes into play. cli makes command line programming fun, organized, and expressive!

go语言中的`github.com/gorilla`可以方便的进行http url 到处理方法的dispatch，`github.com/urfave/cli` 则实现了用户输入命令到处理方法的dispatch。