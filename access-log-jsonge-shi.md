# access log json格式

```xml
<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
 prefix="localhost_access_log." suffix=".txt"
 pattern="{&quot;client_ip&quot;:&quot;%{X-Real-IP}i&quot;,&quot;client_user&quot;:&quot;%l&quot;,&quot;authenticated&quot;:&quot;%u&quot;,&quot;access_time&quot;:&quot;%t&quot;,&quot;method&quot;:&quot;%m&quot;,&quot;status&quot;:&quot;%s&quot;,&quot;total_bytes_sent&quot;:&quot;%b&quot;,&quot;http_referer&quot;:&quot;%{Referer}i&quot;,&quot;url&quot;:&quot;%U&quot;,&quot;http_user_agent&quot;:&quot;%{User-Agent}i&quot;}"/>
```