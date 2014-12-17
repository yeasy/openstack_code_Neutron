#### processutils.py
封装了一些进程操作，提供高层方法，包括：
* _subprocess_setup()方法
* execute()方法，在shell中通过subprocess来执行一条命令。
* trycmd()方法，对execute()方法的封装，处理警告和错误信息。
* ssh_execute
