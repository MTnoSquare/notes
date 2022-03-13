
## StringDecoder

`StringDecoder`将接收到的对象转换为字符串,然后继续调用后面的Handler.
## LineBasedFrameDecoder 

`LineBasedFrameDecoder`通过判断是否有`“\n”`或者`"\r\n"`,即以换行符为结束标志;同时支持配置单行的最大长度,如果连续读取到最大长度后仍然没发现换行符,就会抛出异常.



##  DelimiterBasedFrameDecoder 

按照指定的分隔符对消息解码
## FixedLengthFrameDecoder

固定长度解码器,能够按照指定的长度对消息进行自动解码.
