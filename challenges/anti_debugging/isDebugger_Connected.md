# isDebugger Connected
Task: Your task is to attach a debugger to find a flag.

## Attach jdb to the process
Start the app and get the PID of the process
```
$ adb shell ps | grep hpandro
u0_a101       2526   308 1480888 252268 ep_poll      ea3e8bb9 S com.hpandro.androidsecurity
```

Create a communication channel by using adb between the application process (with the PID) and your host computer by using a specific local port:
```
$ adb forward tcp:55555 jdwp:2526
```

Attach the debugger
```
$ adb -connect com.sun.jdi.SocketAttach:hostname=127.0.0.1,port=55555    
Set uncaught java.lang.Throwable                                                            
Set deferred uncaught java.lang.Throwable   
Initializing jdb ...
```

![img](https://github.com/cygnus-xr1/hpAndro_CTF_walkthrough/blob/main/challenges/anti_debugging/img/solved.png?raw=true)

## Reference
* https://github.com/OWASP/owasp-mstg/blob/master/Document/0x05i-Testing-Code-Quality-and-Build-Settings.md