<h3>一、设计概要</h3>

本设计采用异步任务队列的方式进行样本的分析，所有投放的样本都会形成分析任务，进入消息队列，分析任务id采用guid的方式进行分配，确保分布式沙箱的任务id唯一，样本投放分为web界面投放和http api的方式投放。样本投放的接口会首先判断该样本是否投放过，也就是样本文件系统是否存在该样本，如果不存在则样本会上传到文件系统，如果已存在，则直接生成任务。当样本投放完成后，分布式的分析节点会从消息队列中获取任务，任务消息内容包括了样本资源地址，分析优先级，任务id等信息。分析节点获取文件后进行任务判断，判断该文件是否支持动态分析，如果不支持动态分析则进行静态分析，静态分析采用多病毒引擎扫描结合yara静态扫描，主要产出静态特征字符特征向量的检测结果，如果支持动态分析，则先产生静态分析报告再进行动态分析，动态分析的过程中释放产生的衍生文件在动态分析结束后也会进行对应的静态分析检测，并且输出分析日志。动态分析产生的日志会被推送到一个消息队列进行动态系统恶意行为检测与关联分析，动态行为遵循ATT&CK框架，动态日志分析结束后最终产出动态分析报告。


特别需要指出的是邮件类型和文件和压缩包均会包含多个文件，需要限制遍历的深度，防止一个任务占用太多的资源，同时也应该限制最大上传文件分析的大小。文件的回扫机制需要加以设计，当静态特征库更新后，之前未检出的恶意样本可能会被重新检出，这时需要更新样本分析报告的恶意判定结果。

<h3>二、逻辑业务</h3>

思考问题1：如果动态分析在沙箱内部直接使用kafka 消息队列传输是否存在安全隐患，最好在宿主机器上设置一个代码缓存接受分析日志与文件，然后有宿主机器进行日志推送，所有的日志附带任务全局id，区分不同样本的分析日志。


思考问题2：如果有多个动态日志分析器获取消息队列中的日志进行分析，如何能使用组合规则进行关联分析。应该设计一个组合规则关联缓存，该缓存是所有日志分析器都能访问到的一个地方，组合关联实时更新，多个分析器总是能取到上一个分析器关联后的结果，最终分析完成后输出总体关联结果。


思考问题3：设计合理的组合规则和单条规则库，能够及时更新，提供良好的持续运维能力。


<h3>
三、模块划分及接口
</h3>

<h3>
四、可扩展设计
</h3>

<h3>
五、可靠性设计
</h3>

<h3>
六、安全性设计
</h3>