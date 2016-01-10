# leaf-consistent-hashing## leaf-consistent-hashing是一个遵循PSR-3规范的，分布式算法组件，适用于基于客户端的集群分布式场景：如Redis、Memcached、MySQL。### 特点：1. 使用composer包，便于集成到基于composer的项目中。2. 遵循psr-3标准规范。所以，使用leaf-consistent-hashing替换现有系统的日志组件是非常简单的。### 适用说明：1. 一致性哈希算法相对取模算法，优势在于：①：节点变更重新哈希的Key落点只影响单台。②：分布更均匀。2. 本算法组件，适用于基于客户端的分布式解决方案。3、特别适用场景：Redis的集群分布式算法（Redis官方在3.0版本才出了自己的分布式解决方法）。4、虽然基于MySQL做分布式方案解决方案有很多且比较成熟，但对于小团队而言，使用一致性哈希替代取模算法同样不失为一种较好的解决方案。5、本组件未集成取模算法，也没有集成的必要。## 1.引入leaf-consistent-hashing### 引入有2种方式：1、基于composer的项目的引入方式：``` composer require leaf/leaf-consistent-hashing```2、非composer项目引入方式：``` \Leaf\Consistent\Hashing\Autoloader::register();```## 2.leaf-consistent-hashing的基本使用leaf-consistent-hashing的核心理念是：有一个log manager。你可以向log manager中注册多个日志处理器，比如：handlerFile，用于记录文件日志。你还可以再注册进去一个 handlerMail，用于当记录error级别的日志的时候，发送报警邮件等。当然，你还可以自己定制化handler，注册到log manager中。leaf-consistent-hashing默认仅仅提供了文件处理器。首先：实例化一个日志控制器。``` $loger = new \Leaf\Loger\Loger。```然后，实例化一个日志处理器，这里以文件处理器为例：``` $handlerFile = new \Leaf\Loger\Handler\HandlerFile();```当然，既然是存储文件，你可能需要指定文件的存储路径，存储路径只有文件处理器需要。（倘若你自己写了日志处理器是短信日志处理器，那么设置路径的方法就没有必须要了）``` $handlerFile->setLogFile($path);```接下来，我们将日志处理器注册到日志控制器中。这样的目的是，当触发日志控制器的记录操作的时候，系统会通过文件处理器完成记录操作。``` $loger->addHandler('file',$fileHandler);```最后，我们可以这样记录日志。``` $loger->info("this is a test string");```**集成方式**> 上面的例子，是基本的使用方式。在你的项目中，你很可能不会每次都实例化日志记录器才能记录日志。而是> > 1、通过注册树模式，将日志记录器注册到全局树上。> > 2、或者是注册到项目的容器中。> > 3、注册到项目application的静态方法中，以Yii2为例：Yii::info()。leaf-consistent-hashing提供了一个logDriver，你可以参考其实现，将leaf-consistent-hashing融入到你的项目中：``` leaf-consistent-hashing/src/Example/LogDriver.php```## 3.日志记录级别说明：你可以选择的记录日志级别有8个。**EMERGENCY**> 系统不可用。``` $loger->emergency('emergency message string');```**ALERT**> **必须**立刻采取行动> > 例如：在整个网站都垮掉了、数据库不可用了或者其他的情况下，**应该**发送一条警报短信把你叫醒。``` $loger->alert("alert message string");```**CRITICAL**> 紧急情况> > 例如：程序组件不可用或者出现非预期的异常。``` $loger->critical('critical message string');```**ERROR**> 运行时出现的错误，不需要立刻采取行动，但必须记录下来以备检测。``` $loger->error('error message string');```**WARNING**> 出现非错误性的异常> > 例如：使用了被弃用的API、错误地使用了API或者非预想的不必要错误。``` $loger->warning('warning message string');```**NOTICE**> 一般性重要的事件``` $loger->notice('notice messsage string');```**INFO**> 一般事件> > 例如：用户登录和SQL记录。``` $loger->info("sql...");```**DEBUG**> 调试详情> > 例如：某个断点的内存占用、执行时间等等。``` $loger->debug('debug message string');```## 4.设置日志级别你可以自定义日志级别，这样一来，只有高于或等于此级别的日志才能被记录。举例来说，你可以设置记录级别为warning，那么debug级别的日志将不会记录。``` php#use \Leaf\Loger\LogerClass\LogLevel;$loger->setLogLevel(LogLevel::WARNING);```此时，你执行如下代码``` $logDriver->info('info');		//不记录$logDriver->warning('warning');	//记录$logDriver->error('error');		//记录```## 5.设置日志类别除了可以设置日志级别外，还可以设置日志类别。比如：你的项目中有记录2种类别的日志：application（应用类别）、profile（性能分析类别）。你可以同时在项目中去记录。但是当项目部署的时候，只开放 application 类别，那么profile的即便有函数调用，但依然不会记录。设置日志记录类别：``` $loger->setLogCategory('profile');```设置之后，下面的代码：``` $loger->info('info', [], 'profile');	//记录$loger->warning('warning', [], 'app');	//不记录$loger->error('error', [], 'profile');	//记录```## 6.打开日志实时刷新如果你的日志是记录到文件中的。默认情况下，日志组件是关闭实时刷新日志的。也就是说。你多次调用记录日志的方法，并不会立即将日志信息刷新到文件中，而是当脚本结束的时候才刷新写入日志。这样做的好处是，减少高并发下的磁盘IO负载。当然，你也可以选择手动开启。开启的方式很简单，实例化文件处理器的时候，如下操作：``` /*** 设置日志处理器之文件处理器*/$fileHandler = new HandlerFile();$fileHandler->enableRealTimeFlush();/*** 将文件日志处理器添加到loger中*/$this->getLoger()->addHandler('file', $fileHandler);```## todo.计划内容：- 文件处理器的日志分割。