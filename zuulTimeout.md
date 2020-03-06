1、 spring cluod zuul time out:

1）某个commandkey： "hystrix.command." + commandKey + ".execution.isolation.thread.timeoutInMilliseconds"

2）默认： hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds   

3）ribbion time out
    (ribbonReadTimeout + ribbonConnectTimeout) * (maxAutoRetries + 1) * (maxAutoRetriesNextServer + 1)
    默认ribbonReadTimeout = 1s, 默认ribbionConnectTimeOut = 1s
    
 参看AbstractRibbonCommand  getHystrixTimeout  getRibbonTimeout
