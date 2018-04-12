# 客户端通信协议

* 基于TCP
* Redis制定了RESP(Redis Serialization Protocol)实现客户端和服务端的通信


## RESP简单例子
### 发送命令格式
CRLF代表`\r\n`
```
* <参数数量> CRLF
$ <参数1字节数量> CRLF
<参数1> CRLF
$ <参数n字节数量> CRLF
<参数n> CRLF
```

以`SET HELLO WORLD`为例，其对应的RESP的格式为`*3\r\n$3SET\r\n$5\r\nHELLO\r\n$5\r\nHELLO\r\n`

这么看起来不太方便，格式后的内容是这样的
```
* 3
$3
SET
$5
HELLO
$5
WORLD
``` 

### 返回结果格式
Redis返回结果格式分为5中
* 状态回复: `RESP中第一个字节为+，例如SET`
* 错误回复: `RESP中第一个字节为-，例如错误命令`
* 整数回复: `RESP中第一个字节为:，例如INCR`
* 字符串回复: `RESP中第一个字节为$，例如GET`
* 多条字符串回复: `RESP中第一个字节为*，例如MGET`


### 验证方法一：翻看Redis源代码
由于redis-cli会自动将RESP格式数据格式化为更为可读的数据，所以验证上面的说法，需要看源代码，返回结果处理在`redis-cli.c`的417~482行
```c
 417 static sds cliFormatReplyTTY(redisReply *r, char *prefix) {
 418     sds out = sdsempty();
 419     switch (r->type) {
 420     case REDIS_REPLY_ERROR:
 421         out = sdscatprintf(out,"(error) %s\n", r->str);
 422     break;
 423     case REDIS_REPLY_STATUS:
 424         out = sdscat(out,r->str);
 425         out = sdscat(out,"\n");
 426     break;
 427     case REDIS_REPLY_INTEGER:
 428         out = sdscatprintf(out,"(integer) %lld\n",r->integer);
 429     break;
 430     case REDIS_REPLY_STRING:
 431         /* If you are producing output for the standard output we want
 432         * a more interesting output with quoted characters and so forth */
 433         out = sdscatrepr(out,r->str,r->len);
 434         out = sdscat(out,"\n");
 435     break;
 436     case REDIS_REPLY_NIL:
 437         out = sdscat(out,"(nil)\n");
 438     break;
 439     case REDIS_REPLY_ARRAY:
 440         if (r->elements == 0) {
 441             out = sdscat(out,"(empty list or set)\n");
 442         } else {
 443             unsigned int i, idxlen = 0;
 444             char _prefixlen[16];
 445             char _prefixfmt[16];
 446             sds _prefix;
 447             sds tmp;
 448 
 449             /* Calculate chars needed to represent the largest index */
 450             i = r->elements;
  451             do {
 452                 idxlen++;
 453                 i /= 10;
 454             } while(i);
 455 
 456             /* Prefix for nested multi bulks should grow with idxlen+2 spaces */
 457             memset(_prefixlen,' ',idxlen+2);
 458             _prefixlen[idxlen+2] = '\0';
 459             _prefix = sdscat(sdsnew(prefix),_prefixlen);
 460 
 461             /* Setup prefix format for every entry */
 462             snprintf(_prefixfmt,sizeof(_prefixfmt),"%%s%%%ud) ",idxlen);
 463 
 464             for (i = 0; i < r->elements; i++) {
 465                 /* Don't use the prefix for the first element, as the parent
 466                  * caller already prepended the index number. */
 467                 out = sdscatprintf(out,_prefixfmt,i == 0 ? "" : prefix,i+1);
 468 
 469                 /* Format the multi bulk entry */
 470                 tmp = cliFormatReplyTTY(r->element[i],_prefix);
 471                 out = sdscatlen(out,tmp,sdslen(tmp));
 472                 sdsfree(tmp);
 473             }
 474             sdsfree(_prefix);
 475         }
 476     break;
 477     default:
 478         fprintf(stderr,"Unknown reply type: %d\n", r->type);
 479         exit(1);
 480     }
 481     return out;
 482 }
```

### 验证方法二：使用nc
为了标识，`INPUT> `仅作为标识，后面的是命令
 ```
 root@iZ947mgy3c5Z:/opt/redis/src# nc 127.0.0.1 6379
 INPUT> SET HELLO WORLD
 +OK
 
 INPUT> INCR COUNT  
 :1
 
 INPUT> ERROR_CMD
 -ERR unknown command 'ERROR_CMD'
 
 INPUT> GET HELLO      
 $5
 WORLD
    
 INPUT> MGET HELLO COUNT
 *2
 $5
 WORLD
 $1
 1
 ```
 
 
RESP简单来说就是这样了，redis-py就是基于RESP标准实现的工具，所以我是不是可以认为redis-py主要就是做了，
1. 调用函数接收命令和参数
2. 使用RESP序列化
3. 发送给Redis服务端
4. 解析服务端Redis结果序列
5. 返回给调用函数


