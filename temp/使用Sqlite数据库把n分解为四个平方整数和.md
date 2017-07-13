# 使用Sqlite3数据库把n分解为四个平方整数和

## 一、实验介绍

### 1.1 实验内容
 简单介绍了Sqlite3数据库的使用，以及C语言对其编程，使用穷举法列出了(和值为1..65536）的四个平方整数和的所有可能情况，验证了数论中的
Lagrange四平方和定理 (Lagrange's four-square theorem), 也被叫做Bachet猜想(Bachet's conjecture):
任何自然数可以表示成四个平方整数之和.

### 1.2 实验知识点 
- sqlite3数据库在Linux中的使用
- C接口编程操作Sqlite3数据表
- sqlite数据表导出.csv, .html
- 验证4096以内Lagrange四平方和定理的情况

### 1.3 实验环境
- Linux Ubuntu 14.04
- sqlite3
- gcc

### 1.4 适合人群
本课程难度为一般，属于中级级别课程，适合具有C编程基础和数据库基础的用户

### 1.5 代码获取
【本课代码的获取方法】
https://github.com/avvboy/avvboy.github.io/raw/master/temp/使用Sqlite数据库把n分解为四个平方整数和.md

## 二、实验原理
循环列出1个, 2个, 3个, 4个平方整数和(=n)的等式，写入首行为n的数据表

## 三、开发准备
访问sqlite网站，从 https://www.sqlite.org/download.html 下载
[sqlite3 Precompiled Binaries for Linux](https://www.sqlite.org/2017/sqlite-tools-linux-x86-3190300.zip)
[sqlite3 source code as an amalgamation, version 3.19.3](http://www.sqlite.org/2017/sqlite-amalgamation-3190300.zip)

接着unzip这两个文件。若实验楼无法解析域名sqlite.org，请从其他地方下载中转这两个文件到实验虚拟机里，整个项目文件结构如下

## 四、项目文件结构

```
~/workspace/foursquare $ tree ./
./
├── 4square
├── 4square.c
├── foursquare.db
├── sqlite-amalgamation-3190300
│   ├── shell.c
│   ├── sqlite3.c
│   ├── sqlite3.h
│   └── sqlite3ext.h
├── sqlite-amalgamation-3190300.zip
├── sqlite-tools-linux-x86-3190300
│   ├── sqldiff
│   ├── sqlite3
│   └── sqlite3_analyzer
├── sqlite-tools-linux-x86-3190300.zip
├── squares.csv
├── temp.sql
├── test
└── test.c

2 directories, 16 files
```

## 五、实验步骤

### 1. 下载sqlite3
```
$ mkdir foursquare; cd foursquare

$ wget https://www.sqlite.org/2017/sqlite-tools-linux-x86-3190300.zip
$ unzip sqlite-tools-linux-x86-3190300.zip 
$ wget http://www.sqlite.org/2017/sqlite-amalgamation-3190300.zip
$ unzip sqlite-amalgamation-3190300.zip
```

### 2. 建立数据库foursquare.db，创建表格
```
$ sqlite-tools-linux-x86-3190300/sqlite3 foursquare.db

sqlite> CREATE TABLE t(n INTEGER PRIMARY KEY ASC, OneSquare varchar(16),TwoSquare varchar(128),ThreeSquare varchar(256),FourSquare varchar(1024), eqCount integer default(0));
sqlite> .quit
$ mv foursquare.db foursquare123.db
```

### 3. 编写C程序 4square.c ，写入四平方整数和的数据

$ gcc -O3 sqlite-amalgamation-3190300/sqlite3.c 4square.c -o 4square -lpthread -ldl  
$ ./4square  foursquare.db

4square.c的源代码如下:
```
#include <stdio.h>
#include <string.h>
#include "sqlite-amalgamation-3190300/sqlite3.h"

static int callback(void *NotUsed, int argc, char **argv, char **azColName){
  // int i;
  // for(i=0; i<argc; i++){
  //   printf("%s = %s\n", azColName[i], argv[i] ? argv[i] : "NULL");
  // }
  // printf("\n");
  return 0;
}

int main(int argc, char **argv){
  const int M=4096,sqrtM=64;
  sqlite3 *db;
  char *zErrMsg = 0;
  int rc;
  char sql[1024]="";

  if( argc!=2 ){
    fprintf(stderr, "Usage: %s foursquare.db\n", argv[0]);
    return(1);
  }
  rc = sqlite3_open(argv[1], &db);
  if( rc ){
    fprintf(stderr, "Can't open database: %s\n", sqlite3_errmsg(db));
    sqlite3_close(db);
    return(1);
  }
  
  //创建表格
   sprintf(sql,"CREATE TABLE t(n INTEGER PRIMARY KEY ASC, OneSquare varchar(16),TwoSquare varchar(128),ThreeSquare varchar(256),FourSquare varchar(1024), eqCount integer default(0));");
   rc = sqlite3_exec(db, sql, callback, 0, &zErrMsg);
    if( rc!=SQLITE_OK ){
     fprintf(stderr, "SQL error: %s\n", zErrMsg);
     sqlite3_free(zErrMsg);
   }
   
  
  int i=0;
  for (i = 1; i <= M; i++) {
    sprintf(sql,"insert into t values(%i,null,null,null,null,0)",i);
   // printf("%s",sql);
    rc = sqlite3_exec(db, sql, callback, 0, &zErrMsg);
    if( rc!=SQLITE_OK ){
     fprintf(stderr, "SQL error: %s\n", zErrMsg);
     sqlite3_free(zErrMsg);
   }
  }
  
  for (i = 1; i <= sqrtM; i++) {
    sprintf(sql,"update t set OneSquare='%i^2' , eqCount=1 where n=%i",i,i*i);
    rc = sqlite3_exec(db, sql, callback, 0, &zErrMsg);
    if( rc!=SQLITE_OK ){
     fprintf(stderr, "SQL error: %s\n", zErrMsg);
     sqlite3_free(zErrMsg);
   }
  }
  
  int j=1,k=1,h=1,s=2;
  for (i = 1; i <= sqrtM; i++) {
    for (j = 1; j<=i; j++) {
      s=i*i+j*j;
      if(s>M) break;
      sprintf(sql,"update t set TwoSquare='=%i^2+%i^2' , eqCount=1 where n=%i",i,j,s);
      rc = sqlite3_exec(db, sql, callback, 0, &zErrMsg);
      if( rc!=SQLITE_OK ){
       fprintf(stderr, "SQL error: %s\n", zErrMsg);
       sqlite3_free(zErrMsg);
      }
    }
  }
  
  for (i = 1; i <= sqrtM; i++) {
    for ( j = 1; j<=i; j++) {
      for ( k = 1; k<=j; k++) {
        s=i*i+j*j+k*k;
        if(s>M) break;
        sprintf(sql,"update t set ThreeSquare='=%i^2+%i^2+%i^2' , eqCount=1 where n=%i",i,j,k,s);
        rc = sqlite3_exec(db, sql, callback, 0, &zErrMsg);
        if( rc!=SQLITE_OK ){
         fprintf(stderr, "SQL error: %s\n", zErrMsg);
         sqlite3_free(zErrMsg);
        }
      }
    }
  }
  
  for (i = 1; i <= sqrtM; i++) {
    for ( j = 1; j<=i; j++) {
      for ( k = 1; k<=j; k++) {
        for ( h = 1; h<=k; h++) {
          s=i*i+j*j+k*k+h*h;
          if(s>M) break;
          sprintf(sql,"update t set FourSquare='=%i^2+%i^2+%i^2+%i^2' , eqCount=1 where n=%i",i,j,k,h,s);
          rc = sqlite3_exec(db, sql, callback, 0, &zErrMsg);
          if( rc!=SQLITE_OK ){
           fprintf(stderr, "SQL error: %s\n", zErrMsg);
           sqlite3_free(zErrMsg);
          }
        }
      }
    }
  }
  
  sqlite3_close(db);
  return 0;
}
```

### 4. (步骤4可省略)步骤3采用C编程运行sql语句，也可以使用javascript循环语句生成sql语句集，然后编辑到一个temp.sql文件中执行，
  如在Chrome的console中运行:

```
  const M=4096,sqrtM=64;
  var i=1,j=1,k=1,h=1,s=2,sql="";
  for (i = 1; i <= sqrtM; i++) {
    for (j = 1; j<=i; j++) {
      s=i*i+j*j;
      if(s>M) break;
      sql+= "update t set TwoSquare=TwoSquare||'="+i+"^2+"+j+"^2' , eqCount=1 where n="+s+" ;\n";
    }
  }
  console.log(sql)
```

这将生成一部分的sql语句，把需要的sql编辑好，存到temp.sql,然后执行

```
$ sqlite-tools-linux-x86-3190300/sqlite3  foursquare.db temp.sql
```

### 5. 查看已经生成的数据库文件foursquare.db，导出数据

```
$ sqlite-tools-linux-x86-3190300/sqlite3  foursquare.db
sqlite> select * from t where n >4000;
sqlite> .help
sqlite> .mode csv
sqlite> .output squares.csv
sqlite> select * from t;
sqlite> .quit
$ more squares.csv
```

## 六、实验总结
这个实验演示了Linux下C语言对Sqlite3编程的过程，以及命令行方式处理查看sqlite数据库内容，验证了0<n<=4096时，n可以表示成四个平方整数之和.

## 七、课后习题
Javascript对sqlite编程，采集github热门开源项目信息。可参考 https://github.com/kripken/sql.js/

## 八、参考链接
https://en.wikipedia.org/wiki/Lagrange%27s_four-square_theorem
https://www.sqlite.org/quickstart.html
https://github.com/kripken/sql.js/