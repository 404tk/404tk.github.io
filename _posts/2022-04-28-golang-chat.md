---
title: Golang实现简易的多人聊天室
author: 404tk
date: 2022-04-28
category: golang
layout: post
---

直接贴代码：  
```go
package main

import (
	"bufio"
	"flag"
	"fmt"
	"log"
	"net"
	"time"
)

type Server struct {
	Chaters map[string]net.Conn
}

var port string

func init() {
	flag.StringVar(&port, "p", "8080", "Listen port")
	flag.Parse()
}

func main() {
	//初始化
	chaters := Server{make(map[string]net.Conn, 0)}

	//监听
	conn, err := net.Listen("tcp", "0.0.0.0:"+port)
	if err != nil {
		log.Fatalln("服务端监听失败！")
	}
	for {
		//允许握手
		c, err := conn.Accept()
		if err != nil {
			log.Println(err)
			break
		}

		go chaters.registry(c)
		go chaters.handleConn(c)
	}
}

func (s *Server) handleConn(conn net.Conn) {
	defer conn.Close()
	r := bufio.NewReader(conn)
	for {
		time.Sleep(100 * time.Microsecond)
		data, err := r.ReadString('\n')
		if err != nil {
			// 退出会话
			delete(s.Chaters, conn.RemoteAddr().String())
			msg := fmt.Sprintf("【%s】退出房间\n", conn.RemoteAddr().String())
			s.sendMsg(conn.RemoteAddr().String(), msg)
			log.Print(msg)
			break
		}
		msg := fmt.Sprintf("【%s】%s\n", conn.RemoteAddr().String(), data)
		s.sendMsg(conn.RemoteAddr().String(), msg)
		log.Print(msg)
	}
}

// 注册会话
func (s *Server) registry(conn net.Conn) {
	key := conn.RemoteAddr().String()
	if _, ok := s.Chaters[key]; !ok {
		s.Chaters[key] = conn
		msg := fmt.Sprintf("【%s】加入房间\n", key)
		s.sendMsg(key, msg)
		log.Print(msg)
	}
}

// 广播消息
func (s *Server) sendMsg(addr, msg string) {
	for _addr, conn := range s.Chaters {
		if addr != _addr {
			conn.Write([]byte(msg))
		}
	}
}
```  

使用方式：  
- Server端编译后运行，默认监听`8080`端口  
- Client端使用`nc`连接，如：`nc 192.168.1.2 8080`  