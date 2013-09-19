#streamming进行grep
hadoop jar streamming.jar -mapper 'grep xxx'是不够滴  
由于streamming默认情况下，mapper和reducer的返回值不是0，被认为是异常任务，将被再次执行，默认尝试四次都不是0，整个job就fail了。  
所以需要添加  

    -jobconf stream.no.zero.exit.is.failure=false

意思就是把这个默认的设定disable掉  
