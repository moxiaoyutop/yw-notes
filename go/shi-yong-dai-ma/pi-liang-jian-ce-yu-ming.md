# 批量检测域名

```
package main

import (
   "bufio"
   "fmt"
   "io"
   "io/ioutil"
   "net/http"
   "os"
   "time"
)

func main()  {
   start :=time.Now()
   ch :=make(chan string)

   urls := readFilee("domian.txt")
   for _,url:=range urls[:len(urls)]{
      go fetch(url,ch)
   }

   for range urls[:len(urls)]{
      fmt.Println(<-ch)
   }
   fmt.Printf("%.2fs elapsed \n",time.Since(start).Seconds())
   fmt.Scanf("a")
}

func readFilee(filePath string) ([]string) {
   //打开文件
   fi, err := os.Open(filePath)
   if err != nil {
      fmt.Println(err)
   }
   defer fi.Close()

   buf := bufio.NewScanner(fi)
   // 循环读取
   var lineArr []string
   for {
      if !buf.Scan() {
         break //文件读完了,退出for
      }
      line := buf.Text() //获取每一行
      lineArr = append(lineArr, line)
   }
   return lineArr
}

func fetch(url string,ch chan<- string){
   start:=time.Now()
   res,err:=http.Get("http://"+url)
   if err!=nil{
      ch <- fmt.Sprint(err)
      return
   }
   nbytes, err := io.Copy(ioutil.Discard,res.Body)
   if err!=nil{
      ch <- fmt.Sprintf("while reading %s:%v",url,err)
      return
   }
   secs := time.Since(start).Seconds()
   ch <- fmt.Sprintf("%.2fs %7d %s",secs,nbytes,url)
}


```
