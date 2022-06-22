---
title: Golang实现简易的多人聊天室
author: 404tk
date: 2022-04-28
category: golang
layout: post
---

Server端代码：  
```go
package main

import (
	"bufio"
	"flag"
	"fmt"
	"io"
	"log"
	"net"
	"strings"
	"time"
)

type Server struct {
	Chaters map[string]net.Conn
}

var (
	port   string
	secret = "password"
)

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

		c.Write([]byte("Input your code: "))
		go chaters.handleConn(c)
	}
}

func (s *Server) handleConn(conn net.Conn) {
	defer conn.Close()
	r := bufio.NewReader(conn)
	addr := conn.RemoteAddr().String()
	for {
		time.Sleep(100 * time.Microsecond)
		data, err := r.ReadString('\n')
		if err != nil {
			if err == io.EOF {
				s.quit(conn, addr)
			}
			break
		}
		data = strings.Trim(data, "\n")
		var msg string
		if _, ok := s.Chaters[addr]; !ok {
			if data == secret {
				s.register(conn, addr)
				banner := fmt.Sprintf("Welcome!\nCurrent online: %d\n", len(s.Chaters))
				conn.Write([]byte(banner))
				msg += fmt.Sprintf("【%s】加入房间", addr)
			} else {
				break
			}
		} else if len(data) > 0 {
			if data == "exit" || data == "quit" {
				s.quit(conn, addr)
				break
			}
			msg = fmt.Sprintf("【%s】%s", addr, data)
		}
		if len(msg) > 0 {
			s.sendMsg(addr, msg+"\n")
			log.Print(msg)
		}
		conn.Write([]byte("[Enter]> "))
	}
}

// 注册会话
func (s *Server) register(conn net.Conn, addr string) {
	s.Chaters[addr] = conn
}

// 退出会话
func (s *Server) quit(conn net.Conn, addr string) {
	delete(s.Chaters, addr)
	msg := fmt.Sprintf("【%s】退出房间", addr)
	s.sendMsg(addr, msg+"\n")
	log.Print(msg)
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