---
layout: post
title:  "go pprof trace"
categories: go
tags: pprof trace 
author: wkl
---

关于火焰图与trace的使用

go-torch appBin test.prof test.svg

go tool trace -http=x.x.x.x:8888 trace.trace


    package prof
    
    import (
    	"os"
    	"log"
    	_ "net/http"
    	"runtime/pprof"
    	"runtime/trace"
    	_ "net/http/pprof"
    )
    
    
    var CpuProf *os.File
    var MemProf *os.File
    var TraceProf *os.File
    var StartProfChan = make(chan uint32, 1)
    var StopProfChan = make(chan uint32, 1)
    var StartTraceChan = make(chan uint32, 1)
    var StopTraceChan = make(chan uint32, 1)
    
    func SetStartProf() {
    	StartProfChan <- 1
    }
    func SetStopProf() {
    	StopProfChan <- 1
    }
    
    func SetStartTrace() {
    	StartTraceChan <- 1
    }
    func SetStopTrace() {
    	StopTraceChan <- 1
    }
    func StartTrace() {
    	var errTrace error
    	for {
    		<-StartTraceChan
    		TraceProf, errTrace = os.Create("./trace.trace")
    		if errTrace != nil {
    			log.Fatal(errTrace)
    		}
    		trace.Start(TraceProf)
    	}
    }
    
    func StopTrace() {
    	for {
    		<-StopTraceChan
    		trace.Stop()
    	}
    }
    
    func StartProf() {
    	var errCPU, errMEM error
    	for {
    		<-StartProfChan
    		CpuProf, errCPU = os.OpenFile("./cpu.prof", os.O_RDWR|os.O_CREATE, 0644)
    		if errCPU != nil {
    			log.Fatal(errCPU)
    		}
    		MemProf, errMEM = os.Create("./mem.prof")
    		if errMEM != nil {
    			log.Fatal(errMEM)
    		}
    		pprof.StartCPUProfile(CpuProf)
    	}
    }
    
    func StopProf() {
    	for {
    		<-StopProfChan
    		pprof.StopCPUProfile()
    		CpuProf.Close()
    		pprof.WriteHeapProfile(MemProf)
    		MemProf.Close()
    	}
    }
    